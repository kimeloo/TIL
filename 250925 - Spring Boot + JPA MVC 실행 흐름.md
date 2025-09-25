

## 전체 흐름도
```
브라우저 → 네트워크 → 서버(Tomcat) → Filter → DispatcherServlet → HandlerMapping
→ HandlerInterceptor → Controller(Proxy) → AOP → Service(Proxy) → Repository(Proxy)
→ JPA/Hibernate → Database
```

## 단계별 상세 흐름

### 1. 클라이언트 요청 단계
- **브라우저**: HTTP 요청 생성 (GET, POST 등)
- **네트워크**: TCP/IP를 통해 서버로 전송

### 2. 서버 진입점 (Tomcat)
- **Tomcat**: 내장 웹 서버가 HTTP 요청 수신
- **Thread Pool**: 요청별 스레드 할당
- **Socket Connection**: TCP 연결 수립

### 3. 필터 체인 (Filter Chain)
- **ServletFilter**: 요청/응답 전처리
  - 인코딩 설정
  - CORS 처리
  - 로깅
  - 보안 필터 (Spring Security)

### 4. DispatcherServlet
- **Front Controller**: 모든 HTTP 요청의 중앙 처리점
- **ApplicationContext**: Spring 컨테이너에서 빈 조회
- **HandlerMapping**: URL과 Controller 메서드 매핑

### 5. 핸들러 인터셉터 (HandlerInterceptor)
- **preHandle()**: Controller 실행 전
- **postHandle()**: Controller 실행 후, View 렌더링 전
- **afterCompletion()**: 응답 완료 후

### 6. 컨트롤러 계층 (Controller)
- **Proxy 객체**: Spring이 생성한 CGLIB/JDK 동적 프록시
- **@RequestMapping**: URL 매핑 및 파라미터 바인딩
- **Validation**: @Valid를 통한 입력 값 검증

### 7. AOP (Aspect-Oriented Programming)
- **@Around**: 메서드 실행 전후
- **@Before/@After**: 메서드 실행 전/후
- **Proxy 패턴**: 실제 객체를 감싸는 프록시를 통한 부가 기능
  - 트랜잭션 관리
  - 로깅
  - 보안
  - 캐싱

### 8. 서비스 계층 (Service)
- **@Transactional Proxy**: 트랜잭션 관리 프록시
- **비즈니스 로직**: 핵심 업무 처리
- **Transaction Manager**: 트랜잭션 시작/커밋/롤백

### 9. 리포지토리 계층 (Repository)
- **JPA Repository Proxy**: Spring Data JPA가 생성한 프록시
- **EntityManager**: JPA 영속성 컨텍스트
- **JPQL/Native Query**: 쿼리 실행

### 10. JPA/Hibernate
- **영속성 컨텍스트**: 1차 캐시, 변경 감지, 지연 로딩
- **Entity Lifecycle**: 비영속 → 영속 → 준영속 → 삭제
- **SQL 생성**: JPQL을 네이티브 SQL로 변환

### 11. 데이터베이스
- **Connection Pool**: 데이터베이스 연결 관리 (HikariCP)
- **SQL 실행**: 실제 데이터베이스 쿼리 수행
- **Result Set**: 결과 데이터 반환

## 응답 흐름 (역순)
```
Database → JPA → Repository → Service → Controller → View Resolver
→ View → DispatcherServlet → Filter → Tomcat → 네트워크 → 브라우저
```

### 응답 처리
- **View Resolution**: 논리적 뷰 이름을 물리적 뷰로 변환
- **Model and View**: 데이터와 뷰 정보
- **HTTP Response**: JSON, HTML, XML 등으로 응답 생성

## 주요 프록시 객체들

### 1. Controller Proxy
- Spring AOP에 의한 프록시
- @Transactional, @Cacheable 등의 어노테이션 처리

### 2. Service Proxy
- 트랜잭션 프록시 (TransactionInterceptor)
- AOP 기반 부가 기능

### 3. Repository Proxy
- Spring Data JPA가 인터페이스 기반으로 동적 생성
- SimpleJpaRepository 구현체를 감싸는 프록시

## 트랜잭션 흐름
1. **@Transactional 메서드 호출**
2. **TransactionInterceptor**: 트랜잭션 시작
3. **PlatformTransactionManager**: 실제 트랜잭션 관리
4. **Connection 획득**: 데이터베이스 연결
5. **비즈니스 로직 실행**
6. **커밋/롤백**: 성공 시 커밖, 예외 시 롤백