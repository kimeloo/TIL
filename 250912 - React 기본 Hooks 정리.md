

React는 함수형 컴포넌트에서 상태와 생명주기를 관리할 수 있게 해주는 여러 Hooks를 제공합니다.

## 주요 React Hooks

### 1. useState
컴포넌트에 상태 변수를 추가하는 Hook

```javascript
import { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);
  
  return (
    <button onClick={() => setCount(count + 1)}>
      Count: {count}
    </button>
  );
}
```

- **반환값**: [currentState, setState] 배열
- **용도**: 동적 데이터 관리, 리렌더링 트리거
- **특징**: 상태가 변경되면 컴포넌트가 다시 렌더링됨

### 2. useEffect
부수 효과(side effects)를 처리하는 Hook

```javascript
import { useEffect, useState } from 'react';

function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    
    // cleanup 함수
    return () => {
      connection.disconnect();
    };
  }, [serverUrl, roomId]); // 의존성 배열
  
  return <div>Chat Room</div>;
}
```

- **용도**: API 호출, 구독 설정, DOM 조작, 타이머 설정 등
- **의존성 배열**: 빈 배열 `[]`은 마운트시에만 실행, 생략하면 매 렌더시 실행
- **cleanup**: 컴포넌트 언마운트나 의존성 변경시 정리 작업

### 3. useRef
렌더링에 필요하지 않은 값을 참조하는 Hook

```javascript
import { useRef, useEffect } from 'react';

function VideoPlayer({ src, isPlaying }) {
  const ref = useRef(null);

  useEffect(() => {
    if (isPlaying) {
      ref.current.play();
    } else {
      ref.current.pause();
    }
  });

  return <video ref={ref} src={src} loop playsInline />;
}
```

- **반환값**: `{ current: initialValue }` 객체
- **용도**: DOM 요소 접근, 값 저장 (리렌더링 없이)
- **특징**: `.current` 값 변경시에도 리렌더링 되지 않음

### 4. 기타 주요 Hooks

#### useContext
React Context 값을 읽는 Hook

#### useReducer
복잡한 상태 로직을 관리하는 Hook (useState의 대안)

#### useMemo
계산 비용이 큰 값을 메모이제이션하는 Hook

#### useCallback
함수를 메모이제이션하는 Hook

#### useLayoutEffect
DOM 변경 후 동기적으로 실행되는 useEffect

## Hook 사용 규칙

1. **최상위에서만 호출**: 반복문, 조건문, 중첩 함수 안에서 호출 불가
2. **React 함수에서만 호출**: 컴포넌트나 커스텀 Hook 안에서만 사용
3. **커스텀 Hook**: `use`로 시작하는 이름으로 자체 Hook 생성 가능

## 내부 구현 개념

```javascript
// useRef의 개념적 구현
function useRef(initialValue) {
  const [ref, unused] = useState({ current: initialValue });
  return ref;
}
```

React의 Hooks는 함수형 컴포넌트에서 클래스 컴포넌트의 기능들을 대체하며, 더 간결하고 재사용 가능한 코드 작성을 가능하게 합니다.