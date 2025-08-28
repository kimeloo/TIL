# Java Switch Expression 버전별 변화

## Java 14 이전 (Traditional Switch Statement)

Java 14 이전에는 switch문만 사용 가능했고, 다음과 같은 특징이 있었다:

```java
// 전통적인 switch 문
String dayType;
switch (dayOfWeek) {
    case MONDAY:
    case TUESDAY:
    case WEDNESDAY:
    case THURSDAY:
    case FRIDAY:
        dayType = "Weekday";
        break;
    case SATURDAY:
    case SUNDAY:
        dayType = "Weekend";
        break;
    default:
        throw new IllegalArgumentException("Invalid day: " + dayOfWeek);
}
```

**문제점:**
- `break` 키워드를 빼먹으면 fall-through 발생
- 값을 반환할 수 없음 (statement만 가능)
- 코드가 장황함

## Java 12-13 (Preview Feature)

Java 12에서 switch expression이 preview 기능으로 도입되었다.

```java
// Java 12-13 Preview
String dayType = switch (dayOfWeek) {
    case MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY -> "Weekday";
    case SATURDAY, SUNDAY -> "Weekend";
};
```

**새로운 특징:**
- `->` 화살표 구문 도입
- 여러 case를 쉼표로 구분
- 표현식으로 사용 가능 (값 반환)
- fall-through 방지

## Java 14 (Standard Feature)

Java 14에서 switch expression이 정식 기능으로 채택되었다.

```java
// Java 14 정식 버전
String dayType = switch (dayOfWeek) {
    case MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY -> "Weekday";
    case SATURDAY, SUNDAY -> "Weekend";
    default -> throw new IllegalArgumentException("Invalid day: " + dayOfWeek);
};

// yield 키워드 사용
String result = switch (dayOfWeek) {
    case MONDAY -> {
        System.out.println("Start of work week");
        yield "Weekday";
    }
    case FRIDAY -> {
        System.out.println("End of work week");
        yield "Weekday";
    }
    case SATURDAY, SUNDAY -> "Weekend";
    default -> throw new IllegalArgumentException("Invalid day");
};
```

**주요 개선사항:**
- `yield` 키워드 도입 (블록 내에서 값 반환)
- 모든 case를 처리하지 않으면 컴파일 에러
- 더 안전하고 간결한 코드

## Java 15+ (추가 개선)

Java 15 이후로는 큰 변화는 없지만, 다른 기능들과의 조합이 개선되었다:

```java
// Pattern Matching과 함께 사용 (Java 17+)
String describe = switch (obj) {
    case Integer i -> "Integer: " + i;
    case String s -> "String: " + s;
    case null -> "null value";
    default -> "Unknown type";
};
```

## 비교 정리

| 버전 | 상태 | 주요 특징 |
|------|------|-----------|
| ~Java 11 | Traditional | `break` 필수, statement만 가능 |
| Java 12-13 | Preview | `->` 구문, expression 사용 가능 |
| Java 14 | Standard | `yield` 키워드, 정식 채택 |
| Java 15+ | Stable | Pattern Matching과 조합 개선 |

## 장점

1. **안전성**: fall-through 방지
2. **간결성**: 코드량 감소
3. **표현력**: 값을 직접 반환 가능
4. **완전성**: 모든 case 처리 강제
5. **성능**: 컴파일러 최적화 향상

Switch Expression은 Java의 현대적인 기능 중 하나로, 코드의 안전성과 가독성을 크게 향상시켰다.