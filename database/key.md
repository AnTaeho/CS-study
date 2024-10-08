## 키 (Key)

데이터베이스에서 조건에 만족하는 레코드를 찾거나 정렬할 대 기준이 되는 속성

### 사전 지식
### 무결성
데이터의 정확성, 일관성, 유효성이 유지되는 것을 의미

### 개체 무결성
모든 테이블이 기본키로 선택된 필드를 가져야 한다 \
기본키는 NULL 값을 허용하지 않고, 중복을 허용하지 않는다.
### 참조 무결성
모델간의 참조 관계에 있는 두 테이블의 데이터가 항상 일관된 값을 갖도록 유지한다.

### 도메인 무결성
필드의 무결성을 보장하기 위해 필드의 타입, NULL 값 허용등의 사항을 정의하고 올바르게 입력되었는지 확인한다.

### 유일성

**하나의 키값으로 레코드를 식별할 수 있는 성질**
여러개의 레코드는 각각 유일해야하며 각각의 레코드를 구분할 수 있는 속성이 필요하다.

### 최소성
**키를 구성하는 속성들 중 꼭 필요한 최소한의 속성들로만 키를 구성하는 성질**


## 후보키 (Candidate Key)
- **유일성, 최소성**
- 릴레이션을 구성하는 속성들 중에서 튜플을 유일하게 식별할 수 있는 속성들의 부분집합을 의미
- 모든 릴레이션은 반드시 하나 이상의 후보키를 가져야합니다.

## 기본키 (Primary Key)
- **후보 키 중 선택된 키.** 릴레이션에서 기본 키는 반드시 유일해야 한다.
- 중복을 허용하지 않고 Null 값을 허용하지 않는다.

## 대체키 (Alternate key)
- 후보키가 둘 이상일 때 기본키를 제외한 나머지 후보 키

## 수퍼키 (Super Key)
- **유일성 (최소성 X)**
- 한 릴레이션 내에 있는 속성들의 집합으로 구성된 키

## 외래키 (Foreign key) 
- 관계를 맺고 있는 릴레이션간에 서로 참조하고 있는 다른 릴레이션의 기본키
- 참죄되는 릴레이션의 기본키와 대응되어 릴레이션 간에 참조 관계를 표현
