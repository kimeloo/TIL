

## 1. 의존성 주입 개념

### 정의
**의존성 주입(Dependency Injection, DI)**은 객체가 필요한 의존성을 외부에서 주입받는 디자인 패턴입니다. NestJS는 IoC(Inversion of Control) 컨테이너를 통해 자동으로 의존성을 관리합니다.

### 기본 원리
```typescript
// ❌ 의존성 주입 없이 (직접 생성)
class UsersController {
  private usersService = new UsersService(); // 강한 결합
}

// ✅ 의존성 주입 사용
@Controller('users')
class UsersController {
  constructor(private readonly usersService: UsersService) {} // 느슨한 결합
}
```

### 장점
- **느슨한 결합**: 클래스 간의 의존성을 줄임
- **테스트 용이성**: Mock 객체로 쉽게 대체 가능
- **재사용성**: 다른 구현체로 교체 가능
- **생명주기 관리**: 자동으로 인스턴스 생명주기 관리

## 2. 핵심 키워드

### Provider
**Provider**는 NestJS에서 의존성으로 주입될 수 있는 클래스를 의미합니다.

```typescript
@Injectable()  // Provider로 만드는 데코레이터
export class UsersService {
  findAll() {
    return [];
  }
}

// 모듈에서 Provider 등록
@Module({
  providers: [UsersService],  // Provider 등록
})
export class UsersModule {}
```

### Import
다른 모듈을 현재 모듈에 가져와서 사용하는 키워드입니다.

```typescript
@Module({
  imports: [UsersModule],  // UsersModule의 exports를 사용 가능
  controllers: [AppController],
})
export class AppModule {}
```

### Export
모듈에서 다른 모듈이 사용할 수 있도록 내보내는 키워드입니다.

```typescript
@Module({
  providers: [UsersService],
  exports: [UsersService],    // 다른 모듈에서 UsersService 사용 가능
})
export class UsersModule {}
```

## 3. Provider와 Export의 관계

### 기본 관계
```typescript
@Module({
  providers: [UsersService],  // 1. 모듈 내에서 사용 가능한 Provider 등록
  exports: [UsersService],    // 2. 다른 모듈에서도 사용할 수 있도록 Export
})
export class UsersModule {}

@Module({
  imports: [UsersModule],     // 3. UsersModule을 Import
  controllers: [OrdersController],
})
export class OrdersModule {}

// OrdersController에서 UsersService 사용 가능
@Controller('orders')
export class OrdersController {
  constructor(private readonly usersService: UsersService) {}
}
```

### Export 규칙
- **Provider에 등록된 것만 Export 가능**
- **Export하지 않으면 해당 모듈 내부에서만 사용 가능**

```typescript
@Module({
  providers: [UsersService, InternalService],
  exports: [UsersService],  // InternalService는 외부에서 사용 불가
})
export class UsersModule {}
```

## 4. 싱글톤 패턴과 주의사항

### NestJS의 싱글톤 패턴
NestJS는 기본적으로 **싱글톤 패턴**을 사용합니다. 같은 Provider는 애플리케이션 전체에서 하나의 인스턴스만 생성됩니다.

```typescript
@Injectable()
export class UsersService {
  private users = [];  // 애플리케이션 전체에서 공유되는 상태
  
  addUser(user: any) {
    this.users.push(user);  // 상태가 유지됨
  }
  
  getUsers() {
    return this.users;
  }
}
```

### 싱글톤 사용 시 주의사항

#### 1. 상태 관리 주의
```typescript
// ❌ 위험한 상태 공유
@Injectable()
export class UserService {
  private currentUser: User;  // 전역 상태로 인한 문제 발생 가능
  
  setCurrentUser(user: User) {
    this.currentUser = user;  // 다른 요청에 영향을 줄 수 있음
  }
}

// ✅ 안전한 상태 관리
@Injectable()
export class UserService {
  private users: Map<string, User> = new Map();
  
  setUser(id: string, user: User) {
    this.users.set(id, user);  // ID별로 구분하여 관리
  }
  
  getUser(id: string) {
    return this.users.get(id);
  }
}
```

#### 2. 스레드 안전성
```typescript
// ❌ 동시성 문제 가능
@Injectable()
export class CounterService {
  private count = 0;
  
  increment() {
    this.count++;  // Race condition 발생 가능
    return this.count;
  }
}

// ✅ 안전한 방법
@Injectable()
export class CounterService {
  private count = 0;
  private readonly mutex = new Mutex();  // 동기화 객체 사용
  
  async increment() {
    return await this.mutex.runExclusive(() => {
      this.count++;
      return this.count;
    });
  }
}
```

### Request Scope Provider
필요에 따라 요청별로 새 인스턴스를 생성할 수 있습니다.

```typescript
@Injectable({ scope: Scope.REQUEST })
export class RequestScopedService {
  private requestData: any;
  
  setRequestData(data: any) {
    this.requestData = data;  // 요청별로 격리됨
  }
}
```

## 5. 모듈 계층 구조에서의 의존성 주입 주의점

