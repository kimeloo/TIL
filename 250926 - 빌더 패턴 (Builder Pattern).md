

## 개념
복잡한 객체를 단계별로 생성할 수 있게 해주는 생성 패턴

## 언제 사용?
- 객체 생성 시 많은 매개변수가 필요한 경우
- 선택적 매개변수가 많은 경우
- 객체 생성 과정이 복잡한 경우
- 불변 객체를 만들고 싶은 경우

## 장점
- 복잡한 객체 생성을 단순화
- 가독성 향상 (메서드 체이닝)
- 불변 객체 생성 가능
- 유효성 검사 집중화

## 단점
- 코드 복잡성 증가
- 메모리 사용량 증가 (빌더 객체)

## 예시 (Java)
```java
public class User {
    private String name;
    private int age;
    private String email;

    private User(Builder builder) {
        this.name = builder.name;
        this.age = builder.age;
        this.email = builder.email;
    }

    public static class Builder {
        private String name;
        private int age;
        private String email;

        public Builder name(String name) {
            this.name = name;
            return this;
        }

        public Builder age(int age) {
            this.age = age;
            return this;
        }

        public Builder email(String email) {
            this.email = email;
            return this;
        }

        public User build() {
            return new User(this);
        }
    }
}

// 사용법
User user = new User.Builder()
    .name("김철수")
    .age(25)
    .email("kim@email.com")
    .build();
```

## JavaScript 예시
```javascript
class URLBuilder {
    constructor() {
        this.protocol = 'https';
        this.host = '';
        this.path = '';
        this.params = new Map();
    }

    setProtocol(protocol) {
        this.protocol = protocol;
        return this;
    }

    setHost(host) {
        this.host = host;
        return this;
    }

    setPath(path) {
        this.path = path;
        return this;
    }

    addParam(key, value) {
        this.params.set(key, value);
        return this;
    }

    build() {
        let url = `${this.protocol}://${this.host}${this.path}`;
        if (this.params.size > 0) {
            const queryString = Array.from(this.params.entries())
                .map(([key, value]) => `${key}=${value}`)
                .join('&');
            url += `?${queryString}`;
        }
        return url;
    }
}

// 사용법
const url = new URLBuilder()
    .setHost('api.example.com')
    .setPath('/users')
    .addParam('page', 1)
    .addParam('limit', 10)
    .build();
```