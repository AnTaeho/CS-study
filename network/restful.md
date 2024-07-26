## REST
REST (Representational State Transfer)란 자원을 이름으로 구분하여 해당 자원의 상태를 주고 받는 것을 의미

HTTP URI를 통해 자원을 명시하고, HTTP Method를 통해 해당 자원에 대한 CRUD Operation을 적용하는 것을 의미합니다.

### CRUD Operation
- Create, Read, Update, Delete 를 묶어서 일컫는 말
- Create : 데이터 생성 (POST)
- Read : 데이터 조회 (GET)
- Update : 데이터 수정 (PUT, PATCH)
- Delete : 데이터 삭제 (DELETE)

## REST의 구성 요소
- 자원 : HTTP URI
- 행위 : HTTP Method
- 행위에 대한 내용 : HTTP Message Pay Load

### REST의 특징
1. Server-Client(서버-클라이언트 구조)
2. Stateless(무상태)
3. Cacheable(캐시 처리 가능)
4. Layered System(계층화)
5. Uniform Interface(인터페이스 일관성)

## REST API

REST API(Representational State Transfer API)란 REST 스타일을 따르는 API를 의미합니다.

### 설계 예시
1. URL는 명사, 소문자를 사용해야 한다.

```
❌ http://test.com/Studying/
⭕️ http://test.com/study/
```

2. 파일 확장자는 URI에 포함하지 않는다.
```
❌ http://test.com/photo.png
⭕️ http://test.com/photo
```
3. 행위를 포함하지 않는다.
```
❌ http://test.com/delete-post/1
⭕️ http://test.com/post/1
```

## RESTful
REST API의 설계 규칙을 올바르게 지킨 시스템을 RESTful 하다 말할 수 있다. \
REST 방식을 사용했더라도 모든 CRUD를 POST로 처리하는 등 규칙을 지키지 않으면 RESTful하지 못하다고 할 수 있습니다.

### RESTful API 클라이언트 요청 구성 요소
- **고유 리소스 식별자**  
  서버는 고유한 리소스 식별자로 각 리소스를 식별합니다. REST 서비스의 경우 서버는 일반적으로 URL을 사용하여 리소스 식별을 수행합니다. URL은 요청 엔드포인트 라고도 하며 클라이언트가 요구하는 사항을 서버에 명확하게 지정합니다.

- **메서드**   
  개발자는 HTTP를 사용하여 RESTful API를 구현합니다. HTTP Method는 리소스에 수행해야 하는 작업을 서버에 알려줍니다.

1. *GET*  
   클라이언트는 GET을 사용하여 URL의 리소스에 접근합니다. 

2. *POST*  
   클라이언트는 POST를 사용하여 서버에 데이터를 전송합니다. 동일한 POST를 여러 번 전송 할 경우 동일 리소스가 여러번 생성될 수 있는 주의점이 있습니다.

3. *PUT*  
   클라이언트는 PUT을 사용하여 서버의 기존 리소스를 전체 업데이트 합니다. POST와 달리, RESTful 웹 서비스에서 동일한 PUT 요청을 여러 번 전송해도 결과는 동일합니다. (멱등성)

4. *PATCH*
   클라이언트는 PUT을 사용하여 서버의 기존 리소스를 부분 업데이트 합니다. 멱등성을 보장하지 않는다.

5. *DELETE*  
   클라이언트는 DELETE 요청을 사용하여 리소스를 제거합니다. DELETE 요청은 서버 상태를 변경할 수 있습니다.

- **HTTP header**  
  header는 클라이언트와 서버 간 교환되는 메타데이터입니다. 예를 들어, 요청 헤더는 요청 및 응답의 형식을 나타내고 요청 상태 등에 대한 정보를 제공합니다.

### RESTful API 서버 응답 구성 요소

**응답 코드**  
상태 표시줄에는 요청 성공 또는 실패를 알리는 3자리 상태 코드가 존재합니다. 예를 들어, 2XX 코드는 성공을 의미하며, 4XX 및 5XX 코드는 오류를 나타냅니다. 3XX 코드는 URI 리디렉션을 나타냅니다.

다음은 일반적인 응답 코드 예시입니다.
- 200 : 일반 성공 응답
- 201 : POST 메서드 성공 응답
- 400 : 서버가 처리할 수 없는 잘못된 요청
- 404 : 리소스를 찾을 수 없음
- 5xx : 서버 문제

**메시지 본문**
응답 본문에는 리소스 표현이 포함됩니다. 클라이언트는 데이터 작성 방식을 XML 또는 JSON 방식으로 정보를 요청할 수 있습니다.

**헤더**  
이는 응답에 대한 추가 컨텍스트를 제공하고 서버, 인코딩, 날짜 및 콘텐츠 유형과 같은 정보를 포함합니다.
