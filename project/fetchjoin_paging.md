## JPA 2-depth 연관관계와 페이지네이션이 얽힌 N+1 문제 해결

### 문제 상황

### 연관관계
<img src="https://tech-blog-image.s3.ap-northeast-2.amazonaws.com/image/225b2740-873d-42a1-b5dc-e8ba0990ecf3posttag.png" alt="이미지" style="max-width: 100%; height: auto; display: block; margin: 0 auto;"/>

- Post-PostTag는 일대다 양방향 관계, PostTag-Tag는 다대일 양방향 관계
- DTO로 변환 과정에서 N+1 문제가 발생했으나 페이지네이션으로 인해 fetch join을 사용할 수 없는 문제 발생

기본적으로 JPA에서는 1:N fetch join과 페이지네이션을 동시에 처리할 수 없습니다. \
만약 동시에 사용한다면 페치 조인만 쿼리로 진행한 뒤에 메모리에서 직접 페이징 합니다.

이렇게 처리할 경우 속도가 매우 느려지고, 최악의 경우 OOM (Out of Memory) 문제를 일으킬 수 있습니다. \
이런 문제들을 해결하기 위해 여러가지 옵션들을 통해 최적의 방법을 찾았습니다.

### 해결 과정

### 1. fetch join + 페이징

- **소요시간 : OOM (Out Of Memory 발생) / 총 쿼리 : 2개** → 1 (카운트 쿼리) + 1 (본 쿼리)

실행되는 쿼리의 수는 적지만 20만개 데이터 기준으로 Out of memory 예외가 발생했습니다.

### 2. fetch join X + 페이징

- **소요시간 : 약 3초 / 총 쿼리 : 107개** -> 1 (카운트 쿼리) + 1 (본 쿼리) + 20 (PostTag N+1) + 85 (Tag N+1)

속도는 빨라지지만 조인 처리를 전혀 하지 않기 때문에 가능한 모든 곳에서 N+1 문제가 발생합니다. \
실제 데이터가 적재되기 시작하면 훨씬 무거운 쿼리가 발생할 수 있습니다.

### 3. fetch join X + 페이징 + BatchSize

- **소요시간 : 약 3초 / 총 쿼리 : 4개** → 1 (카운트 쿼리) + 1 (본 쿼리) + 1 (PostTag) + 1(Tag)

BatchSize로 인해서 쿼리가 줄어들었지만 여전히 매우 느린 응답 속도를 보여줍니다.

하지만 이 시점에서  카운트 쿼리 실행이 무겁다는 점을 인지하였고, 카운트 쿼리 최적화를 진행했습니다 

### 4. fetch join X + 페이징 + BatchSize

- **소요시간 : 약 0.2초**  
- **총 쿼리 : 4개** → 1 (카운트 쿼리) + 1 (본 쿼리) + 1 (PostTag) + 1(Tag)

쿼리 카운터를 최적화 하니 훨씬 빨라졌고, 쿼리의 수는 그대로 입니다.

### 5.Sub Query Paging + fetch join

- **소요시간 : 약 0.2 초**
- **총 쿼리 : 3개** → 1개 (카운트 쿼리) + 1개 (본 쿼리) + 1개 (서브 쿼리)

먼저 조건에 맞는 postId를 페이징을 적용해서 전부 조회, 커버링 인덱스를 활용해 성능 최적화 \
실제로 필요한 정보를 해당 PostId 리스트를 조건에 넣어 검색

4번 케이스의 결과와 5번 케이스의 결과가 거의 비슷해서 추가적으로 테스트를 진행한 결과 \
5번 케이스의 응답 속도가 약 5 ~ 10% 더 빠르다는 결과를 얻어 최종적으로 5번 케이스의 코드를 적용했습니다. \
하지만 5번의 경우엔 본 쿼리가 많은 정보를 가져오기 때문에 더 복잡한 연관관계로 발전할 경우에 성능 저하가 발생할 수 있습니다. 

### 최종 코드
```java
@Override
public Page<PostSimpleResponse> searchByIds(List<Long> ids, Pageable pageable) {
    List<Long> postIds = queryFactory
            .select(post.id)
            .from(post)
            .where(post.id.in(ids))
            .orderBy(post.id.desc())
            .offset(pageable.getOffset())
            .limit(pageable.getPageSize())
            .fetch();

    if (postIds.isEmpty()) {
        return new PageImpl<>(Collections.emptyList(), pageable, 0);
    }

    List<Post> posts = queryFactory
            .selectFrom(post)
            .leftJoin(post.writer, user).fetchJoin()
            .leftJoin(post.postTags, postTag).fetchJoin()
            .leftJoin(postTag.tag, QTag.tag).fetchJoin()
            .where(post.id.in(postIds))
            .orderBy(post.id.desc())
            .fetch();

    long total = queryFactory
            .select(post.id)
            .from(post)
            .where(post.id.in(ids))
            .fetch().size();

    List<PostSimpleResponse> result = posts.stream()
            .map(this::toSimpleResponse)
            .toList();

    return new PageImpl<>(result, pageable, total);
}
```

위 코드가 만들어 내는 실제 쿼리는 아래와 같습니다.
```java
SELECT post.*
FROM post
LEFT JOIN user AS writer ON post.writer_id = writer.id
LEFT JOIN post_tag ON post.id = post_tag.post_id
LEFT JOIN tag ON post_tag.tag_id = tag.id
WHERE post.id IN (
        SELECT post.id 
        FROM post 
        WHERE post.id IN (조건에 맞는 PostId 리스트)
        ORDER BY 
            post.id DESC 
        LIMIT (전체 사이즈) OFFSET 20;
    )
ORDER BY post.id DESC;
```

### 결과

- 서브 쿼리와 페치 조인의 적절한 조합으로 정보를 최소한의 쿼리로 조회하고 응답 속도 향상
- 1+1+N+N 개 → **1+1+1 개의 쿼리로 최적화**
- **소요 시간 Out of Memory → 0.2초로 최적화**
