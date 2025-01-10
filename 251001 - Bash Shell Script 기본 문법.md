

## 1. Shebang (셔뱅)
```bash
#!/bin/bash
```
- 스크립트 첫 번째 줄에 위치
- 어떤 인터프리터로 실행할지 지정

## 2. 변수
```bash
# 변수 선언 (공백 없이)
name="John"
age=25

# 변수 사용
echo "Name: $name"
echo "Age: ${age}"

# 읽기 전용 변수
readonly PI=3.14159

# 환경 변수
export PATH="/usr/local/bin:$PATH"
```

## 3. 조건문
```bash
# if-else
if [ $age -gt 18 ]; then
    echo "Adult"
elif [ $age -eq 18 ]; then
    echo "Just turned adult"
else
    echo "Minor"
fi

# 비교 연산자
# -eq (equal), -ne (not equal), -gt (greater than)
# -lt (less than), -ge (greater equal), -le (less equal)

# 문자열 비교
if [ "$name" = "John" ]; then
    echo "Hello John"
fi

# 파일 테스트
if [ -f "file.txt" ]; then
    echo "File exists"
fi
# -d (directory), -r (readable), -w (writable), -x (executable)
```

## 4. 반복문
```bash
# for 루프
for i in {1..5}; do
    echo "Number: $i"
done

# 배열 순회
fruits=("apple" "banana" "orange")
for fruit in "${fruits[@]}"; do
    echo "$fruit"
done

# while 루프
counter=1
while [ $counter -le 5 ]; do
    echo "Counter: $counter"
    ((counter++))
done

# until 루프
until [ $counter -gt 10 ]; do
    echo "Counter: $counter"
    ((counter++))
done
```

## 5. 함수
```bash
# 함수 정의
function greet() {
    echo "Hello $1"
}

# 또는
greet() {
    local name=$1
    echo "Hello $name"
}

# 함수 호출
greet "World"

# 반환값
calculate() {
    local result=$(($1 + $2))
    echo $result
}

sum=$(calculate 5 3)
echo "Sum: $sum"
```

## 6. 배열
```bash
# 배열 선언
arr=("element1" "element2" "element3")

# 배열 요소 접근
echo ${arr[0]}        # 첫 번째 요소
echo ${arr[@]}        # 모든 요소
echo ${#arr[@]}       # 배열 길이

# 배열에 요소 추가
arr+=("element4")
```

## 7. 문자열 조작
```bash
text="Hello World"

# 문자열 길이
echo ${#text}

# 부분 문자열
echo ${text:0:5}      # "Hello"
echo ${text:6}        # "World"

# 문자열 치환
echo ${text/World/Universe}    # "Hello Universe"
echo ${text//l/L}              # "HeLLo WorLd"

# 대소문자 변환
echo ${text^^}        # 대문자로
echo ${text,,}        # 소문자로
```

## 8. 입출력
```bash
# 사용자 입력
echo "Enter your name:"
read name
echo "Hello $name"

# 여러 변수 입력
echo "Enter name and age:"
read name age

# 파일 읽기
while IFS= read -r line; do
    echo "$line"
done < "file.txt"

# 출력 리다이렉션
echo "Hello" > file.txt        # 덮어쓰기
echo "World" >> file.txt       # 추가
```

## 9. 명령어 실행과 캡처
```bash
# 명령어 실행
ls -la

# 명령어 결과 캡처
current_date=$(date)
file_count=`ls | wc -l`

echo "Today is $current_date"
echo "File count: $file_count"
```

## 10. 조건부 실행
```bash
# AND 연산자
command1 && command2    # command1이 성공하면 command2 실행

# OR 연산자
command1 || command2    # command1이 실패하면 command2 실행

# 예제
mkdir backup && cp *.txt backup/
ping -c 1 google.com || echo "Network is down"
```

## 11. 매개변수와 특수 변수
```bash
# 스크립트 매개변수
echo "Script name: $0"
echo "First argument: $1"
echo "Second argument: $2"
echo "All arguments: $@"
echo "Number of arguments: $#"

# 특수 변수
echo "Process ID: $$"
echo "Exit status: $?"
echo "Background process ID: $!"
```

## 12. case 문
```bash
case $1 in
    "start")
        echo "Starting service..."
        ;;
    "stop")
        echo "Stopping service..."
        ;;
    "restart")
        echo "Restarting service..."
        ;;
    *)
        echo "Usage: $0 {start|stop|restart}"
        ;;
esac
```

## 13. 에러 처리
```bash
# set 옵션
set -e    # 에러 발생시 스크립트 종료
set -u    # 정의되지 않은 변수 사용시 에러
set -x    # 실행되는 명령어 출력

# 에러 트랩
trap 'echo "Error occurred"' ERR

# 함수에서 에러 처리
safe_command() {
    if ! command_that_might_fail; then
        echo "Command failed"
        return 1
    fi
}
```

## 14. 산술 연산
```bash
# 산술 확장
result=$((5 + 3))
echo $result

# let 명령어
let "result = 5 + 3"

# expr 명령어
result=$(expr 5 + 3)

# 부동소수점 연산 (bc 사용)
result=$(echo "scale=2; 5/3" | bc)
```

## 15. 파일 처리
```bash
# 파일 존재 확인
if [ -f "file.txt" ]; then
    echo "File exists"
fi

# 디렉토리 확인
if [ -d "directory" ]; then
    echo "Directory exists"
fi

# 파일 권한 확인
if [ -r "file.txt" ]; then
    echo "File is readable"
fi

# 파일 크기 확인
if [ -s "file.txt" ]; then
    echo "File is not empty"
fi
```