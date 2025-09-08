
## 개요
Record 클래스는 Java 14에서 프리뷰 기능으로 처음 도입되어 Java 16에서 정식 기능이 된 새로운 클래스 유형입니다. 데이터를 담는 불변(immutable) 클래스를 간결하게 정의할 수 있게 해주며, 보일러플레이트 코드를 크게 줄여줍니다.

## 기본 문법

```java
record Rectangle(double length, double width) { }
```

위의 간단한 Record 선언은 다음과 같은 일반 클래스와 동등합니다:

```java
public final class Rectangle {
    private final double length;
    private final double width;

    public Rectangle(double length, double width) {
        this.length = length;
        this.width = width;
    }

    public double length() { return this.length; }
    public double width() { return this.width; }

    public boolean equals(Object obj) { ... }
    public int hashCode() { ... }
    public String toString() { ... }
}
```

## 자동 생성되는 요소들

Record 클래스는 다음 요소들을 자동으로 생성합니다:

1. **private final 필드**: 각 컴포넌트에 대해
2. **public 접근자 메서드**: 각 컴포넌트에 대해 (getter)
3. **표준 생성자**: 모든 컴포넌트를 매개변수로 받는 생성자
4. **equals()와 hashCode()**: 모든 필드를 기반으로 한 구현
5. **toString()**: 모든 필드의 값을 포함한 문자열 표현

## 생성자 사용법

### 표준 생성자
```java
Rectangle r = new Rectangle(4.0, 5.0);
```

### 명시적 표준 생성자 (검증 포함)
```java
record Rectangle(double length, double width) {
    public Rectangle(double length, double width) {
        if (length <= 0 || width <= 0) {
            throw new IllegalArgumentException(
                String.format("Invalid dimensions: %f, %f", length, width));
        }
        this.length = length;
        this.width = width;
    }
}
```

### 컴팩트 생성자
```java
record Rectangle(double length, double width) {
    public Rectangle {
        if (length <= 0 || width <= 0) {
            throw new IllegalArgumentException(
                String.format("Invalid dimensions: %f, %f", length, width));
        }
    }
}
```

## 메서드 오버라이드

접근자 메서드를 명시적으로 구현할 수 있습니다:

```java
record Rectangle(double length, double width) {
    public double length() {
        System.out.println("Length is " + length);
        return length;
    }
}
```

## 정적 멤버

Record에는 정적 필드, 정적 초기화 블록, 정적 메서드를 선언할 수 있습니다:

```java
record Rectangle(double length, double width) {
    static double goldenRatio;
    
    static {
        goldenRatio = (1 + Math.sqrt(5)) / 2;
    }
    
    public static Rectangle createGoldenRectangle(double width) {
        return new Rectangle(width, width * goldenRatio);
    }
}
```

## 인스턴스 메서드와 중첩 클래스

```java
record Rectangle(double length, double width) {
    // 중첩 Record 클래스
    record RotationAngle(double angle) {
        public RotationAngle {
            angle = Math.toRadians(angle);
        }
    }
    
    // 인스턴스 메서드
    public Rectangle getRotatedRectangleBoundingBox(double angle) {
        RotationAngle ra = new RotationAngle(angle);
        double x = Math.abs(length * Math.cos(ra.angle())) +
                   Math.abs(width * Math.sin(ra.angle()));
        double y = Math.abs(length * Math.sin(ra.angle())) +
                   Math.abs(width * Math.cos(ra.angle()));
        return new Rectangle(x, y);
    }
}
```

## Record의 특징

### 1. 제네릭 지원
```java
record Triangle<C extends Coordinate>(C top, C left, C right) { }
```

### 2. 인터페이스 구현
```java
record Customer(String name, String email) implements Billable { 
    @Override
    public void generateBill() {
        // 청구서 생성 로직
    }
}
```

### 3. 어노테이션 지원
```java
record Rectangle(
    @GreaterThanZero double length,
    @GreaterThanZero double width
) { }
```

### 4. 로컬 Record 클래스
메서드 내부에서 Record를 정의할 수 있습니다:

```java
public List<Merchant> findTopMerchants(List<Sale> sales, int year, Month month) {
    record MonthlySales(Merchant merchant, double sales) { }
    
    return sales.stream()
        .filter(sale -> sale.date().getYear() == year && 
                       sale.date().getMonth() == month)
        .collect(groupingBy(Sale::merchant, summingDouble(Sale::value)))
        .entrySet().stream()
        .map(entry -> new MonthlySales(entry.getKey(), entry.getValue()))
        .sorted((ms1, ms2) -> Double.compare(ms2.sales(), ms1.sales()))
        .limit(10)
        .map(MonthlySales::merchant)
        .collect(toList());
}
```

## 제한사항

1. **final 클래스**: Record는 암시적으로 final이므로 상속할 수 없습니다
2. **인스턴스 변수 불가**: non-static 필드를 추가로 선언할 수 없습니다
3. **인스턴스 초기화 블록 불가**: 인스턴스 초기화 블록을 사용할 수 없습니다
4. **native 메서드 불가**: native 메서드를 선언할 수 없습니다

## 사용 사례

Record는 다음과 같은 경우에 적합합니다:

1. **DTO (Data Transfer Object)**: 데이터 전송을 위한 객체
2. **Value Object**: 값만을 담는 불변 객체
3. **Configuration**: 설정 정보를 담는 객체
4. **API Response/Request**: REST API의 요청/응답 객체

## 예제: 실제 사용

```java
// 사용자 정보 DTO
record UserDto(Long id, String name, String email, LocalDateTime createdAt) {
    public UserDto {
        Objects.requireNonNull(name, "Name cannot be null");
        Objects.requireNonNull(email, "Email cannot be null");
    }
}

// 좌표 시스템
record Point(int x, int y) {
    public double distanceFrom(Point other) {
        return Math.sqrt(Math.pow(this.x - other.x, 2) + 
                        Math.pow(this.y - other.y, 2));
    }
}

// API 응답
record ApiResponse<T>(boolean success, String message, T data) {
    public static <T> ApiResponse<T> success(T data) {
        return new ApiResponse<>(true, "Success", data);
    }
    
    public static <T> ApiResponse<T> error(String message) {
        return new ApiResponse<>(false, message, null);
    }
}
```

Record 클래스는 Java의 표현력을 높이고 보일러플레이트 코드를 줄여주는 강력한 기능으로, 특히 데이터 중심의 애플리케이션에서 매우 유용합니다.