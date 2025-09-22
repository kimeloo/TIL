# React xterm과 NestJS node-pty 연결

## 개요

React xterm.js와 NestJS node-pty를 WebSocket으로 연결하여 실시간 터미널을 구현하는 방법. 복잡한 버퍼링 없이 바로바로 전송하는 단순한 방식을 사용한다.

## 아키텍처

```
React Frontend (xterm.js) <---> WebSocket <---> NestJS Backend (node-pty)
```

- **Frontend**: xterm.js의 AttachAddon을 사용해 WebSocket에 직접 연결
- **Backend**: 각 WebSocket 연결마다 개별 pty 프로세스 생성
- **통신**: 키 입력 즉시 전송, pty 출력 즉시 전달

## 구현

### 1. 패키지 설치

```bash
# Frontend
npm install xterm xterm-addon-attach xterm-addon-fit

# Backend
npm install @nestjs/websockets @nestjs/platform-socket.io node-pty
npm install -D @types/node-pty
```

### 2. React 컴포넌트

```tsx
// Terminal.tsx
import React, { useEffect, useRef } from 'react';
import { Terminal } from 'xterm';
import { AttachAddon } from 'xterm-addon-attach';
import { FitAddon } from 'xterm-addon-fit';
import 'xterm/css/xterm.css';

const TerminalComponent: React.FC = () => {
  const terminalRef = useRef<HTMLDivElement>(null);
  const terminal = useRef<Terminal>();
  const socket = useRef<WebSocket>();

  useEffect(() => {
    if (!terminalRef.current) return;

    // 터미널 초기화
    terminal.current = new Terminal({
      cursorBlink: true,
      fontSize: 14,
      fontFamily: 'Monaco, Menlo, "Ubuntu Mono", monospace',
      theme: {
        background: '#1e1e1e',
        foreground: '#ffffff'
      }
    });

    // Fit addon 로드 (화면 크기에 맞춤)
    const fitAddon = new FitAddon();
    terminal.current.loadAddon(fitAddon);

    // DOM에 터미널 열기
    terminal.current.open(terminalRef.current);
    fitAddon.fit();

    // WebSocket 연결
    socket.current = new WebSocket('ws://localhost:3000/terminal');

    socket.current.onopen = () => {
      // AttachAddon으로 WebSocket 직접 연결
      const attachAddon = new AttachAddon(socket.current!);
      terminal.current!.loadAddon(attachAddon);
    };

    // 리사이즈 핸들러
    const handleResize = () => {
      fitAddon.fit();
      const { cols, rows } = terminal.current!;
      socket.current?.send(JSON.stringify({ type: 'resize', cols, rows }));
    };

    window.addEventListener('resize', handleResize);

    return () => {
      terminal.current?.dispose();
      socket.current?.close();
      window.removeEventListener('resize', handleResize);
    };
  }, []);

  return (
    <div
      ref={terminalRef}
      style={{
        height: '400px',
        width: '100%',
        padding: '10px',
        backgroundColor: '#1e1e1e'
      }}
    />
  );
};

export default TerminalComponent;
```

### 3. NestJS WebSocket Gateway

