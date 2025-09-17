

## Custom Decorator?

NestJS에서 Custom Decorator는 기본적으로 메타데이터를 함수나 클래스에 붙여서 특별한 동작을 하게 만드는 기능이다. 스프링 부트 써본 사람이라면 `@RequestMapping`, `@Service` 같은 어노테이션을 생각하면 된다.

TypeScript의 Decorator 문법을 활용해서 코드를 더 선언적으로 만들 수 있고, 중복되는 로직들을 깔끔하게 분리할 수 있다.

## 동작
핵심은 **메타데이터**다. `SetMetadata`로 함수에 데이터를 붙이고, `Reflector`로 런타임에 그 데이터를 읽어오는 구조다.

```typescript
export const MessageType = (type: string) => SetMetadata('messageType', type);

@MessageType('chat')
handleMessage() { /* ... */ }
```

이렇게 하면 `handleMessage` 함수에 `messageType: 'chat'`이라는 메타데이터가 붙는다.

## SocketIO에서 활용

내가 예전에 SocketIO 프로젝트 할 때 스프링 부트의 `@RequestMapping` 느낌으로 소켓 메시지 타입별로 핸들러를 만들고 싶었다. 그래서 이런 식으로 설계했다:

- `@SocketMessage('chat')`: 메시지 타입 지정
- `@RoomBroadcast()`: 룸 단위 브로드캐스팅
- `@AllBroadcast()`: 전체 브로드캐스팅

이렇게 하니까 새로운 메시지 타입 추가할 때 딱 decorator만 붙이면 끝이었다. 설정 파일 건드릴 필요도 없고, 핸들러 등록하는 코드 따로 짤 필요도 없었다.

## 앱 시작할 때 자동으로 스캔하기

NestJS의 `DiscoveryService`를 쓰면 앱이 시작될 때 모든 provider를 돌면서 특정 decorator가 붙은 메서드들을 찾을 수 있다. 스프링 부트의 component scan이랑 똑같은 개념이다.

```typescript
@Injectable()
export class SocketMessageScanner implements OnModuleInit {
  onModuleInit() {
    // 모든 provider 스캔해서 @SocketMessage 붙은 메서드 찾기
    // 찾으면 자동으로 핸들러 등록
  }
}
```

이렇게 하면 개발자는 그냥 decorator만 붙이면 되고, 나머지는 프레임워크가 알아서 처리해준다.

## 왜 이렇게 하는게 좋을까?

### 1. 선언적 프로그래밍
코드만 봐도 "아, 이 함수는 chat 메시지를 처리하고 룸에 브로드캐스트하는구나"라고 바로 알 수 있다. 비즈니스 로직과 인프라 로직이 깔끔하게 분리된다.

### 2. 확장성
새로운 기능 추가할 때 decorator만 만들면 된다. 기존 코드는 건드릴 필요 없다.

### 3. 재사용성
`@RequireAuth`, `@LogMessage` 같은 decorator를 만들어두면 여러 곳에서 조합해서 쓸 수 있다.

## 사용 코드

```typescript
@SocketMessage('chat')
@RoomBroadcast()
@RequireAuth()
@LogMessage()
handleChatMessage(client, data) {
  // 여기서는 순수한 비즈니스 로직만
  return { message: data.message };
}
```

이렇게 decorator를 여러 개 조합하면:
- 인증 체크
- 로깅
- 룸 브로드캐스팅
- 메시지 타입 라우팅

자동으로 처리된다. 개발자는 핵심 로직에만 집중하면 된다.

## 성능은 어떨까?

메타데이터 읽기에는 reflection이 들어가서 약간의 오버헤드가 있다. 하지만 앱 시작할 때 한 번만 스캔해서 캐싱해두면 런타임 성능에는 거의 영향 없다.

타입스크립트의 타입 시스템과도 잘 맞아서 컴파일 타임에 많은 에러를 잡을 수 있다.

## 정리하면

Custom Decorator는 NestJS에서 코드를 더 선언적이고 깔끔하게 만들어주는 도구다. 특히 SocketIO 같은 실시간 통신에서는 메시지 타입별 라우팅을 엄청 elegant하게 처리할 수 있다.

핵심은 메타데이터 기반으로 비즈니스 로직과 인프라 로직을 분리하는 것. 이렇게 하면 코드 읽기도 쉽고, 유지보수도 편하고, 확장하기도 좋다.