
```table-of-contents
```
https://sundries-in-myidea.tistory.com/167

## 핵심 결론

**Self Injection은 사용하지 않는 것이 좋다**

- Spring과 Spring Boot에서 기본적으로 차단하고 있음
- 무한 참조 위험성 때문
- 프레임워크 가이드라인을 따르는 것이 권장됨

## 문제 배경

### SonarQube 경고

```
Methods with Spring proxy should not be called via "this"
```

### 일반적인 해결 방식의 한계

- 트랜잭션 중첩이 필요한 상황 (예: readonly → readwrite)
- `this.method()` 호출 시 프록시가 작동하지 않음
- 기존 해결책: 별도 클래스 분리 → 번거로움

### Self Injection 아이디어

- 자기 자신을 주입받아서 프록시 문제 해결
- 클래스 분리 없이 중첩 트랜잭션 가능할까?

## 실제 테스트

### 구현 방식

```java
@Service
@Transactional(readOnly = true)
public class TestService {
    private TestService testService;
    
    @Autowired
    public void setTestService(TestService testService) {
        this.testService = testService;
    }
    
    public void readOnlyMethod() {
        // readonly 트랜잭션
        testService.writeMethod(); // self injection으로 호출
    }
    
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void writeMethod() {
        // write 트랜잭션
    }
}
```

### 결과

- **순환 참조 오류 발생**
- Spring Boot는 기본적으로 순환 참조를 차단
- `spring.main.allow-circular-references=true` 설정하면 실행 가능

## Spring 공식 입장

### GitHub 이슈 조사 결과

**Spring Boot Issue #27652**

- 순환 참조를 기본으로 금지하는 정책 도입
- 많은 개발자들이 반대 의견 제시

**Spring Framework Issue #28299**

- Self Injection의 합법적 사용 사례에 대한 질문
- 공식 문서에 주석 추가로 답변

### 공식 권고사항

> "자체 주입(self-injection)은 최후의 수단으로만 사용해야 합니다. 영향을 받는 메서드를 별도의 델리게이트 빈으로 분리하는 것을 고려하세요."

## 저자의 결론 변화

**초기 입장**: Self Injection도 괜찮지 않을까?

- SonarQube의 Compliant solution 예시를 보고 생각
- 클래스 분리의 번거로움 때문

**최종 입장**: Spring 권고사항을 따르자

- 프레임워크에서 권고하지 않는 방식은 피하는 게 좋음
- 무한 루프 위험성 존재
- 명확한 대안(클래스 분리)이 있음

## 실무 적용 지침

### ✅ 권장 방식

```java
// 트랜잭션 관련 로직을 별도 서비스로 분리
@Service
public class TestTransactionService {
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void writeMethod() {
        // write 트랜잭션
    }
}

@Service
@Transactional(readOnly = true)  
public class TestService {
    private final TestTransactionService transactionService;
    
    public void readOnlyMethod() {
        // readonly 트랜잭션
        transactionService.writeMethod(); // 별도 서비스 호출
    }
}
```

### ❌ 비권장 방식

- Self Injection 구현
- 순환 참조 강제 허용 설정
- 프레임워크 가이드라인 무시

## 참고사항

- 개발자 커뮤니티에서도 의견이 갈리는 주제
- 하지만 Spring 팀의 명확한 입장: **사용하지 말 것**
- 번거롭더라도 클래스 분리가 안전한 방법