### 계층 구조 예시
```
AppModule
├── UsersModule
│   ├── UsersService
│   └── UsersRepository
├── OrdersModule
│   ├── OrdersService
│   └── OrdersController
└── SharedModule
    └── DatabaseService
```

### 1. 순환 의존성 문제
```typescript
// ❌ 순환 의존성 발생
@Module({
  imports: [OrdersModule],  // OrdersModule을 Import
  providers: [UsersService],
  exports: [UsersService],
})
export class UsersModule {}

@Module({
  imports: [UsersModule],   // UsersModule을 Import (순환 참조)
  providers: [OrdersService],
})
export class OrdersModule {}
```

**해결책: forwardRef() 사용**
```typescript
@Module({
  imports: [forwardRef(() => OrdersModule)],
  providers: [UsersService],
  exports: [UsersService],
})
export class UsersModule {}

@Module({
  imports: [forwardRef(() => UsersModule)],
  providers: [OrdersService],
})
export class OrdersModule {}
```

### 2. 모듈 Export vs 서비스 Export

#### ❌ 잘못된 방법: 하위 모듈의 서비스를 직접 Export
```typescript
@Module({
  imports: [UsersModule],
  providers: [AppService],
  exports: [UsersService],  // ❌ 직접 import한 서비스를 export
})
export class AppModule {}
```

#### ✅ 올바른 방법: 모듈 자체를 re-export
```typescript
@Module({
  imports: [UsersModule],
  providers: [AppService],
  exports: [UsersModule],   // ✅ 모듈 자체를 export
})
export class AppModule {}
```

### 3. Global Module 사용
공통으로 사용되는 모듈은 Global로 설정할 수 있습니다.

```typescript
@Global()  // 전역 모듈로 설정
@Module({
  providers: [DatabaseService, ConfigService],
  exports: [DatabaseService, ConfigService],
})
export class SharedModule {}

// 다른 모듈에서 import 없이 사용 가능
@Module({
  // imports: [SharedModule],  // ✅ Global 모듈이므로 불필요
  controllers: [UsersController],
  providers: [UsersService],
})
export class UsersModule {}
```

### 4. Dynamic Module Pattern
설정에 따라 동적으로 모듈을 생성할 수 있습니다.

```typescript
@Module({})
export class DatabaseModule {
  static forRoot(options: DatabaseOptions): DynamicModule {
    return {
      module: DatabaseModule,
      providers: [
        {
          provide: 'DATABASE_OPTIONS',
          useValue: options,
        },
        DatabaseService,
      ],
      exports: [DatabaseService],
      global: true,  // 전역으로 설정
    };
  }
}

// 사용
@Module({
  imports: [
    DatabaseModule.forRoot({
      host: 'localhost',
      port: 5432,
    }),
  ],
})
export class AppModule {}
```

## 6. 의존성 주입 패턴과 Best Practices

### 1. Interface 기반 의존성 주입
```typescript
// 인터페이스 정의
interface IUsersRepository {
  findAll(): Promise<User[]>;
  findById(id: string): Promise<User>;
}

// 구현체
@Injectable()
export class UsersRepository implements IUsersRepository {
  async findAll(): Promise<User[]> {
    // 실제 구현
  }
  
  async findById(id: string): Promise<User> {
    // 실제 구현
  }
}

// 토큰 기반 주입
const USERS_REPOSITORY = Symbol('USERS_REPOSITORY');

@Module({
  providers: [
    {
      provide: USERS_REPOSITORY,
      useClass: UsersRepository,
    },
  ],
  exports: [USERS_REPOSITORY],
})
export class UsersModule {}

// 사용
@Injectable()
export class UsersService {
  constructor(
    @Inject(USERS_REPOSITORY)
    private readonly usersRepository: IUsersRepository,
  ) {}
}
```

### 2. Factory Provider
복잡한 초기화가 필요한 경우 사용합니다.

```typescript
@Module({
  providers: [
    {
      provide: 'DATABASE_CONNECTION',
      useFactory: async (configService: ConfigService) => {
        const config = configService.getDatabaseConfig();
        return await createDatabaseConnection(config);
      },
      inject: [ConfigService],
    },
  ],
})
export class DatabaseModule {}
```

### 3. 조건부 Provider
```typescript
@Module({
  providers: [
    {
      provide: 'CACHE_SERVICE',
      useFactory: (configService: ConfigService) => {
        if (configService.get('USE_REDIS')) {
          return new RedisService();
        }
        return new MemoryCacheService();
      },
      inject: [ConfigService],
    },
  ],
})
export class CacheModule {}
```

## 7. 주요 주의사항 요약

1. **Provider 등록**: exports에 포함하려면 반드시 providers에도 등록
2. **모듈 re-export**: 하위 모듈의 서비스가 아닌 모듈 자체를 export
3. **순환 의존성**: forwardRef() 사용하거나 구조 개선
4. **싱글톤 상태**: 전역 상태 관리 시 동시성 문제 주의
5. **Global 모듈**: 남용하지 말고 정말 전역적으로 필요한 것만 사용
6. **Interface 기반**: 테스트와 유지보수를 위해 인터페이스 기반 설계 권장