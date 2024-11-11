## Redis를 활용한 대기열 구현

### 문제 상황
- 인기 상품이나 선착순 쿠폰 이벤트시 동시에 수많은 요청 발생
- 예상치 못한 수준의 트래픽에 서버가 다운될 수도 있다고 판단했습니다.
- 실제로 서버에 요청하는 트래픽의 수를 제한해서 트래픽 부하를 제어하기로 결정했습니다.

### 해결 방법

![스크린샷 2024-10-02 오후 6 26 12](https://github.com/user-attachments/assets/9295177d-007b-41d3-8e0b-9fd05228976a)


### Redis ZSet을 활용한 대기열 구현

- Redis의 Sorted Set은 **Key, Score, Value**의 형태로 이루어져 있으며 **Score를 기반으로 정렬**
- 개별 요청의 **Request Timestamp를 Score**로, 사용자 식별 키 값을 Value로 사용하여 진입한 **순서대로 사용자가 서비스에 접근**할 수 있도록 구현
- 3초마다 Processing Queue의 빈자리를 확인해 Waiting Queue에서 이동

```java
public String enqueueToken(Long userId) {
    if (redisQueueRepository.alreadyEnQueued(userId)) {
        throw new IllegalArgumentException(ALREADY_IN_QUEUE_KEY);
    }

    String token = jwtUtil.generateToken(userId);
    Long score = System.currentTimeMillis();

    // 처리 큐에 자리가 있으면 처리큐로, 없으면 대기 큐로 들어갑니다.
    redisQueueRepository.addToQueue(token, userId, score);
    return token;
}
```

### 스케줄러를 통한 대기열 갱신
- 일정 시간마다 처리 큐의 빈 공간을 찾아 대기 큐에서 순서대로 꺼내 넣어줍니다.
- score를 통해 정렬된 순서대로 가져올 수 있습니다.
```java
@Scheduled(fixedRate = 30000)
public void updateQueueStatus() {
    Long availableProcessingRoom = calculateAvailableProcessingRoom();
    if (availableProcessingRoom <= ZERO) return;

    Set<String> tokensNeedToUpdateToProcessing =
            redisQueueRepository.getWaitingQueueNeedToUpdateToProcessing(availableProcessingRoom.intValue());
    for (String token : tokensNeedToUpdateToProcessing) {
        redisQueueRepository.updateToProcessingQueue(
                token,
                System.currentTimeMillis() + FIFTEEN_MINUTE
        );
    }
}
```

### 결과
- 모든 요청 앞에서 해당 사용자가 처리 큐에 존재하는지 확인하여 요청을 처리합니다.
- 요청 처리 속도를 고려해 Processing Queue의 사이즈와 스케줄링 주기 조절 가능
- 집중적으로 몰리는 트래픽을 보다 안정적으로 처리가능해졌습니다.
