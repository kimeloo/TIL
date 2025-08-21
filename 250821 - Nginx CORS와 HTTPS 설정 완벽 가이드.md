
## 📝 한줄 요약

Nginx에서 발생하는 CORS 오류를 OPTIONS 메소드 처리와 다중 도메인 허용으로 해결하고, 무중단 배포 환경에서 HTTPS 설정을 유지하는 방법

## ⚡ 언제 이 자료가 필요한가?

- **프론트엔드 개발 시**: 로컬 개발 환경에서 API 요청 시 CORS 오류가 발생할 때
- **배포 환경 구성 시**: Nginx를 리버스 프록시로 사용하면서 CORS 정책 설정이 필요할 때
- **무중단 배포 도입 시**: Blue-Green 배포 방식에서 HTTPS 인증이 계속 풀리는 문제를 해결해야 할 때
- **다중 도메인 지원 시**: localhost와 배포 도메인을 동시에 허용해야 할 때

## 💡 핵심 내용

### 주요 개념

- **Preflight 요청**: 브라우저가 실제 요청 전에 보내는 OPTIONS 메소드 요청으로 CORS 정책 확인
- **동적 Origin 설정**: map 지시어를 사용해 여러 도메인을 조건부로 허용하는 방식
- **Certbot 설정 유지**: nginx 설정 파일 재생성 시에도 SSL 인증서 설정을 보존하는 방법

### 기술적 특징

- OPTIONS 메소드에 대한 별도 헤더 설정으로 preflight 요청 처리
- map 지시어로 Access-Control-Allow-Origin의 단일 값 제한 해결
- SSL 설정을 nginx 설정 파일에 직접 포함하여 배포 시 유지

## 🔧 실습 & 적용

### 바로 적용 가능한 부분

- **CORS 오류 해결**: OPTIONS 메소드 처리 블록을 location 내부에 추가
- **다중 도메인 허용**: map 지시어로 $allowed_origin 변수 설정 후 헤더에 적용
- **HTTPS 설정 보존**: certbot이 자동 생성한 SSL 설정을 nginx 템플릿에 복사

### 주의사항

- `add_header` 지시어에 `always` 옵션 추가로 모든 응답에 헤더 포함 보장
- Access-Control-Allow-Headers에 실제 사용하는 헤더만 명시 (Content-Type, Authorization 등)
- 무중단 배포 시 각 nginx 설정 파일(blue, green)에 동일한 SSL 설정 적용 필요

## 🔗 관련 기술 & 학습 경로

- **Prerequisites**: Nginx 기본 설정, HTTP/HTTPS 프로토콜, CORS 정책 이해
- **관련 기술**: Docker, Certbot, Blue-Green 배포, Vercel, React 개발 환경
- **다음 학습**: Nginx 보안 헤더 설정, Load Balancing, Rate Limiting

## 📚 추가 참고자료

- Nginx CORS 설정 공식 문서
- Let's Encrypt Certbot 가이드
- 무중단 배포 전략 및 패턴

---

🔗 **원본 링크**: [https://medium.com/@hee98.09.14/nginx-cors-https-설정-문제-해결하기-5af71812f4a1](https://medium.com/@hee98.09.14/nginx-cors-https-%EC%84%A4%EC%A0%95-%EB%AC%B8%EC%A0%9C-%ED%95%B4%EA%B2%B0%ED%95%98%EA%B8%B0-5af71812f4a1)