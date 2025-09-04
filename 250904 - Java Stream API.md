

## 개념

**Stream API**는 Java 8에서 도입된 함수형 프로그래밍 패러다임을 지원하는 API입니다. 컬렉션, 배열 등의 데이터를 함수형으로 처리할 수 있도록 도와주며, 선언적이고 간결한 코드 작성이 가능합니다.

### 주요 특징
- **함수형 프로그래밍**: 람다식과 메서드 참조 활용
- **지연 연산(Lazy Evaluation)**: 최종 연산이 호출될 때까지 중간 연산을 지연
- **파이프라이닝**: 여러 연산을 연결하여 처리
- **내부 반복**: 외부 반복 대신 내부에서 반복 처리

## Stream의 구조

```
데이터 소스 → 중간 연산 → 최종 연산 → 결과
```

### 1. 데이터 소스 (Stream 생성)
```java
// Collection에서 생성
List<String> list = Arrays.asList("a", "b", "c");
Stream<String> stream = list.stream();

// 배열에서 생성
String[] array = {"a", "b", "c"};
Stream<String> stream = Arrays.stream(array);

// 직접 생성
Stream<String> stream = Stream.of("a", "b", "c");

// 무한 스트림
Stream<Integer> infinite = Stream.iterate(0, n -> n + 1);
Stream<Double> random = Stream.generate(Math::random);
```

### 2. 중간 연산 (Intermediate Operations)
Stream을 변환하지만 지연 실행됩니다.

#### Filter - 조건에 맞는 요소 선택
```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6);
List<Integer> evens = numbers.stream()
    .filter(n -> n % 2 == 0)  // 짝수만 필터링
    .collect(Collectors.toList());
// 결과: [2, 4, 6]
```

#### Map - 요소 변환
```java
List<String> words = Arrays.asList("apple", "banana", "cherry");
List<Integer> lengths = words.stream()
    .map(String::length)  // 문자열 길이로 변환
    .collect(Collectors.toList());
// 결과: [5, 6, 6]
```

#### FlatMap - 중첩 구조 평면화
```java
List<List<Integer>> nested = Arrays.asList(
    Arrays.asList(1, 2),
    Arrays.asList(3, 4),
    Arrays.asList(5, 6)
);
List<Integer> flattened = nested.stream()
    .flatMap(List::stream)  // 중첩 리스트를 평면화
    .collect(Collectors.toList());
// 결과: [1, 2, 3, 4, 5, 6]
```

#### Distinct - 중복 제거
```java
List<Integer> numbers = Arrays.asList(1, 2, 2, 3, 3, 4);
List<Integer> unique = numbers.stream()
    .distinct()
    .collect(Collectors.toList());
// 결과: [1, 2, 3, 4]
```

#### Sorted - 정렬
```java
List<String> words = Arrays.asList("banana", "apple", "cherry");
List<String> sorted = words.stream()
    .sorted()  // 자연 순서로 정렬
    .collect(Collectors.toList());
// 결과: [apple, banana, cherry]

// 사용자 정의 정렬
List<String> sortedByLength = words.stream()
    .sorted(Comparator.comparing(String::length))
    .collect(Collectors.toList());
// 결과: [apple, banana, cherry]
```

#### Limit & Skip
```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

// 처음 5개만
List<Integer> first5 = numbers.stream()
    .limit(5)
    .collect(Collectors.toList());
// 결과: [1, 2, 3, 4, 5]

// 처음 3개 건너뛰고 나머지
List<Integer> skip3 = numbers.stream()
    .skip(3)
    .collect(Collectors.toList());
// 결과: [4, 5, 6, 7, 8, 9, 10]
```

### 3. 최종 연산 (Terminal Operations)
Stream을 소비하고 결과를 반환합니다.

#### Collect - 결과 수집
```java
List<String> words = Arrays.asList("apple", "banana", "cherry");

// List로 수집
List<String> list = words.stream().collect(Collectors.toList());

// Set으로 수집
Set<String> set = words.stream().collect(Collectors.toSet());

// Map으로 수집
Map<String, Integer> map = words.stream()
    .collect(Collectors.toMap(
        word -> word,           // key
        String::length         // value
    ));

// 문자열 결합
String joined = words.stream()
    .collect(Collectors.joining(", "));
// 결과: "apple, banana, cherry"
```

#### forEach - 각 요소에 대해 실행
```java
List<String> words = Arrays.asList("apple", "banana", "cherry");
words.stream().forEach(System.out::println);
```

#### Reduce - 누적 연산
```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

// 합계
int sum = numbers.stream()
    .reduce(0, Integer::sum);  // 0 + 1 + 2 + 3 + 4 + 5 = 15

// 최댓값
Optional<Integer> max = numbers.stream()
    .reduce(Integer::max);

// 문자열 연결
List<String> words = Arrays.asList("Java", "Stream", "API");
String result = words.stream()
    .reduce("", (s1, s2) -> s1 + " " + s2);
// 결과: " Java Stream API"
```

