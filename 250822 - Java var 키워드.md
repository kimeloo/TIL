## 📝 한줄 요약

Java 10부터 도입된 var 키워드로 로컬 변수의 타입 선언을 생략하여 코드 간결성을 높이면서도 컴파일 시점 타입 추론으로 런타임 에러를 방지하는 기능

## ⚡ 언제 이 자료가 필요한가?

- **코드 가독성 향상 시**: 긴 제네릭 타입 선언으로 코드가 복잡해질 때
- **리팩토링 작업 시**: 타입 변경이 빈번한 코드에서 유지보수성을 높이고 싶을 때
- **Java 10+ 환경**: 최신 Java 버전을 사용하면서 모던한 문법을 적용하고 싶을 때
- **IDE 지원 활용 시**: 자동 완성과 타입 추론 기능을 최대한 활용하고 싶을 때

## 💡 핵심 내용

### 주요 개념

- **로컬 변수 타입 추론**: 컴파일러가 초기화 값을 통해 변수 타입을 자동으로 결정
- **컴파일 타임 타입 안전성**: 런타임이 아닌 컴파일 시점에서 타입 오류 검출
- **코드 간결성**: 반복되는 타입 선언 제거로 코드 가독성 향상

### 기술적 특징

- Java 10부터 지원되는 기능
- 로컬 변수에만 사용 가능 (인스턴스 변수, 전역 변수 불가)
- 반드시 초기화와 함께 선언해야 함
- 컴파일러의 타입 추론으로 강타입 언어의 안전성 유지

## 🔧 실습 & 적용

### 바로 적용 가능한 부분

- **기본 타입 추론**: `var test1 = "this is first";` (String으로 추론)
- **컬렉션 타입**: `var testArrayList = new ArrayList<String>();` (`ArrayList<String>`으로 추론)
- **복잡한 제네릭**: `var map = new HashMap<String, List<Integer>>();`

### var 사용 베스트 프랙티스

#### ✅ 권장 사용 사례
```java
// 1. 명확한 타입 컨텍스트가 있는 경우
var reader = Files.newBufferedReader(path);
var outputStream = new ByteArrayOutputStream();
var connection = DriverManager.getConnection(url);

// 2. 복잡한 제네릭 타입 간소화
var map = new HashMap<String, List<Integer>>();
var result = Collections.unmodifiableList(new ArrayList<>());

// 3. 지역 변수 스코프가 짧은 경우
var items = getItemList();
for (var item : items) {
    process(item);
}
```

#### ❌ 피해야 할 사용 사례
```java
// 1. 타입이 명확하지 않은 경우
var x = dbconn.executeQuery(query);  // 무엇을 반환하는지 불분명
var data = processData();            // 반환 타입을 추측해야 함

// 2. 원시 타입에서 의미가 불분명한 경우
var count = getCount();    // int? long? BigInteger?
var flag = isValid();      // boolean이지만 명시적 타입이 더 명확

// 3. 리터럴 값으로만 초기화하는 경우
var num = 10;        // int인지 다른 타입인지 불분명
var text = "hello";  // String이지만 명시적 선언이 더 좋음
```

### 사용 제한사항과 상세 이유

#### 컴파일 시점 제약사항
- **초기화 없는 선언 불가**: `var x;` 
  - 이유: 컴파일러가 타입을 추론할 초기값이 없음
- **null 값으로 초기화 불가**: `var x = null;`
  - 이유: null은 어떤 참조 타입이든 될 수 있어 타입 결정 불가
- **배열 직접 초기화 불가**: `var array = {1, 2, 3};`
  - 이유: 배열 리터럴만으로는 정확한 배열 타입 추론 불가
- **람다 표현식 직접 할당 불가**: `var lambda = () -> {};`
  - 이유: 람다는 여러 함수형 인터페이스로 해석될 수 있음

#### 스코프 제약사항
- **클래스 필드 사용 불가**: 메서드 내 지역 변수에만 사용 가능
- **메서드 파라미터 불가**: 메서드 시그니처의 명시적 타입 필요
- **메서드 반환 타입 불가**: API 계약의 명확성을 위해 필요

## 💻 성능 및 IDE 고려사항

### 컴파일 타임 vs 런타임
- **성능 영향 없음**: var는 컴파일 시점에만 작동하며 런타임 성능에 영향 없음
- **바이트코드 동일**: var로 선언된 변수와 명시적 타입 변수는 동일한 바이트코드 생성
- **타입 안전성 유지**: 컴파일 후에는 일반적인 강타입 변수와 동일하게 동작

### IDE 의존성과 주의사항
```java
// IDE 없이는 타입 파악이 어려운 경우
var result = someComplexMethod().chain().anotherMethod();

// 명시적 타입이 더 좋은 경우
ResultType result = someComplexMethod().chain().anotherMethod();
```

#### IDE 활용 팁
- **타입 힌트 표시**: 대부분의 IDE는 var 변수의 추론된 타입을 표시
- **리팩토링 도구**: var 사용 시 IDE의 자동 리팩토링 기능 활용
- **타입 추론 디버깅**: IDE의 타입 추론 정보를 통해 예상과 다른 타입 추론 확인

