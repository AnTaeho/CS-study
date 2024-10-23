## Redis를 활용한 대기열 구현

### 문제 상황
- 수강신청의 경우 트래픽이 특정 시간대에 집중
- 예상치 못한 수준의 트래픽에 서버가 다운될 수도 있다고 판단
- 실제로 서버에 요청하는 트래픽의 수를 제한해서 트래픽 부하를 방지할 수 있다고 판단

### 해결 방법

![스크린샷 2024-10-02 오후 6 26 12](https://github.com/user-attachments/assets/9295177d-007b-41d3-8e0b-9fd05228976a)


### Redis ZSet을 활용한 대기열 구현
- Redis의 Sorted Set (ZSet)을 활용해서 대기 큐와 처리 큐를 구현했습니다.
- 현재 시간을 기준으로 순서를 보장합니다.
- 처리 큐의 사이즈를 제한하고, 그 이상의 사용자에 대해선 대기 큐에 순서대로 적제합니다.

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
- 실제 서비스에 요청을 하는 사용자를 1000명을 제한
- 집중적으로 몰리는 트래픽을 보다 안정적으로 처리가능해졌습니다.
