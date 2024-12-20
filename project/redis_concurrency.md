### 레디스 재고 관리와 비동기 이벤트 처리

### 문제 상황

- 동시성 이슈를 해결했지만 순간적으로 발생하는 **대량의 트래픽에 대한 대응 미흡**
- 락을 사용하면 트랜잭션의 점유가 길게 발생하기 때문에 성능 저하 발생
- 락을 활용하지 않고 해결하는 방법이 필요

### 해결 방법
![스크린샷 2024-11-11 오전 10 15 53](https://github.com/user-attachments/assets/5d5251dc-07d1-4547-bf62-e205573fde74)

- 락을 사용하지 않고 싱글 스레드 기반의 **Redis에 재고 데이터를 서빙**해서 관리
- 최초에 데이터를 서빙하고 decr 명령어를 통해 재고를 차감하는 방식 사용
- **Redis Transaction**을 활용해 지급 조회와 쿠폰 지급을 하나의 트랜잭션 내에서 동작하도록 구현
- DB Write 연산은 이벤트 비동기 처리를 통해 DB 부하 분산

### 결과

- 락 없이 레디스의 Set과 decr 명령어를 통해 동시성 이슈를 제어
- 중요한 쓰기 연산은 해당 트랜잭션에서 분리되어 **응답 속도 증가 및 부하 감소**
