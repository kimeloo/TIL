

## 정의
헥사고날 아키텍처는 **기술과 실제 비즈니스 로직의 분리**를 목표로 하는 아키텍처 패턴이다. 포트와 어댑터(Ports and Adapters) 패턴을 기반으로 한 소프트웨어 설계 방식으로, 애플리케이션의 핵심 비즈니스 로직을 외부 의존성으로부터 격리시키는 것이 주요 목표다.

## 주요 특징

### 1. 도메인과 인프라의 명확한 분리
- **도메인 영역**: 비즈니스 로직과 규칙이 포함된 핵심 영역
- **인프라 영역**: 데이터베이스, 외부 API, UI 등 기술적 구현 사항

### 2. 핵심 제약 조건
1. **단방향 의존성**: 도메인 영역 → 인프라 영역으로만 접근 가능
2. **포트와 어댑터를 통한 접근**: 모든 외부 상호작용은 포트와 어댑터를 통해서만 가능

## 구조

```
        ┌─────────────────┐
        │   인바운드      │
        │   어댑터        │
        └─────────────────┘
              │
        ┌─────────────────┐
        │   인바운드      │
        │   포트          │
        └─────────────────┘
              │
    ┌─────────────────────────┐
    │                         │
    │     도메인/서비스       │
    │   (비즈니스 로직)       │
    │                         │
    └─────────────────────────┘
              │
        ┌─────────────────┐
        │   아웃바운드    │
        │   포트          │
        └─────────────────┘
              │
        ┌─────────────────┐
        │   아웃바운드    │
        │   어댑터        │
        └─────────────────┘
```

### 주요 구성 요소

1. **인바운드 어댑터**: 외부에서 애플리케이션으로 들어오는 요청 처리 (REST API, GraphQL, CLI 등)
2. **인바운드 포트**: 애플리케이션이 외부에 노출하는 인터페이스
3. **도메인/서비스**: 핵심 비즈니스 로직이 구현된 영역
4. **아웃바운드 포트**: 애플리케이션이 외부 시스템에 접근하기 위한 인터페이스
5. **아웃바운드 어댑터**: 외부 시스템과의 실제 통신을 담당 (데이터베이스, 외부 API 등)

## 장점

### 1. 느슨한 결합 (Loose Coupling)
- 비즈니스 로직과 기술적 구현 사항이 독립적
- 각 계층의 변경이 다른 계층에 미치는 영향 최소화

### 2. 기술적 세부사항 변경 용이
- 데이터베이스 변경 (MySQL → PostgreSQL)
- 프레임워크 변경 (Spring → Express)
- 외부 API 변경 등이 도메인 로직에 영향 없음

### 3. 테스트 용이성
- 도메인 로직을 독립적으로 테스트 가능
- Mock 객체를 통한 단위 테스트 작성 용이

### 4. 유지보수성 향상
- 비즈니스 규칙 변경 시 도메인 영역만 수정
- 기술적 변경 시 어댑터만 수정

## 단점

### 1. 복잡성 증가
- 작은 프로젝트에는 과도한 추상화
- 초기 설계 및 구현 시간 증가

### 2. 학습 곡선
- 팀원들의 아키텍처 패턴 이해 필요
- 올바른 계층 분리에 대한 경험 필요

### 3. 성능 오버헤드
- 여러 계층을 거치면서 발생하는 성능 손실
- 과도한 인터페이스 추상화로 인한 복잡도

## 구현 방법

### 1. 포트 정의
```java
// 인바운드 포트
public interface UserService {
    User createUser(CreateUserRequest request);
    User getUserById(Long id);
}

// 아웃바운드 포트
public interface UserRepository {
    User save(User user);
    Optional<User> findById(Long id);
}
```

### 2. 도메인 서비스 구현
```java
@Service
public class UserServiceImpl implements UserService {
    private final UserRepository userRepository;
    
    public UserServiceImpl(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
    
    @Override
    public User createUser(CreateUserRequest request) {
        // 비즈니스 로직 구현
        User user = new User(request.getName(), request.getEmail());
        return userRepository.save(user);
    }
}
```

### 3. 어댑터 구현
```java
// 인바운드 어댑터 (REST Controller)
@RestController
public class UserController {
    private final UserService userService;
    
    @PostMapping("/users")
    public ResponseEntity<User> createUser(@RequestBody CreateUserRequest request) {
        User user = userService.createUser(request);
        return ResponseEntity.ok(user);
    }
}

// 아웃바운드 어댑터 (JPA Repository)
@Repository
public class JpaUserRepository implements UserRepository {
    private final UserJpaRepository jpaRepository;
    
    @Override
    public User save(User user) {
        return jpaRepository.save(user);
    }
}
```

## 적용 시나리오

### 적합한 경우
- 복잡한 비즈니스 로직을 가진 애플리케이션
- 다양한 외부 시스템과 연동이 필요한 경우
- 장기적 유지보수가 중요한 프로젝트
- 기술 스택 변경 가능성이 높은 경우

### 부적합한 경우
- 단순한 CRUD 애플리케이션
- 빠른 프로토타입 개발이 필요한 경우
- 소규모 팀 또는 프로젝트
- 성능이 최우선인 시스템

## 결론

헥사고날 아키텍처는 비즈니스 로직과 기술적 구현을 명확히 분리하여 **유지보수성**, **테스트 용이성**, **확장성**을 크게 향상시키는 아키텍처 패턴이다. 하지만 프로젝트의 복잡도와 규모를 고려하여 적절히 적용해야 하며, 팀의 이해도와 프로젝트 요구사항을 충분히 검토한 후 도입을 결정해야 한다.

---
*참고: https://blog.imqa.io/hexagonal-architecture/*