```typescript
// terminal.gateway.ts
import {
  WebSocketGateway,
  WebSocketServer,
  OnGatewayConnection,
  OnGatewayDisconnect,
  SubscribeMessage,
  MessageBody,
} from '@nestjs/websockets';
import { Server, Socket } from 'socket.io';
import * as pty from 'node-pty';
import * as os from 'os';

@WebSocketGateway({
  cors: { origin: '*' },
  namespace: 'terminal'
})
export class TerminalGateway implements OnGatewayConnection, OnGatewayDisconnect {
  @WebSocketServer()
  server: Server;

  private terminals = new Map<string, any>();

  handleConnection(client: Socket) {
    console.log(`터미널 연결: ${client.id}`);

    // 각 연결마다 새로운 pty 프로세스 생성
    const shell = os.platform() === 'win32' ? 'powershell.exe' : 'bash';
    const ptyProcess = pty.spawn(shell, [], {
      name: 'xterm-color',
      cols: 80,
      rows: 24,
      cwd: process.env.HOME || process.cwd(),
      env: process.env
    });

    this.terminals.set(client.id, ptyProcess);

    // pty 출력을 클라이언트로 바로 전송
    ptyProcess.onData((data) => {
      client.emit('data', data);
    });

    // 터미널 종료 처리
    ptyProcess.onExit((code, signal) => {
      console.log(`터미널 종료: ${code}, ${signal}`);
      client.disconnect();
    });
  }

  handleDisconnect(client: Socket) {
    console.log(`터미널 연결 해제: ${client.id}`);

    const ptyProcess = this.terminals.get(client.id);
    if (ptyProcess) {
      ptyProcess.kill();
      this.terminals.delete(client.id);
    }
  }

  @SubscribeMessage('data')
  handleData(client: Socket, @MessageBody() data: string) {
    const ptyProcess = this.terminals.get(client.id);
    if (ptyProcess) {
      // 입력을 pty로 바로 전송
      ptyProcess.write(data);
    }
  }

  @SubscribeMessage('resize')
  handleResize(client: Socket, @MessageBody() size: { cols: number; rows: number }) {
    const ptyProcess = this.terminals.get(client.id);
    if (ptyProcess) {
      ptyProcess.resize(size.cols, size.rows);
    }
  }
}
```

### 4. 순수 WebSocket 방식 (선택사항)

```typescript
// 만약 Socket.IO 대신 순수 WebSocket을 쓸 경우
import { WebSocketServer } from 'ws';
import * as pty from 'node-pty';
import * as os from 'os';

const wss = new WebSocketServer({ port: 8080 });

wss.on('connection', (ws) => {
  const shell = os.platform() === 'win32' ? 'powershell.exe' : 'bash';
  const ptyProcess = pty.spawn(shell, [], {
    name: 'xterm-color',
    cols: 80,
    rows: 24,
    cwd: process.env.HOME || process.cwd(),
    env: process.env
  });

  // pty → WebSocket
  ptyProcess.onData((data) => {
    if (ws.readyState === ws.OPEN) {
      ws.send(data);
    }
  });

  // WebSocket → pty
  ws.on('message', (data) => {
    ptyProcess.write(data.toString());
  });

  ws.on('close', () => {
    ptyProcess.kill();
  });
});
```

## 핵심 포인트

### 1. 바로바로 전송 방식
- `AttachAddon`이 키 입력을 WebSocket으로 즉시 전송
- node-pty가 출력을 즉시 WebSocket으로 전달
- 별도 버퍼링 없이 실시간 처리

### 2. Echo 처리
- node-pty가 자체적으로 echo 처리하므로 별도 구현 불필요
- 터미널 프로그램(bash, zsh 등)이 자연스럽게 echo 제공

### 3. 연결 관리
- 각 WebSocket 연결마다 독립적인 pty 프로세스
- 연결 해제 시 자동으로 pty 프로세스 정리
- 메모리 누수 방지

### 4. 리사이즈 처리
- FitAddon으로 터미널 크기 자동 조정
- 크기 변경 시 pty에 resize 신호 전송

## 보안 고려사항

```typescript
// 인증 추가 예시
@WebSocketGateway({
  cors: { origin: 'http://localhost:3000' }, // 특정 도메인만 허용
})
export class TerminalGateway {
  handleConnection(client: Socket) {
    // 토큰 검증
    const token = client.handshake.auth.token;
    if (!this.validateToken(token)) {
      client.disconnect();
      return;
    }
    // ... 터미널 생성
  }
}
```

## 실행

1. NestJS 서버 실행: `npm run start:dev`
2. React 앱 실행: `npm start`
3. 브라우저에서 터미널 컴포넌트 확인

node-pty의 echo 덕분에 복잡한 입출력 처리 없이도 자연스러운 터미널 경험을 제공할 수 있다.