#### Match 연산
```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

// 모든 요소가 조건을 만족하는가?
boolean allEven = numbers.stream().allMatch(n -> n % 2 == 0);  // false

// 하나라도 조건을 만족하는가?
boolean anyEven = numbers.stream().anyMatch(n -> n % 2 == 0);  // true

// 모든 요소가 조건을 만족하지 않는가?
boolean noneNegative = numbers.stream().noneMatch(n -> n < 0);  // true
```

#### Find 연산
```java
List<String> words = Arrays.asList("apple", "banana", "cherry");

// 첫 번째 요소
Optional<String> first = words.stream().findFirst();

// 아무 요소나 (병렬 스트림에서 유용)
Optional<String> any = words.stream().findAny();
```

#### Count
```java
List<String> words = Arrays.asList("apple", "banana", "cherry");
long count = words.stream()
    .filter(word -> word.length() > 5)
    .count();  // 2
```

## 병렬 스트림 (Parallel Stream)

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

// 순차 스트림
int sum1 = numbers.stream()
    .mapToInt(Integer::intValue)
    .sum();

// 병렬 스트림
int sum2 = numbers.parallelStream()
    .mapToInt(Integer::intValue)
    .sum();

// 순차를 병렬로 변환
int sum3 = numbers.stream()
    .parallel()
    .mapToInt(Integer::intValue)
    .sum();
```

## 실용적인 예제

### 1. 데이터 처리 파이프라인
```java
public class Person {
    private String name;
    private int age;
    private String city;
    
    // constructor, getters, setters
}

List<Person> people = Arrays.asList(
    new Person("Alice", 25, "Seoul"),
    new Person("Bob", 30, "Busan"),
    new Person("Charlie", 35, "Seoul"),
    new Person("David", 20, "Incheon")
);

// 서울에 사는 25세 이상인 사람들의 이름을 나이 순으로 정렬
List<String> result = people.stream()
    .filter(p -> "Seoul".equals(p.getCity()))
    .filter(p -> p.getAge() >= 25)
    .sorted(Comparator.comparing(Person::getAge))
    .map(Person::getName)
    .collect(Collectors.toList());
```

### 2. 그룹화와 집계
```java
// 도시별로 그룹화
Map<String, List<Person>> peopleByCity = people.stream()
    .collect(Collectors.groupingBy(Person::getCity));

// 도시별 평균 나이
Map<String, Double> avgAgeByCity = people.stream()
    .collect(Collectors.groupingBy(
        Person::getCity,
        Collectors.averagingInt(Person::getAge)
    ));

// 나이대별 인원 수
Map<String, Long> countByAgeGroup = people.stream()
    .collect(Collectors.groupingBy(
        p -> p.getAge() >= 30 ? "30+" : "20s",
        Collectors.counting()
    ));
```

## 성능 고려사항

### 언제 Stream을 사용해야 할까?
- **사용하기 좋은 경우**: 복잡한 데이터 변환, 필터링, 집계 작업
- **피해야 할 경우**: 단순한 반복문, 성능이 중요한 작은 데이터셋

### 병렬 스트림 주의사항
```java
// 잘못된 예 - 상태 변경
List<Integer> result = new ArrayList<>();
numbers.parallelStream()
    .filter(n -> n % 2 == 0)
    .forEach(result::add);  // 스레드 안전하지 않음

// 올바른 예
List<Integer> result = numbers.parallelStream()
    .filter(n -> n % 2 == 0)
    .collect(Collectors.toList());  // 스레드 안전
```

## 주요 장점과 단점

### 장점
- **가독성**: 선언적 코드로 의도가 명확
- **재사용성**: 함수형 접근으로 모듈화 가능
- **병렬 처리**: 쉬운 병렬 처리 지원
- **지연 평가**: 필요한 시점까지 연산 지연

### 단점
- **디버깅 어려움**: 스택 트레이스 복잡
- **성능 오버헤드**: 작은 데이터셋에서는 전통적 반복문이 더 빠름
- **메모리 사용**: 중간 연산에서 임시 객체 생성

## 결론

Stream API는 Java에서 함수형 프로그래밍을 가능하게 해주는 강력한 도구입니다. 복잡한 데이터 처리 로직을 간결하고 읽기 쉬운 코드로 작성할 수 있으며, 병렬 처리를 통한 성능 향상도 기대할 수 있습니다. 단, 적절한 상황에서 사용해야 하며, 성능과 가독성을 모두 고려하여 선택하는 것이 중요합니다.