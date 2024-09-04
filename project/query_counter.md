## AOP 쿼리 카운터를 통한 쿼리 발생 횟수 추적
### 문제 상황

- 운영 서버에서 쿼리 발생 횟수에 대한 추적 필요
- 모든 쿼리를 전부 로그로 출력해서 카운팅하는 것은 현실적으로 불가능

### 스프링 AOP를 활용

### 포인트 컷

AOP의 포인트 컷으로 지정되기 위해서는 스프링 bean에서 관리하는 싱글톤 객체여야 합니다. \
기본 Connection 객체는 싱글톤이 아니지만, DataSource는 Bean에 등록 되어 있고, getConnection() 메서드를 통해 Connection 객체를 얻기 때문에 해당 시점에 포인트 컷을 설정해주었습니다.
```java
@Pointcut("execution(* javax.sql.DataSource.getConnection(..))")
public void performancePointcut() {
}
```

### 어드바이스

**동적 프록시 활용** \
원본 객체에 원하는 기능을 추가하기 위해 프록시를 사용하는데, 프록시 객체를 직접 생성하기 위해선 모든 메서드를 구현해주어야 합니다. 하지만 동적 프록시를 사용하면 런타임시에 프록시 객체를 만들어주기 때문에 간편하게 프록시를 사용할 수 있습니다.

**HTTP 요청 정보 로깅** \
RequestContextHolder의 getRequestAttributes() 메서드를 활용해서 현재 로그가 발생된 HTTP 요청 정보를 로깅합니다.

**RequestContextHolder** \
해당 클래스는 HTTP Request의 정보를 담아주는 객체로, Servlet이 생성될 때 초기화 된다. \
즉, HTTP Request 가 들어올 때 생성되고, servlet이 destory 될 때 함꼐 clean 된다. \
더 많은 정보를 가져오기 위해 ServletRequestAttributes 로 캐스팅해서 사용할 수 있다.

```java
@Around("performancePointcut()")
public Object start (ProceedingJoinPoint point) throws Throwable {
  RequestAttributes requestAttributes = RequestContextHolder.getRequestAttributes();
  if (requestAttributes instanceof ServletRequestAttributes) {
    HttpServletRequest request = ((ServletRequestAttributes) requestAttributes).getRequest();
    String httpMethod = request.getMethod();
    String requestURI = request.getRequestURI();

    log.info("HTTP Method: {}, Request URI: {}", httpMethod, requestURI);
  } else {
    log.info("No active HTTP request context available.");
  }

  final Connection connection = (Connection) point.proceed();
  queryCounter.set(new QueryCounter());
  final QueryCounter counter = this.queryCounter.get();

  final Connection proxyConnection = getProxyConnection(connection, counter);
  queryCounter.remove();
  return proxyConnection;
}

private Connection getProxyConnection(Connection connection, QueryCounter counter) {
  return (Connection) Proxy.newProxyInstance(
          getClass().getClassLoader(),
          new Class[]{Connection.class},
          new ConnectionHandler(connection, counter)
  );
}
```

### 다이나믹 프록시

다이나믹 프록시는 `Java java.lang.reflect.Proxy package`에서 제공해주는 API를 이용하여 손쉽게 만들 수 있습니다. \
준비물은 총 세가지입니다. \
클래스로더, 원 타겟 클래스(인터페이스), 동적으로 프록시를 설정할 ConnectionHandler 입니다.

이중 ConnectionHandler 를 상속받아서 필요한 기능을 추가했습니다.

```java
@Slf4j
public class ConnectionHandler implements InvocationHandler {

  private final Object target;
  private final QueryCounter counter;

  public ConnectionHandler(Object target, QueryCounter counter) {
    this.target = target;
    this.counter = counter;
  }

  @Override
  public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
    countPrepareStatement(method);
    logQueryCount(method);
    return method.invoke(target, args);
  }

  private void logQueryCount(Method method) {
    if (method.getName().equals("close")) {
      warnTooManyQuery();
      log.info("====== 발생한 쿼리 수 : {} =======\n", counter.getCount());
    }
  }

  private void countPrepareStatement(Method method) {
    if (method.getName().equals("prepareStatement")) {
      counter.increase();
    }
  }

  private void warnTooManyQuery() {
    if (counter.isWarn()) {
      log.warn("======= Too Many Query !!!! =======");
    }
  }
}
```

Connection 객체가 `prepareStatement()` 메서드를 호출 할 때 카운트를 1씩 올리고, \
`close()` 메서드가 호출 될 때 지금까지 쌓인 카운트를 출력하고 종료합니다. \
이떄, 만약 10개 이상의 쿼리가 발생하는 경우에는 추가로 경고 로그를 남깁니다.

### QueryCounter

```java
@Getter
public class QueryCounter {

  private int count;

  public void increase() {
    count++;
  }

  public boolean isWarn() {
    return count > 10;
  }
}
```

### 결과
<img src="https://tech-blog-image.s3.ap-northeast-2.amazonaws.com/image/7d36dc93-abfe-41a0-8d55-d230ac62e0abec2_counter.png" alt="이미지" width="777px"/>

- Connection 객체를 사용하는 모든 요청에 쓰레드를 할당
- 쓰레드가 할당된 **요청 내에 발생하는 쿼리의 수 추적**
- 10개 이상의 쿼리 발생시 경고 로그 출력


