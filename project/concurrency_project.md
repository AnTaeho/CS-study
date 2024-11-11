## 동시성 문제 해결

### 문제 상황
![스크린샷 2024-11-04 오후 2 37 22](https://github.com/user-attachments/assets/eeffbdbf-6fe8-4548-b59f-4606cb27b006)


- 선착순 이벤트의 경우 동시에 많은 사용자가 접근하면 **감소 수량과 발급 수량이 일치하지 않는 정합성 문제** 발생
- 감소 수량과 발급 수량의 정합성을 보장하는 방법이 필요하다.

### 해결 방법

### 분산락 활용
```java
public class DistributedLockAspect {

    private final RedisSimpleLock redisSimpleLock;

    @Around("@annotation(com.example.e_commerce.aop.lock.DistributedLock)")
    public Object around(ProceedingJoinPoint joinPoint) throws Throwable {

        MethodSignature signature = (MethodSignature) joinPoint.getSignature();
        var method = signature.getMethod();
        DistributedLock distributedLock = method.getAnnotation(DistributedLock.class);

        String lockKey = distributedLock.key();
        String lockValue = UUID.randomUUID().toString();

        try {
            boolean acquired = redisSimpleLock.tryLock(
                    lockKey,
                    lockValue,
                    distributedLock.leaseTime(),
                    distributedLock.timeUnit()
            );
            if (!acquired) {
                throw new IllegalArgumentException("락을 얻을 수 없습니다.");
            }
            return joinPoint.proceed();
        } finally {
            redisSimpleLock.releaseLock(lockKey, lockValue);
        }
    }
}
```

![스크린샷 2024-11-04 오후 2 33 40](https://github.com/user-attachments/assets/13c04930-7c8f-4a08-b355-34a91d0da731)

- Spring AOP를 활용한 분산락을 통해서 해당 요청에 대한 락을 제어합니다.
### PESSIMISTIC_READ 비관적 락 적용
```java
@Lock(LockModeType.PESSIMISTIC_READ)
@Query("select p from Product p where p.id = :productId")
Optional<Product> findByIdWithLock(@Param("productId") Long productId);
```

- 해당 요청 이외의 요청에서 자원을 수정하는 것을 방지하기 위해 PESSIMISTIC_READ 옵션의 비관적 락 추가
- **분산 락과 비관적 락을 2중으로 활용**해서 안정적으로 동시성 문제를 해결할 수 있다.

### 결과

- 동시에 많은 요청이 발생하더라도 **정확한 수량만큼 쿠폰 발급**
- 락은 사용하는 방식은 트랜잭션의 점유가 늘어나 **성능 저하의 문제**는 남아있다.
