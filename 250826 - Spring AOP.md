
## AOP(Aspect-Oriented Programming)란?
- 관점 지향 프로그래밍
- 횡단 관심사(Cross-cutting Concerns)를 분리하여 모듈성 향상
- 비즈니스 로직과 부가 기능의 분리

## 주요 개념
### 1. Aspect
- 횡단 관심사를 모듈화한 것
- Advice + Pointcut

### 2. Advice
- 실제 부가 기능 로직
- **@Before**: 메서드 실행 전
- **@After**: 메서드 실행 후
- **@Around**: 메서드 실행 전후
- **@AfterReturning**: 메서드 성공적 반환 후
- **@AfterThrowing**: 예외 발생 시

### 3. Pointcut
- Advice가 적용될 위치를 정의
- execution(), within(), @annotation() 등

### 4. Target
- Advice가 적용되는 대상 객체

### 5. Proxy
- Target을 감싸는 객체
- 실제 AOP가 동작하는 방식

## Spring AOP 특징
- 프록시 기반 AOP
- 스프링 빈에만 적용 가능
- 메서드 호출 지점에서만 동작
- 런타임 시점에 동적 프록시 생성

## 프록시 생성 방식
### JDK Dynamic Proxy
- 인터페이스 기반
- 인터페이스를 구현한 클래스만 프록시 생성 가능

### CGLIB Proxy
- 클래스 기반
- 상속을 통한 프록시 생성
- final 클래스나 메서드는 프록시 불가

## 주요 애노테이션
- **@Aspect**: 클래스를 Aspect로 선언
- **@EnableAspectJAutoProxy**: AOP 기능 활성화
- **@Pointcut**: Pointcut 정의

## 실무 활용 예시
### 로깅
```java
@Around("execution(* com.example.service.*.*(..))")
public Object logging(ProceedingJoinPoint joinPoint) throws Throwable {
    // 로깅 로직
}
```

### 트랜잭션
```java
@Transactional
public void businessMethod() {
    // 비즈니스 로직
}
```

### 보안
```java
@PreAuthorize("hasRole('ADMIN')")
public void adminMethod() {
    // 관리자 전용 로직
}
```

## 장점
- 코드 중복 제거
- 비즈니스 로직과 부가 기능 분리
- 유지보수성 향상
- 재사용성 증대

## 주의사항
- 프록시 기반이므로 내부 호출 시 AOP 적용 안됨
- 성능 오버헤드 존재
- 디버깅 복잡도 증가
- 과도한 사용 시 코드 복잡성 증가