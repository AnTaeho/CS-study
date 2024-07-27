# 람다 (+ 함수형 인터페이스)
## 람다식이란?
람다식이란 함수를 하나의 식으로 표현한 것
```java
//기존 메서드 표현 방식
void sayhello() {
	System.out.println("HELLO!")
}

//위의 코드를 람다식으로 표현한 식
() -> System.out.println("HELLO!")
```

```java
int sum (int x, int y) {
    return x + y ;
}
    
 ==람다식으로 변환=>
(x, y) -> x + y
```
기존 함수와 다르게 메소드 명이 필요하지 않고, 괄호와 ->를 이용해 함수를 선언하게 된다. \ 
람다식은 불필요한 코들르 줄이고 가독성을 올리기 위해 등장했다.

### 특징
1. 람다식 내에서 사용되는 지역변수는 final이 붙지 않아도 상수로 간주된다. 
2. 람다식으로 선언된 변수명은 다른 변수명과 중복될 수 없다.

### 장점
1. 코드를 간결하게 만들 수 있다. 
2. 식에 개발자의 의도가 명확히 드러나 가독성이 높아진다. 
3. 함수를 만드는 과정없이 한번에 처리할 수 있어 생산성이 높아진다. 
4. 병렬프로그래밍이 용이하다.

## 함수형 인터페이스란?
자바는 순수 함수와 일반 함수를 다르게 취급하고 있으면 자바에서는 이를 구분하기 위해 함수형 인터페이스가 등장하였다. \
함수형 인터페이스란 함수를 1급 객체처럼 다룰 수 있게 해주는 어노테이션으로 인터페이스에 선언하여 \ 
단 하나의 추상 메소드만을 갖도록 제한하는 역할을 한다.

## Java 에서 제공하는 함수형 인터페이스
### 1. Supplier
매개 변수 없이 반환값만을 갖는 함수형 인터페이스
```java
@FunctionalInterface
public interface Supplier<T> {
    T get();
}
```

### 2. Consumer
Consumer는 객체 T를 매개변수로 받고, 반환값은 없다.
```java
@FunctionalInterface
public interface Consumer<T> {
    void accept(T t);
}
```

### 3. Function<T, R>
Function은 객체 T를 매개변수로 받아서 처리한 후에 R로 반환한다.
```java
@FunctionalInterface
public interface Function<T, R> {
    R apply(T t);
}
```

### 4. Predicate
Predicate는 객체 T를 매개변수로 받아 처리한 후 boolean을 반환한다.
```java
@FunctionalInterface
public interface Predicate<T> {
    boolean test(T t);
}
```

## 메서드 참조
함수형 인터페이스를 람다식이 아닌 일반 메소드를 참조시켜 선언하는 방법 \ 
참조 가능한 메서드는 일반 메서드, static, 생성자가 있고, '클래스::메서드이름' 형식으로 참조한다.

### 일반 메서드 참조
```java
Function<String, Integer> function = String::length;
```

### Static 메서드 참조
```java
Predicate<Boolean> predicate = Objects::isNull;
```

### 생성자 참조
```java
Supplier<String> supplier = String::new;
```



