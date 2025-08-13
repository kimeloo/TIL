# Database Connection Pool 튜닝

## Connection Pool이란?
- 미리 일정 수의 데이터베이스 연결을 생성하고 Pool에서 관리
- 사용자 요청 시 Pool에서 연결을 할당하고, 사용 완료 후 다시 Pool로 반환
- DB 연결 생성/해제 비용을 줄여 성능 향상

## 주요 발생 문제

### 1. Too many connections 에러
- **원인**: 연결 사용 후 자원 반납 누락
- **해결**: Connection 사용 후 반드시 `close()` 호출

### 2. Connection 타임아웃 문제
- **원인**: MySQL의 `wait_timeout` 설정과 Connection Pool의 유지 시간 불일치
- **MySQL 기본값**: 28800초 (8시간)
- **해결방법**:
  - `wait_timeout` 값을 600초로 조정
  - Connection recycle 설정으로 주기적 갱신

## MySQL 성능 튜닝 설정

### Connection 관련 설정
```sql
-- 최대 연결 수 증가
SET GLOBAL max_connections = 1000;  -- 기본값: 151

-- Connection 타임아웃 조정
SET GLOBAL wait_timeout = 600;      -- 기본값: 28800

-- 스레드 캐시 크기 최적화
SET GLOBAL thread_cache_size = 16;
```

### Connection Pool 설정 예시
```properties
# HikariCP 예시
spring.datasource.hikari.maximum-pool-size=50
spring.datasource.hikari.minimum-idle=10
spring.datasource.hikari.connection-timeout=30000
spring.datasource.hikari.idle-timeout=600000
spring.datasource.hikari.max-lifetime=1800000
```

## 모니터링 포인트
- Connection 사용률 추적
- Pool에서 연결 대기 시간 모니터링
- 메모리 사용량 체크
- `SHOW PROCESSLIST`로 활성 연결 상태 확인

## 주요 성과
- Connection 관련 오류 대폭 감소
- 데이터베이스 응답 속도 향상
- 시스템 안정성 개선
- 운영 비용 절감

## 주의사항
- 물리 메모리 한계 고려하여 최대 연결 수 설정
- 워크로드 패턴에 맞는 Pool 크기 조정 필요
- 지속적인 모니터링을 통한 최적화 필수

## 관련 링크
- [원본 글](https://jiku90.tistory.com/14)