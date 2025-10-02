

## 정의
옵저버 패턴은 한 객체의 상태가 바뀌면 그 객체에 의존하는 다른 객체들에게 연락이 가고, 자동으로 내용이 갱신되는 방식으로 **일대다(one-to-many) 의존성을 정의**하는 행동 디자인 패턴이다.

## 핵심 개념
- **Subject (주제)**: 상태를 가지고 있는 객체 (Observable)
- **Observer (관찰자)**: 상태 변화를 감지하고 반응하는 객체들
- **구독 메커니즘**: 여러 객체가 관찰 중인 객체의 이벤트를 구독

## 주요 특징

### 1. 느슨한 결합 (Loose Coupling)
- 주제와 옵저버가 서로 독립적으로 변경 가능
- 상호작용하는 객체 간의 의존성 최소화

### 2. 일대다 관계
- 하나의 주제가 여러 옵저버에게 동시에 알림
- 옵저버는 동적으로 추가/제거 가능

### 3. 자동 갱신
- 주제의 상태 변화 시 모든 옵저버가 자동으로 업데이트

## 기본 구조

### Subject 인터페이스
```java
interface Subject {
    void registerObserver(Observer observer);  // 옵저버 등록
    void removeObserver(Observer observer);     // 옵저버 제거
    void notifyObservers();                     // 모든 옵저버에게 알림
}
```

### Observer 인터페이스
```java
interface Observer {
    void update(Object data);  // 상태 변화 알림 수신
}
```

## 구현 예제

### 1. 뉴스 구독 시스템
```java
// Subject 구현
class NewsAgency implements Subject {
    private List<Observer> observers = new ArrayList<>();
    private String news;

    @Override
    public void registerObserver(Observer observer) {
        observers.add(observer);
    }

    @Override
    public void removeObserver(Observer observer) {
        observers.remove(observer);
    }

    @Override
    public void notifyObservers() {
        for (Observer observer : observers) {
            observer.update(news);
        }
    }

    public void setNews(String news) {
        this.news = news;
        notifyObservers();
    }
}

// Observer 구현
class NewsChannel implements Observer {
    private String name;

    public NewsChannel(String name) {
        this.name = name;
    }

    @Override
    public void update(Object news) {
        System.out.println(name + "에서 뉴스 수신: " + news);
    }
}

// 사용 예시
NewsAgency agency = new NewsAgency();
NewsChannel channel1 = new NewsChannel("KBS");
NewsChannel channel2 = new NewsChannel("MBC");

agency.registerObserver(channel1);
agency.registerObserver(channel2);

agency.setNews("옵저버 패턴 학습 완료!");
```

### 2. 유튜브 구독 시스템
```java
// 유튜버 (Subject)
class YouTuber implements Subject {
    private List<Observer> subscribers = new ArrayList<>();
    private String videoTitle;

    public void uploadVideo(String title) {
        this.videoTitle = title;
        notifyObservers();
    }

    // registerObserver, removeObserver, notifyObservers 구현...
}

// 구독자 (Observer)
class Subscriber implements Observer {
    private String name;

    @Override
    public void update(Object videoTitle) {
        System.out.println(name + "님, 새 영상이 업로드됐습니다: " + videoTitle);
    }
}
```

## 활용 사례

### 1. GUI 프로그래밍
- 버튼 클릭, 마우스 이벤트 등의 이벤트 처리
- 모델-뷰 아키텍처에서 모델 변화 시 뷰 업데이트

### 2. 웹 개발
- JavaScript의 이벤트 리스너
- React의 상태 관리 (useState, useEffect)

### 3. 시스템 모니터링
- 로그 시스템, 알림 시스템
- 메트릭 수집 및 대시보드 업데이트

## 장점
1. **개방-폐쇄 원칙**: 새로운 옵저버 추가 시 기존 코드 수정 불필요
2. **런타임 관계 설정**: 실행 중에 옵저버 등록/해제 가능
3. **재사용성**: 주제와 옵저버를 독립적으로 재사용 가능

## 단점
1. **복잡성 증가**: 많은 옵저버가 있을 때 성능 저하 가능
2. **순환 참조**: 옵저버가 다시 주제를 업데이트하는 경우 무한 루프 위험
3. **디버깅 어려움**: 간접적인 호출로 인한 흐름 추적 곤란

## 주의사항
- **순환 실행 방지**: 이벤트 처리 중 재귀 호출 방지 메커니즘 필요
- **메모리 누수 방지**: 옵저버 해제를 명시적으로 수행
- **스레드 안전성**: 멀티스레드 환경에서는 동기화 고려

## Java 내장 지원
```java
// Java의 Observable 클래스와 Observer 인터페이스 활용
import java.util.Observable;
import java.util.Observer;

class WeatherData extends Observable {
    private float temperature;

    public void setTemperature(float temperature) {
        this.temperature = temperature;
        setChanged();           // 변화 상태 설정
        notifyObservers(temperature);  // 알림 전송
    }
}
```

옵저버 패턴은 **느슨한 결합**을 통해 유연하고 확장 가능한 시스템을 구축할 수 있게 해주는 핵심적인 디자인 패턴이다.