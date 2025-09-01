# 250901 - Git Push 오류 해결

## 핵심 개념
Git push 시 발생하는 RPC failed, curl error 등의 네트워크 관련 오류에 대한 해결 방법

## 주요 오류 유형

### RPC failed 오류
- Remote Procedure Call 실패로 인한 push 중단
- 주로 대용량 파일이나 네트워크 불안정성에 의해 발생

### curl error
- HTTP 전송 과정에서 발생하는 오류
- 버퍼 크기 부족이나 HTTP 버전 호환성 문제

## 해결 방법

### 1. HTTP postBuffer 크기 증가
```bash
git config --global http.postBuffer 524288000
```
- Git의 HTTP 버퍼 크기를 500MB로 설정
- 대용량 파일 전송 시 버퍼 부족 문제 해결
- 글로벌 설정으로 모든 저장소에 적용

### 2. 대용량 파일 관리
- 50MB 이상 파일들이 push 실패의 주요 원인
- `.gitignore`에 대용량 파일 확장자 추가 권장
- Git LFS(Large File Storage) 활용 고려

### 3. HTTP 버전 다운그레이드
```bash
git config --global http.version HTTP/1.1
```
- HTTP/2 호환성 문제 해결
- 일부 Git 서버에서 HTTP/1.1이 더 안정적
- 네트워크 환경에 따른 최적화

## 활용 상황
- 대용량 파일이 포함된 프로젝트
- 네트워크가 불안정한 환경
- CI/CD 파이프라인에서 push 실패 시
- 원격 저장소 연결 타임아웃 발생 시

## 추가 고려사항

### 임시 해결책
- 파일을 분할하여 여러 번에 나누어 push
- `git push --force-with-lease` 옵션 활용

### 근본적 해결
- 저장소 구조 개선
- 바이너리 파일의 별도 관리
- Git 워크플로우 최적화

## 관련 설정
```bash
# 전체 설정 확인
git config --global --list | grep http

# 설정 초기화 (필요 시)
git config --global --unset http.postBuffer
git config --global --unset http.version
```

## 결론
Git push 오류는 주로 네트워크 설정과 파일 크기 문제에서 기인. postBuffer 크기 조정과 HTTP 버전 설정으로 대부분 해결 가능하며, 근본적으로는 저장소 관리 전략 개선이 중요.