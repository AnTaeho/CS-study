## 비동기 처리되는 이벤트 유실을 방지하기 위해 Transactional Outbox Pattern 도입

### 문제 상황

- 게시글 작성시 게시글에 포함되는 이미지 저장을 스프링 Event와 비동기 처리 활용
- 저장에 실패하거나 서버 다운등 이벤트 유실시에 저장을 보장할 수 없다.
- 발행된 이벤트 처리를 보장할 수 있는 방법을 탐색

### 해결 방법

- 분산 처리 환경에서 카프카 이벤트 유실 문제에 사용되는 **Transactional Outbox Pattern**을 도입하기로 결정
  - **BEFORE_COMMIT** : 활용하여 게시글 저장 트랜잭션 내에서 이벤트의 발행 정보와 처리 상태를 RDBMS에서 저장
  - **AFTER_COMMIT** : 활용하여 비동기적으로 이미지 저장을 실행하고 수행됨을 업데이트
- 주기적으로 실패한 로직을 재시도하고, 처리후 일정 시간이 지난 이벤트 정보를 삭제하여 테이블 크기를 조절

### 코드

### EventListener

트랜잭션 내에서 실행되는 정보 저장 로직은 동기적으로 처리하고, 이후에 실제 저장하고 상태 업데이트 로직은 비동기로 처리했습니다.

```java
@TransactionalEventListener(value = ImageSaveEvent.class, phase = TransactionPhase.BEFORE_COMMIT)
public void savePublishRecord(ImageSaveEvent event) {
    Long postId = event.postId();
    for (String url : event.values()) {
        imageOutboxEventRepository.save(new ImageOutboxEvent(url, postId));
    }
}

@Async
@TransactionalEventListener(value = ImageSaveEvent.class, phase = TransactionPhase.AFTER_COMMIT)
public void executePublishedEvent(ImageSaveEvent event) {
    Long postId = event.postId();
    for (String url : event.values()) {
        try {
            imageRepository.save(new Image(url, postId));
            imageOutboxEventRepository.updateExecuted(url, postId);
        } catch (Exception e) {
            log.warn(e.getMessage());
        }
    }
}
```

### ImageService

```java
@Scheduled(fixedDelay = 300000)
public void executeFailedEvent() {
    List<ImageOutboxEvent> result =imageOutboxEventRepository.findAllFailEvent();
    for (ImageOutboxEvent imageOutboxEvent : result) {
        imageRepository.save(new Image(imageOutboxEvent.getImageUrl(), imageOutboxEvent.getPostId()));
        imageOutboxEventRepository.updateExecuted(imageOutboxEvent.getImageUrl(), imageOutboxEvent.getPostId());
    }
}

@Scheduled(fixedDelay = 3600000)
public void deleteOldEvent() {
    LocalDateTime oneHourAgo = LocalDateTime.now().minusHours(1);
    imageOutboxEventRepository.deleteOlderThan(oneHourAgo);
}
```

### 결과

- 추가적인 DB 공간과 부하가 발생하지만 **이벤트가 실행되는 것을 보장**
- 일정 시간마다 실패한 이벤트 처리를 재시도 하여 **이벤트 유실 방지**
- 발행 정보가 저장 되는 테이블의 크기는 일정시간마다 정리하여 **부하 관리**
