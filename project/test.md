## 제어할 수 없는 코드 개선 및 테스트 작성
### 문제 상황

- 쿠폰의 유효기간 판단을 위해 LocalDate.now()를 사용해 날짜 비교
- 테스트 시점에 따라 결과에 영향를 주기 때문에 항상 일정한 결과를 얻기 힘들다

→ **제어할 수 없는 로직을 테스트 하는 방법을 탐색**

### 해결 방법

**1. 최대한 밖으로 밀어내기**

- Controller 계층으로 제어할 수 없는 코드을 옮기고 Service 계층에 인자 전달
- Service 계층에서는 정해진 날짜를 받을 수 있기 때문에 테스트시 문제 감소

→ **Controller 계층에 대한 테스트는 여전히 어렵고, 여전히 제어할 수 없는 인자 존재**

**2. 추상화와 IoC/DI**

<img src="https://tech-blog-image.s3.ap-northeast-2.amazonaws.com/image/b16315ea-7c83-41f0-8559-3acec1733a27test1.png" alt="이미지" style="max-width: 100%; height: auto; display: block; margin: 0 auto;"/>

- 현재 시간을 Time이라는 인터페이스로 추상화 해서 서비스 코드와 테스트 코드에서 구현 분리
- 실제 서비스의 OnServiceTime은 스프링의 컴포넌트 스캔을 통해 의존성 주입해서 사용
- 테스트 코드의 FakeTime은 직접 구현체를 주입해서 사용

### 코드

```java
interface Time {
    fun now(): LocalDate
}

@Component
class OnServiceTime: Time {
    override fun now(): LocalDate {
        return LocalDate.now()
    }
}

class FakeTime: Time {
    override fun now(): LocalDate {
        return LocalDate.of(2024, 12, 20)
    }
}
```

### 결론

- 서비스 클래스에서는 Time 인터페이스만 의존하면 필요에 따라 원하는 구현체를 사용할 수 있게 수정
- 테스트 코드용 구현체를 분리해서 제어할 수 없던 코드를 항상 의도된 값으로 동작하도록 수정
- 스프링의 IoC/DI 를 활용해서 서비스에 영향을 주지 않고 올바른 테스트 코드를 작성할 수 있게 됨