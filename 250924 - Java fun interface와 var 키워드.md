

## fun interface (함수형 인터페이스)

### 개요
Java 8부터 도입된 함수형 인터페이스는 정확히 하나의 추상 메서드를 가진 인터페이스다.

### @FunctionalInterface 어노테이션
```java
@FunctionalInterface
public interface MyFunction {
    int apply(int x, int y);

    // default 메서드는 여러 개 가능
    default void print() {
        System.out.println("Function");
    }
}
```

### 주요 내장 함수형 인터페이스
```java
// Function<T, R> - 입력을 받아 출력 반환
Function<String, Integer> lengthFunc = String::length;

// Consumer<T> - 입력을 받아 소비 (void)
Consumer<String> printer = System.out::println;

// Supplier<T> - 입력 없이 값 생성
Supplier<String> supplier = () -> "Hello";

// Predicate<T> - 입력을 받아 boolean 반환
Predicate<String> isEmpty = String::isEmpty;
```

### 람다 표현식과의 관계
```java
// 람다 표현식
MyFunction func = (x, y) -> x + y;

// 메서드 참조
Function<String, Integer> length = String::length;
```

---

## var 키워드

### 개요
Java 10부터 도입된 `var` 키워드는 지역 변수의 타입 추론을 가능하게 한다.

## 기본 사용법

### 타입 추론
```java
// 기존 방식
String name = "Hello";
List<String> list = new ArrayList<>();

// var 사용
var name = "Hello";           // String으로 추론
var list = new ArrayList<String>(); // ArrayList<String>으로 추론
var number = 10;              // int로 추론
```

## 사용 제한사항

### 1. 지역 변수에서만 사용 가능
```java
public class Example {
    // var field;        // 컴파일 에러
    // static var field; // 컴파일 에러

    public void method() {
        var local = "OK";    // 가능
    }
}
```

### 2. 초기화 필수
```java
var x;           // 컴파일 에러 - 초기화 필요
var y = null;    // 컴파일 에러 - 타입 추론 불가
```

### 3. 메서드 매개변수, 리턴 타입 불가
```java
// public var method(var param) { } // 컴파일 에러
```

## 적절한 사용 예시

### 1. 복잡한 제네릭 타입
```java
// 기존
Map<String, List<Integer>> map = new HashMap<String, List<Integer>>();

// var 사용
var map = new HashMap<String, List<Integer>>();
```

### 2. 반복문
```java
// 향상된 for문
var list = List.of("a", "b", "c");
for (var item : list) {
    System.out.println(item);
}

// try-with-resources
try (var scanner = new Scanner(System.in)) {
    // ...
}
```

### 3. 스트림 API
```java
var result = list.stream()
    .filter(s -> s.length() > 3)
    .map(String::toUpperCase)
    .collect(Collectors.toList());
```

## 주의사항

### 1. 가독성 고려
```java
// 명확한 경우
var name = "John";
var count = 10;

// 불명확한 경우 - 피해야 함
var result = someMethod(); // 리턴 타입이 명확하지 않음
```

### 2. 다이아몬드 연산자와 함께 사용 시
```java
var list = new ArrayList<>(); // 컴파일 에러 - 타입 추론 불가
var list = new ArrayList<String>(); // 올바른 사용
```

## 장점과 단점

### 장점
- 코드 간결성
- 타입 안정성 유지
- 리팩토링 용이성

### 단점
- 가독성 저하 가능성
- IDE 의존성 증가
- 타입이 명확하지 않을 때 혼란

## 권장사항
- 타입이 명확하게 추론 가능한 경우에만 사용
- 복잡한 제네릭 타입에서 유용
- 코드 리뷰 시 가독성 고려