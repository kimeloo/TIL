

## NestJS EventEmitter2 사용법

### 설치
```bash
npm install @nestjs/event-emitter
```

### 모듈 설정
```typescript
import { Module } from '@nestjs/common';
import { EventEmitterModule } from '@nestjs/event-emitter';

@Module({
  imports: [
    EventEmitterModule.forRoot()
  ],
})
export class AppModule {}
```

### Event 정의
```typescript
// events/user-created.event.ts
export class UserCreatedEvent {
  constructor(
    public readonly userId: string,
    public readonly email: string,
  ) {}
}
```

### Event Emit하기
```typescript
import { Injectable } from '@nestjs/common';
import { EventEmitter2 } from '@nestjs/event-emitter';

@Injectable()
export class UserService {
  constructor(private eventEmitter: EventEmitter2) {}

  async createUser(userData: any) {
    // 사용자 생성 로직
    const user = await this.userRepository.save(userData);

    // 이벤트 발생
    this.eventEmitter.emit(
      'user.created',
      new UserCreatedEvent(user.id, user.email)
    );

    return user;
  }
}
```

### Event 수신하기 (@OnEvent)
```typescript
import { Injectable } from '@nestjs/common';
import { OnEvent } from '@nestjs/event-emitter';
import { UserCreatedEvent } from './events/user-created.event';

@Injectable()
export class NotificationService {
  @OnEvent('user.created')
  handleUserCreatedEvent(event: UserCreatedEvent) {
    console.log(`새 사용자 생성: ${event.email}`);
    // 환영 이메일 발송 로직
  }

  @OnEvent('user.created', { async: true })
  async handleUserCreatedAsync(event: UserCreatedEvent) {
    // 비동기 처리
    await this.sendWelcomeEmail(event.email);
  }
}
```

### 와일드카드 패턴
```typescript
@OnEvent('user.*')
handleAnyUserEvent(event: any) {
  console.log('사용자 관련 이벤트 발생');
}

@OnEvent('*.created')
handleAnyCreatedEvent(event: any) {
  console.log('생성 이벤트 발생');
}
```

## JavaScript 기본 EventEmitter

### Node.js EventEmitter
```javascript
const EventEmitter = require('events');

class MyEmitter extends EventEmitter {}
const myEmitter = new MyEmitter();

// 이벤트 리스너 등록
myEmitter.on('event', (data) => {
  console.log('이벤트 수신:', data);
});

// 한 번만 실행되는 리스너
myEmitter.once('special', (data) => {
  console.log('특별 이벤트:', data);
});

// 이벤트 발생
myEmitter.emit('event', { message: 'Hello World' });
myEmitter.emit('special', { type: 'special' });
```

### 브라우저 CustomEvent
```javascript
// 커스텀 이벤트 생성
const customEvent = new CustomEvent('myEvent', {
  detail: { message: 'Hello from custom event' }
});

// 이벤트 리스너 등록
document.addEventListener('myEvent', (event) => {
  console.log('Custom event received:', event.detail);
});

// 이벤트 발생
document.dispatchEvent(customEvent);
```

## 고급 사용법

### 에러 핸들링
```typescript
@OnEvent('user.created')
handleUserCreated(event: UserCreatedEvent) {
  try {
    // 이벤트 처리 로직
  } catch (error) {
    console.error('이벤트 처리 중 오류:', error);
    // 에러 이벤트 발생
    this.eventEmitter.emit('user.creation.failed', {
      originalEvent: event,
      error
    });
  }
}
```

### 이벤트 우선순위
```typescript
@OnEvent('user.created', { priority: 1 })
highPriorityHandler(event: UserCreatedEvent) {
  // 높은 우선순위로 먼저 실행
}

@OnEvent('user.created', { priority: 0 })
normalPriorityHandler(event: UserCreatedEvent) {
  // 일반 우선순위로 나중에 실행
}
```

### 조건부 이벤트 처리
```typescript
@OnEvent('user.created')
handleUserCreated(event: UserCreatedEvent) {
  if (event.email.includes('@company.com')) {
    // 회사 이메일인 경우에만 처리
    this.processCompanyUser(event);
  }
}
```

## 장점과 사용 사례

### 장점
- **느슨한 결합**: 모듈 간 직접적인 의존성 제거
- **확장성**: 새로운 이벤트 핸들러 쉽게 추가
- **재사용성**: 하나의 이벤트를 여러 핸들러가 처리 가능
- **비동기 처리**: 이벤트 기반 비동기 작업 처리

### 사용 사례
- 사용자 생성 시 환영 이메일 발송
- 주문 완료 시 재고 업데이트 및 알림 발송
- 로그 기록 및 모니터링
- 캐시 무효화
- 외부 서비스 연동

## 주의사항

### 메모리 누수 방지
```typescript
// 리스너 제거
this.eventEmitter.removeListener('user.created', this.handler);

// 모든 리스너 제거
this.eventEmitter.removeAllListeners('user.created');
```

### 순환 이벤트 방지
```typescript
@OnEvent('user.updated')
handleUserUpdated(event: UserUpdatedEvent) {
  // 무한 루프 방지를 위해 조건 체크
  if (!event.triggeredBySystem) {
    // 처리 로직
  }
}
```

이벤트 기반 아키텍처는 모듈간 결합도를 낮추고 확장 가능한 애플리케이션을 만드는 데 매우 유용한 패턴입니다.