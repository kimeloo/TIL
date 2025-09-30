

## 정의
Chain of Responsibility (책임 연쇄 패턴)는 클라이언트의 요청을 여러 처리 객체(핸들러)들로 나누고, 사슬(chain) 형태로 연결해 연쇄적으로 처리하는 행동 패턴이다. 각 핸들러는 요청을 처리할 수 있는지 판단하고, 불가능하면 다음 핸들러에게 책임을 전가한다.

## 주요 구조

### 1. Handler (추상 핸들러)
- 요청을 수신하고 처리 객체들의 집합을 정의하는 인터페이스
- 다음 핸들러에 대한 참조를 가짐
- 요청 처리 메서드를 정의

### 2. ConcreteHandler (구체적 핸들러)
- 실제 요청을 처리하는 객체
- 자신이 처리할 수 있는 요청인지 확인
- 처리 불가능시 다음 핸들러로 요청 전달

### 3. Client (클라이언트)
- 요청을 Handler에 전달
- 구체적인 처리 객체들에 대해 알 필요 없음

## 장점

### 1. 유연한 확장성
- 클라이언트 코드 변경 없이 핸들러 추가/수정 가능
- 런타임에 체인 구성 변경 가능

### 2. 느슨한 결합
- 요청 호출자와 수신자 분리
- 객체 간 결합도 감소

### 3. 코드 최적화
- 중첩 if-else문 제거
- 단일 책임 원칙 준수

## 단점

### 1. 디버깅 어려움
- 요청 처리 흐름 추적 복잡
- 테스트 케이스 작성 어려움

### 2. 성능 문제
- 무한 사이클 발생 가능성
- 긴 체인으로 인한 요청 처리 지연

### 3. 실행 보장 없음
- 모든 핸들러가 요청을 거부할 수 있음

## 실무 사용 예시

### Java Logger
```java
Logger logger = Logger.getLogger("com.company.app");
logger.setLevel(Level.INFO);
```

### Servlet Filter
```java
public class AuthenticationFilter implements Filter {
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) {
        // 인증 로직
        chain.doFilter(request, response); // 다음 필터로 전달
    }
}
```

### Spring Security
- 여러 보안 필터들이 체인 형태로 연결
- 각 필터가 특정 보안 검사 수행

## 핵심 포인트
- 요청을 유연하고 동적으로 처리할 수 있는 객체 연결 메커니즘
- 클라이언트와 처리 객체 간의 결합도를 낮춤
- 새로운 처리 로직 추가 시 기존 코드 수정 최소화
- 책임의 분산과 연쇄적 처리가 핵심 개념

## 참고
- GoF 디자인 패턴의 행동 패턴 중 하나
- Command 패턴, Observer 패턴과 함께 자주 사용됨
- 미들웨어 패턴의 기반이 되는 구조