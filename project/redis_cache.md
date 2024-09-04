## Redis를 활용한 메인 페이지 캐싱

### 문제 상황

메인 페이지에서 제공하는 전체 글 목록의 경우 거의 변경사항이 없고, 가장 많이 호출되는 API 입니다.
하지만 가장 많은 데이터를 호출하고, 가장 많은 정보를 담기때문에 모든 호출에 대해 항상 DB에서 정보를 가져온다면 서버에 부하가 발생할 수 있습니다.
따라서 메인 페이지 정보를 캐시하기로 결정

ehcache와 redis를 비교한 결과 약 30% 더 성능이 좋은 redis를 선택해서 적용했습니다. \
추가적으로 nGrinder를 활용한 부하테스트를 통해 캐시 적용 전 후의 성능차이를 수치화 하여 정리하였습니다.

### 해결 방법

```java
public Page<PostSimpleResponse> findAllPost(Pageable pageable) {
  int pageNumber = pageable.getPageNumber();
  if (pageNumber > 4) {
    return postRepository.getPostPageWithWriterPage(pageable);
  }

  RestPage<PostSimpleResponse> cachedPosts = (RestPage<PostSimpleResponse>) redisTemplate.opsForValue().get(MAIN_PAGE_CACHE_KEY + pageNumber);
  if (cachedPosts != null) {
    log.info("Using cached data page: {}", pageNumber);
    return cachedPosts;
  }

  RestPage<PostSimpleResponse> postPageWithWriterPage = postRepository.getPostPageWithWriterPage(pageable);
  redisTemplate.opsForValue().set(MAIN_PAGE_CACHE_KEY + pageNumber, postPageWithWriterPage, CACHE_EXPIRATION, TimeUnit.SECONDS);
  return postPageWithWriterPage;
}
```

캐시를 확인해서 Cache hit 시에는 캐시에 저장된 값을 반환하고, miss의 경우엔 값을 찾아서 캐시에 저장하고 반환합니다.

현재 사용자가 본인 1명으로 적기 때문에 캐시의 유효시간은 1시간으로 설정해 두었고, 게시글 작성/수정/삭제시에 캐시도 함께 최기화 됩니다. 실제 사용자가 많아 진다면 유효시간과 삭제 정책을 조절할 수 있습니다.

### 성능 개선 테스트
nGrinder를 활용했고, Vuser는 10, 테스트 시간은 1분으로 진행했습니다.

1 ~ 10페이지 (pageNumber로는 0~9)를 랜덤하게 호출하는 방식으로 테스트 했습니다. 스크립트는 아래에 토글로 남겨두겠습니다.

### 스크립트
```java
@Test
public void test() {
	
    def randomPage = new Random().nextInt(7).toString()
		
    HTTPResponse response = request.GET("http://127.0.0.1:8081/posts", ["page" : randomPage, "size" : "20"])

    if (response.statusCode == 301 || response.statusCode == 302) {
        grinder.logger.warn("Warning. The response may not be correct. The response code was {}.", response.statusCode)
    } else {
        assertThat(response.statusCode, is(200))
    }
}
```

### 1 페이지 단일 캐시 테스트

우선, 1페이지만을 기준으로 캐싱 했을 때와 캐싱하지 않았을 때를 비교했습니다. \
단일 페이지 캐싱의 경우에는 적용 전후가 극단적으로 차이나고, 실제 서비스에서 호출 되는 비율과 큰 괴리가 있다고 생각했습니다.

실제 서비스에선 자주 호출되는 초반 1 ~ 5 페이지까지 캐싱하기로 결정했습니다.

<img src="https://tech-blog-image.s3.ap-northeast-2.amazonaws.com/image/87f37254-c59a-4145-8d87-004828e227e8page11.png" alt="이미지" style="max-width: 100%; height: auto; display: block; margin: 0 auto;"/>
<img src="https://tech-blog-image.s3.ap-northeast-2.amazonaws.com/image/55a50a90-343e-4b82-a344-00139b456c74page12.png" alt="이미지" style="max-width: 100%; height: auto; display: block; margin: 0 auto;"/>


### `수치 변화`

- 평균 TPS : { 6.4 } → { 2,118.7 } **331배 증가**
- Peek TPS : { 11 } → { 2877 } **262배 증가**
- Mean Test Time : { 1,583.19 } ms → { 4.19 }ms **99.7% 감소**
- Exected Tests : { 358 } → { 123,574 } **345배 증가**

### 1 ~ 10 페이지 호출 테스트 (1~5 페이지 캐싱)

1 ~ 5 페이지 캐싱 적용 전후로 1 ~ 10 페이지 사이의 랜덤 페이지를 호출하는 테스트를 진행했습니다. 

<img src="https://tech-blog-image.s3.ap-northeast-2.amazonaws.com/image/abf90e7e-c152-4498-b903-15592ca044bdpage110.png" alt="이미지" style="max-width: 100%; height: auto; display: block; margin: 0 auto;"/>
<img src="https://tech-blog-image.s3.ap-northeast-2.amazonaws.com/image/c1212a7a-23ac-4f60-ab9e-1872351c6270page1102.png" alt="이미지" style="max-width: 100%; height: auto; display: block; margin: 0 auto;"/>


### `수치 변화`

- 캐시 히트율 : **50% (5/10)**
- 평균 TPS : { 13.7 } → { 24.7 } **44% 증가**
- Peek TPS : { 16.5 } → { 42 } **155% 증가**
- Mean Test Time : { 720.88 } ms → { 394.33 } ms **45% 감소**
- Exected Tests : { 799 } → { 1,393 } **74% 증가**
-
### 1 ~ 7 페이지 호출 테스트 (1~5 페이지 캐싱)

~ 5 페이지 캐싱 적용 전후로 1 ~ 7 페이지 사이의 랜덤 페이지를 호출하는 테스트를 진행했습니다. \
실제 서비스의 경우에도 이와 비슷한 비율로 캐시 히트/미스가 발생할 것이라고 예상했습니다.

<img src="https://tech-blog-image.s3.ap-northeast-2.amazonaws.com/image/4c9d2c9e-b8e6-419d-8d00-0bccabce11e9page171.png" alt="이미지" style="max-width: 100%; height: auto; display: block; margin: 0 auto;"/>
<img src="https://tech-blog-image.s3.ap-northeast-2.amazonaws.com/image/c55be845-dfe3-4fd5-957d-de4c40a7828cpage172.png" alt="이미지" style="max-width: 100%; height: auto; display: block; margin: 0 auto;"/>

### `수치 변화`

- 캐시 히트율 : **71% (5/7)**
- 평균 TPS : { 13.1 } → { 47.5 } **263% 증가**
- Peek TPS : { 16 } → { 77.5 } **384% 증가**
- Mean Test Time : { 761.44 } ms → { 204.09 }ms **73% 감소**
- Exected Tests : { 734 } → { 2,770 } **277% 증가**

### 결과
- 1~7 사이의 랜덤 페이지 호출로 부하테스트시 캐시 적용시
  - 평균 TPS **260% 상승**, 요청 처리 시간 **73% 감소**
- 1~10 사이의 랜덤 페이지 호출로 부하테스트시 캐시 적용시
  - 평균 TPS **44% 상승**, 요청 처리 시간 **45% 감소**
- 모든 테스트는 nGrinder를 활용
  - Vuser : 10, TestTime : 1m
  - 더미 데이터 20만개 기준