### 팀 개발 고려사항
- **코딩 컨벤션**: 팀 내 var 사용 가이드라인 수립 필요
- **코드 리뷰**: var 사용이 적절한지 리뷰 시점에서 검토
- **가독성 우선**: 간결함보다는 코드 가독성을 우선시

## 🚀 고급 사용 시나리오

### Stream API와의 조합
```java
// 복잡한 스트림 파이프라인에서 var 활용
var filteredUsers = users.stream()
    .filter(user -> user.getAge() > 18)
    .collect(Collectors.toList());

var groupedByDept = employees.stream()
    .collect(Collectors.groupingBy(Employee::getDepartment));

// 중간 결과를 var로 저장
var activeUsers = users.stream()
    .filter(User::isActive)
    .collect(Collectors.toList());
var premiumUsers = activeUsers.stream()
    .filter(user -> user.isPremium())
    .collect(Collectors.toList());
```

### 제네릭 타입의 복잡한 시나리오
```java
// 매우 복잡한 제네릭 타입 간소화
var nestedMap = new HashMap<String, Map<Integer, List<CustomObject>>>();
var builderPattern = SomeComplexBuilder.<String, Integer>builder()
    .withType(String.class)
    .withValue(42)
    .build();

// Optional과 함께 사용
var optionalResult = Optional.of(getData())
    .filter(Objects::nonNull)
    .map(this::transform);
```

### 메서드 체이닝과 빌더 패턴
```java
// 빌더 패턴에서 var 활용
var request = HttpRequest.newBuilder()
    .uri(URI.create("https://api.example.com"))
    .header("Content-Type", "application/json")
    .POST(HttpRequest.BodyPublishers.ofString(jsonData))
    .build();

// Fluent API와 함께
var queryResult = database
    .select("users")
    .where("age", ">", 18)
    .orderBy("name")
    .limit(100)
    .execute();
```

### 예외 처리와 리소스 관리
```java
// try-with-resources에서 var 사용
try (var reader = Files.newBufferedReader(path);
     var writer = Files.newBufferedWriter(outputPath)) {
    
    var lines = reader.lines()
        .filter(line -> !line.isEmpty())
        .collect(Collectors.toList());
    
    for (var line : lines) {
        writer.write(process(line));
        writer.newLine();
    }
}
```

## ⚠️ 주의사항과 엣지 케이스

### 타입 추론의 함정
```java
// 예상과 다른 타입 추론
var list = Arrays.asList(1, 2, 3);          // List<Integer>가 아닌 Arrays$ArrayList
var numbers = List.of(1, 2, 3);             // 불변 리스트 (ImmutableCollections$ListN)

// 다이아몬드 연산자와 함께 사용 시 주의
var map = new HashMap<>();                   // HashMap<Object, Object>로 추론됨
var typedMap = new HashMap<String, Integer>(); // 올바른 사용
```

### 익명 클래스와 람다의 한계
```java
// 익명 클래스는 사용 가능하지만 권장하지 않음
var runnable = new Runnable() {
    @Override
    public void run() {
        System.out.println("Hello");
    }
}; // 타입이 복잡해짐

// 대신 명시적 타입 사용 권장
Runnable runnable = new Runnable() {
    @Override
    public void run() {
        System.out.println("Hello");
    }
};
```

### 마이그레이션 시 고려사항
```java
// 기존 코드에서 var로 변경 시 주의점
// Before
List<String> items = getItems();
items.add("new item");  // 가능

// After - 만약 getItems()가 불변 리스트를 반환한다면
var items = getItems();
items.add("new item");  // 런타임 에러 가능성
```

## 🔗 관련 기술 & 학습 경로

- **Prerequisites**: Java 기본 문법, 타입 시스템, 제네릭 이해
- **관련 기술**: Java 10+ 신기능, 함수형 프로그래밍, Stream API
- **다음 학습**: 패턴 매칭 (Java 14+), Record 클래스 (Java 14+), Switch Expression

## 📚 추가 참고자료

- Java 10 공식 문서 - Local Variable Type Inference
- Oracle Java Language Updates - JEP 286
- 모던 Java 개발 가이드
- Effective Java 3rd Edition - Item 7 (var 사용법)
- Java Magazine - var 베스트 프랙티스 가이드

## 🎯 핵심 원칙 요약

> **"var는 코드를 더 읽기 쉽게 만들 때만 사용하라"**

1. **가독성 우선**: 타입이 명확하지 않으면 var 사용 금지
2. **의미 있는 변수명**: var 사용 시 변수명으로 의도를 명확히 표현
3. **짧은 스코프**: 지역 변수의 생명주기를 최소화
4. **팀 컨벤션 준수**: 일관된 사용 규칙 적용
5. **IDE 의존성 최소화**: IDE 없이도 코드 이해 가능하도록 작성

---

🔗 **참고 링크**: 
- 원본: https://lucas-owner.tistory.com/63
- 심화: https://imksh.com/117