## 스트림 API
스트림 API는 자바에서 컬렉션을 처리하는 강력한 도구로서 가독성 좋은 코드로 컬렉션에 대한 복잡한 작업을 수행할 수 있다.

### 주요 기능
- 함수형 프로그래밍 : 스트림은 함수형 프로그래밍 기술을 사용하도록 설계되어서 람다나 메서드 참조를 활용할 수 있다
- 병렬 처리 : 병렬 처리가 가능하도록 설계되어 멀티 코어 프로세서를 활용해서 처리 속도를 높일 수 있음
- 지연 연산 : 필요할 때만 요소를 평가해서 메모리 사용량을 줄이고 성능을 향상시킨다.

### 특징
- 원본의 데이터를 직접 변경하지 않는다. 원본 데이터를 읽기만 하고 스트림에 담아서 여러 작업을 처리한다.
- 일회용이다. 재사용이 불가능하다.
- `filter()` 와 `map()`을 활용하여 지연 연산을 최적화 한다.
- `parallelStream()`을 통해 손쉬운 병렬처리를 지원한다.

### 스트림 API의 3단계

1. 생성하기
 - Stream 객체를 생성한다.
 - 재사용이 불가능 하므로 닫히면 다시 생성해야한다.
2. 가공하기
 - 원본의 데이터를 별도의 데이터로 가공하기 위한 중간 연산
 - 연산 결과를 Stream으로 다시 반환하기 때문에 중간 연산을 이어갈 수 있다.
3. 결과 만들기
 - 가공된 데이터로부터 원하는 결과를 만들기 위한 최종 연산
 - Stream 요소들을 소모하면서 연산이 수행되기 때문에 1번만 처리가능하다.

```java
List<String> myList = Arrays.asList("a1", "a2", "b1", "c2", "c1");

mylist.stream()
        .filter(it -> it.startWith("c"))
        .map(String::toUpperCase)
        .sorted()
        .count();
```
### 스트림의 중간 연산
1. 스트림 필터링
- `filter()` : 조건에 맞는 요소만으로 구성된 새로운 스트림 반환
- `distinct()` : equals() 메소드를 사용하여 중복 요소 제거

2. 스트림 변환
- `map()` : 요소들을 주어진 함수에 인수로 전달하여, 그 반환값들로 이루어진 새로운 스트림 반환
- `flatMap()` : 각 배열의 각 요소의 반환값을 하나로 합친 새로운 스트림을 얻을 수 있음.

3. 스트림 제한
- `limit()` : 전달된 개수만큼의 요소만으로 이루어진 새로운 스트림 반환
- `skip()` : 전달된 개수만큼의 요소를 제외한 나머지 요소만으로 이루어진 새로운 스트림 반환

4. 스트림 정렬
- `sorted()` : 주어진 조건으로 정렬하고 전달하지 않으면 기본적으로 오름차순 정렬

5. 스트림 연산 결과 확인
- `peek()` : 연산과 연산 사이에 결과를 확인하고 싶을 때 사용.

### 스트림의 최종 연산

1. 요소의 출력
- `forEach()` : 스트림의 각 요소를 소모하여 명시된 동작 수행. 반환 타입이 void 이므로 스트림의 모든 요소를 출력하는 용도로 많이 사용

2. 요소의 소모
- `reduce()` :  Stream의 요소들을 하나의 데이터로 만드는 작업을 수행

3. 요소의 검색
- `findFirst()`, `findAny()` : 첫 번째 요소를 참조하는 Optional 객체 반환. 병렬 스트림일 경우 findAny() 메소드를 사용해야만 정확한 연산 결과 반환.

4. 요소의 검사
- `anyMatch()` : 스트림의 일부 요소가 특정 조건을 만족할 경우 true 반환
- `allMatch()` : 스트림의 모든 요소가 특정 조건을 만족할 경우 true 반환
- `noneMatch()` : 스트림의 모든 요소가 특정 조건을 만족하지 않을 경우 true 반환

5. 요소의 통계
- `count()` : 스트림의 요소의 총 개수를 long타입으로 반환
- `max()`, `min()` : 스트림의 요소 중 가장 큰 값과 가장 작은 값을 참조하는 Optional 객체 반환

6. 요소의 연산
- `sum()`, `average()` : IntStream, DoubleStream과 같은 기본 타입 스트림에는 모든 요소에 대해 합과 평균을 구할 수 있는 메소드가 정의되어 있음. average() 메소드는 각 기본 타입으로 래핑 된 Optional 객체 반환

7. 요소의 수집

`collect()` 메소드는 인수로 전달되는 Collectors 객체에 구현된 방법대로 스트림의 요소를 수집

- 스트림을 배열이나 컬렉션으로 변환 : toArray(), toCollection(), toList(), toSet(), toMap()
- 요소의 통계와 연산 메소드와 같은 동작 수행 : counting(), maxBy(), minBy(), summingInt(), averagingInt() 등
- 요소의 소모와 같은 동작을 수행 : reducing(), joining()
- 요소의 그룹화와 분할 : groupingBy(), partitioningBy()