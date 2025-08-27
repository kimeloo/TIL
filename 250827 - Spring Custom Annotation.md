

## Custom Annotation이란?
- 사용자 정의 애노테이션
- 비즈니스 로직을 메타데이터로 표현
- 코드의 가독성과 재사용성 향상
- AOP와 결합하여 횡단 관심사 처리

## Meta Annotation
### @Target
- 애노테이션 적용 대상 지정
- **ElementType.TYPE**: 클래스, 인터페이스
- **ElementType.METHOD**: 메서드
- **ElementType.FIELD**: 필드
- **ElementType.PARAMETER**: 파라미터

### @Retention
- 애노테이션 유지 정책
- **RetentionPolicy.SOURCE**: 소스코드에만 유지
- **RetentionPolicy.CLASS**: 바이트코드까지 유지
- **RetentionPolicy.RUNTIME**: 런타임까지 유지

### @Documented
- JavaDoc에 포함 여부

### @Inherited
- 하위 클래스 상속 여부

## Custom Annotation 생성 과정
### 1. 애노테이션 정의
```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface LogExecutionTime {
    String value() default "";
}
```

### 2. Aspect 클래스 작성
```java
@Aspect
@Component
public class LoggingAspect {
    @Around("@annotation(LogExecutionTime)")
    public Object logExecutionTime(ProceedingJoinPoint joinPoint) throws Throwable {
        // 로직 구현
    }
}
```

### 3. 사용
```java
@Service
public class UserService {
    @LogExecutionTime
    public User findUser(Long id) {
        // 비즈니스 로직
    }
}
```

## 실무 활용 예시
### 1. 로깅 애노테이션
```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Loggable {
    String value() default "";
    boolean includeArgs() default true;
    boolean includeResult() default true;
}
```

### 2. 성능 측정 애노테이션
```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface PerformanceMonitoring {
    String operationName() default "";
    boolean alertOnSlowExecution() default false;
    long thresholdMs() default 1000;
}
```

### 3. 권한 검사 애노테이션
```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface RequirePermission {
    String[] value();
    boolean requireAll() default true;
}
```

### 4. 캐시 애노테이션
```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface CustomCache {
    String key() default "";
    int expireAfterWrite() default 300;
    String condition() default "";
}
```

### 5. API 버전 관리
```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface ApiVersion {
    String[] value();
    boolean deprecated() default false;
}
```

## 애노테이션 값 처리 방식
### 기본값 지정
```java
public @interface CustomAnnotation {
    String value() default "default";
    int timeout() default 5000;
    boolean enabled() default true;
}
```

### 배열 타입
```java
public @interface MultiValue {
    String[] roles() default {};
    Class<?>[] exceptions() default {};
}
```

### 열거형 사용
```java
public enum LogLevel {
    DEBUG, INFO, WARN, ERROR
}

public @interface Log {
    LogLevel level() default LogLevel.INFO;
}
```

## Reflection을 통한 애노테이션 정보 추출
```java
Method method = clazz.getMethod("methodName");
if (method.isAnnotationPresent(CustomAnnotation.class)) {
    CustomAnnotation annotation = method.getAnnotation(CustomAnnotation.class);
    String value = annotation.value();
    // 애노테이션 정보 활용
}
```

## Spring에서 애노테이션 처리 방식
### BeanPostProcessor
- 빈 생성 후 처리 과정에서 애노테이션 스캔
- 초기화 전후로 커스텀 로직 실행

### MethodInterceptor
- AOP 프록시를 통한 메서드 인터셉트
- 애노테이션 기반 횡단 관심사 처리

## 주의사항
### 성능 고려사항
- Reflection 사용으로 인한 성능 오버헤드
- 애노테이션 스캔 비용
- 프록시 생성 오버헤드

### 설계 원칙
- 단일 책임 원칙 준수
- 명확한 이름 사용
- 기본값 설정으로 사용성 향상
- 문서화 중요

## 장점
- 코드 중복 제거
- 선언적 프로그래밍
- 메타데이터 기반 설정
- 가독성 향상
- 유지보수성 증대

## 단점
- 런타임 의존성
- 디버깅 복잡도 증가
- 학습 곡선
- 과도한 추상화 위험