# Windows 개행문자 실행 오류 해결

## 문제 상황
- 쉘 스크립트 실행 시 `env: bash\r: No such file or directory` 오류 발생
- Windows 환경에서 작성된 스크립트를 Linux 환경에서 실행할 때 발생

## 원인 분석
Windows와 Linux의 개행문자 차이로 인한 문제:
- **Windows**: CRLF (`\r\n`) 사용
- **Linux/Unix**: LF (`\n`) 사용
- Windows에서 작성된 스크립트의 `^M` 문자가 Linux에서 인식되지 않음

## 해결 방법

### 1. dos2unix 명령어 사용
```bash
dos2unix install.sh
```

### 2. sed 명령어 사용
```bash
# 기본 형태
sed -i 's/\r$//' [filename]

# 새 파일로 저장
sed $'s/\r$//' ./install.sh > ./install.Unix.sh
```

### 3. vi 에디터 활용
```bash
# 방법 1: vi에서 직접 삭제
vi -b [filename]
# ^M 문자를 수동으로 삭제

# 방법 2: 파일 포맷 변경
vi [filename]
:set fileformat=unix
:wq
```

### 4. Git 설정
```bash
# Git 전역 설정으로 자동 변환
git config --global core.autocrlf input
```

### 5. 텍스트 에디터 설정
- **VS Code**: 하단 상태바에서 CRLF를 LF로 변경
- **Vim**: `:set fileformat=unix` 명령어 사용

## 예방 방법

### Git 설정으로 예방
```bash
# Windows 사용자
git config --global core.autocrlf true

# Linux/Mac 사용자
git config --global core.autocrlf input
```

### 에디터 기본 설정
- 개발 환경에서 기본 개행문자를 LF로 설정
- 프로젝트별 `.editorconfig` 파일 활용

## 주의사항
- 개행문자 변경 시 기존 파일의 버전 관리 히스토리에 영향 가능
- 팀 프로젝트에서는 개행문자 정책 통일 필요
- 바이너리 파일에는 개행문자 변환 도구 사용 금지

## 관련 링크
- [블로그 참고글 1](https://gethlemn.tistory.com/66)
- [Stack Overflow 참고글](https://stackoverflow.com/questions/29045140/env-bash-r-no-such-file-or-directory)