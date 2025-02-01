# 初めに  
  
前自分で書いたDIの記事を見て、いろいろと説明が足りてないと思うところがあったので、  
改めて記事を書いてみました。  
  
https://qiita.com/Nicasdream0/items/483fce259f0e94a25377  
  
## 対象読者  
  
- DIの基本概念を理解したいエンジニア  
- 設計パターンとしてDIを使うことを検討しているエンジニア  
  
## まず結論  
  
DI（Dependency Injection）とはコードの一部が他のコードに依存している場合、その依存関係を外部から注入するデザインパターン。  
  
- **メリット**：  
  - コードの柔軟性が向上し、変更やテストがしやすくなります。  
- **デメリット**：  
  - テストを書かない場合は冗長な実装になります。  
  
# Dependencyの管理にかかわる要素  
  
Dependency (依存)にかかわる要素について解説します。  
  
## Dependency（依存）  
  
Dependency（依存）とは、あるクラスが他のクラスの機能を必要とする関係を指します。  
例えば、クラスAがクラスBのインスタンスを作成して使用している場合、クラスAはクラスBに依存しています。  
  
```typescript
class Service {
  getMessage() {
    return "Hello, Dependency!";
  }
}

class Client {
  private service: Service;

  constructor() {
    this.service = new Service();
  }

  showMessage() {
    console.log(this.service.getMessage());
  }
}

const client = new Client();
client.showMessage();
```  
  
**ポイント**  
  
- `Client`クラスは直接`Service`クラスがないと動きません(`Client` ← `Service`の依存関係がある。)  
- `Service`クラスの変更があると`Client`クラスにも変更が必要になります。  
  
## Dependency Injection（依存性の注入）  
  
Dependency Injectionとは、依存を外部から注入するデザインパターンです。  
クラスAがクラスBのインスタンスを外部から受け取る場合、クラスAに依存性(クラスB)を注入したといえます。  
  
```typescript
class Service {
  getMessage() {
    return "Hello, DI!";
  }
}

class Client {
  private service: Service;

  constructor(service: Service) {
    this.service = service;
  }

  showMessage() {
    console.log(this.service.getMessage());
  }
}

const service = new Service();
const client = new Client(service);
client.showMessage();
```  
  
**ポイント**  
  
- `Client`クラス内で`Service`クラスのインスタンスを作成する必要がありません。  
- `Service`クラス以外を`Client`クラスで使いたい場合、`Client`クラスに渡すインスタンスを変えるだけで済みます。  
  
## Dependency Inversion Principle（依存性逆転の原則）  
  
Dependency Inversion PrincipleはSOLID原則の一つで、クラスは別のクラスに依存するべきではなく、両者は抽象(インターフェイス)に依存するべきという考え方です。  
  
インターフェイスとはクラスのメソッドの型のみを定義したものです。  
クラスはインターフェイスを継承(implements)して実装されます。  
  
Dependency InjectionとDependency Inversion Principleは併用されることが多いです。  
  
**service.ts**  
  
```typescript
interface IService {
  getMessage(): string;
}

class Service implements IService {
  getMessage(): string {
    return "Hello, Dependency Inversion!";
  }
}
```  
  
**client.ts**  
  
```typescript
class Client {
  private service: IService;

  constructor(service: IService) {
    this.service = service;
  }

  showMessage() {
    console.log(this.service.getMessage());
  }
}

const service = new Service();
const client = new Client(service);
client.showMessage();
```  
  
**ポイント**  
  
- `Client`クラスは`IService`インターフェイスという抽象的なものに依存し、具体的な実装に依存しないため、変更に強い設計になります。  
  
# JavascriptでのDIの実装例  
  
よく使われそうなNest.jsとInversifyJSを例に挙げて説明します。  
  
## Nest.jsでのDIの実装  
  
**service.module.ts**  
  
```typescript
import { Module } from '@nestjs/common';
import { Service } from './service';

@Module({
  providers: [Service],
  exports: [Service],
})
export class ServiceModule {}
```  
  
**service.ts**  
  
```typescript
import { Injectable } from '@nestjs/common';

@Injectable()
export class Service {
  getMessage(): string {
    return "Hello, Nest.js DI!";
  }
}
```  
  
**app.controller.ts**  
  
```typescript
import { Controller, Get } from '@nestjs/common';
import { Service } from './service';

@Controller()
export class AppController {
  constructor(private readonly service: Service) {}

  @Get()
  getMessage(): string {
    return this.service.getMessage();
  }
}
```  
  
**app.module.ts**  
  
```typescript
import { Module } from '@nestjs/common';
import { AppController } from './app.controller';
import { ServiceModule } from './service.module';

@Module({
  imports: [ServiceModule],
  controllers: [AppController],
})
export class AppModule {}
```  
  
1. `service.module.ts`：`Service`クラスを提供するモジュールを定義しています。  
2. `service.ts`：`Service`クラスはメッセージを返すメソッド`getMessage`を持っています。  
3. `app.controller.ts`：`AppController`クラスは、`Service`クラスのインスタンスをコンストラクタで受け取り、HTTP GETリクエストに応じてメッセージを返します。  
4. `app.module.ts`：アプリケーションモジュールを定義し、`ServiceModule`と`AppController`をインポートしています。  
  
## InversifyJSでのDIの実装  
  
**inversify.config.ts**  
  
```typescript
import { Container } from 'inversify';
import { Service } from './service';
import { Client } from './client';
import { TYPES } from './types';

const container = new Container();
container.bind<Service>(TYPES.Service).to(Service);
container.bind<Client>(TYPES.Client).to(Client);

export { container };
```  
  
**service.ts**  
  
```typescript
import { injectable } from 'inversify';

@injectable()
export class Service {
  getMessage(): string {
    return "Hello, InversifyJS!";
  }
}
```  
  
**client.ts**  
  
```typescript
import { inject, injectable } from 'inversify';
import { Service } from './service';
import { TYPES } from './types';

@injectable()
export class Client {
  private service: Service;

  constructor(@inject(TYPES.Service) service: Service) {
    this.service = service;
  }

  showMessage() {
    console.log(this.service.getMessage());
  }
}
```  
  
**types.ts**  
  
```typescript
const TYPES = {
  Service: Symbol.for("Service"),
  Client: Symbol.for("Client")
};

export { TYPES };
```  
  
**app.ts**  
  
```typescript
import 'reflect-metadata';
import { container } from './inversify.config';
import { Client } from './client';
import { TYPES } from './types';

const client = container.get<Client>(TYPES.Client);
client.showMessage();
```  
  
1. `inversify.config.ts`：DIコンテナを設定し、`Service`と`Client`クラスをバインドします。  
2. `service.ts`：`Service`クラスは`@injectable`デコレーターを使用してDIコンテナに登録されています。  
3. `client.ts`：`Client`クラスは`Service`クラスのインスタンスをコンストラクタで受け取ります。`@inject`デコレーターを使用して、`Service`クラスのインスタンスを注入します。  
4. `types.ts`：DIコンテナで使用する識別子を定義します。  
5. `app.ts`：DIコンテナから`Client`クラスのインスタンスを取得し、`showMessage`メソッドを呼び出します。  
  
# DIの活用事例  
  
## テストコードにおいてテストクラスを使う  
  
テスト時に依存関係のあるクラスをモック化してInjectionすることで、自身のクラスのメソッドのロジックのテストのみに集中できます。  
  
**app.controller.spec.ts**  
  
```typescript
import { Test, TestingModule } from '@nestjs/testing';
import { AppController } from './app.controller';
import { Service } from './service';

describe('AppController', () => {
  let appController: AppController;
  let service: Service;

  beforeEach(async () => {
    // モックサービスの作成
    const mockService = {
      getMessage: jest.fn().mockReturnValue('Hello, Mocked DI!'),
    };

    const app: TestingModule = await Test.createTestingModule({
      controllers: [AppController],
      providers: [
        {
          provide: Service,
          useValue: mockService,
        },
      ],
    }).compile();

    appController = app.get<AppController>(AppController);
    service = app.get<Service>(Service);
  });

  describe('getMessage', () => {
    it('should return "Hello, Mocked DI!"', () => {
      expect(appController.getMessage()).toBe('Hello, Mocked DI!');
    });

    it('should call getMessage method of the service', () => {
      appController.getMessage();
      expect(service.getMessage).toHaveBeenCalled();
    });
  });
});
```  
  
1. **モックサービスの作成**:  
   - `jest.fn()`を使用して`Service`クラスのモックを作成し、その`getMessage`メソッドが`'Hello, Mocked DI!'`を返すように設定しています。  
  
2. **DIコンテナの設定**:  
   - `Test.createTestingModule`を使用してテストモジュールを設定し、`AppController`をコントローラーとして登録しています。  
   - `Service`プロバイダーに対してモックサービスを`useValue`オプションで提供しています。  
  
3. **依存関係の注入**:  
   - `app.get<AppController>(AppController)`で`AppController`のインスタンスを取得し、モックサービスが注入された状態でテストを実行します。  
   - 同様に、`app.get<Service>(Service)`で`Service`のインスタンスを取得します。  
  
4. **テストの実行**:  
   - `getMessage`メソッドが正しいメッセージを返すかをテストしています。  
   - `getMessage`メソッドが呼び出されたかを確認するテストを追加しています。  
  
## 環境ごとに依存関係を切り替える  
  
状況によって使用されるサービスクラスを切り替えたいような場面で、依存関係を切り替えることが容易になります。  
  
**app.module.ts**  
  
```typescript
import { Module } from '@nestjs/common';
import { AppController } from './app.controller';
import { Service } from './service';
import { AlternativeService } from './alternative.service';

@Module({
  imports: [],
  controllers: [AppController],
  providers: [
    {
      provide: Service,
      useClass: process.env.USE_ALTERNATIVE_SERVICE ? AlternativeService : Service,
    },
  ],
})
export class AppModule {}
```  
  
**service.ts**  
  
```typescript
import { Injectable } from '@nestjs/common';

@Injectable()
export class Service {
  getMessage(): string {
    return 'Hello, Service!';
  }
}
```  
  
**alternative.service.ts**  
  
```typescript
import { Injectable } from '@nestjs/common';

@Injectable()
export class AlternativeService {
  getMessage(): string {
    return 'Hello, Alternative Service!';
  }
}
```  
  
**app.controller.ts**  
  
```typescript
import { Controller, Get } from '@nestjs/common';
import { Service } from './service';

@Controller()
export class AppController {
  constructor(private readonly service: Service) {}

  @Get()
  getMessage(): string {
    return this.service.getMessage();
  }
}
```  
  
1. **app.module.ts**:  
   - 環境変数`USE_ALTERNATIVE_SERVICE`が設定されている場合、`AlternativeService`を使用し、そうでない場合は`Service`を使用します。  
   - `Service`プロバイダーに対して`useClass`オプションを使用して、実行時に依存関係を切り替えるように設定しています。  
  
2. **service.ts**:  
   - デフォルトのサービス実装です。`getMessage`メソッドが`'Hello, Service!'`を返します。  
  
3. **alternative.service.ts**:  
   - 代替サービス実装です。`getMessage`メソッドが`'Hello, Alternative Service!'`を返します。  
  
4. **app.controller.ts**:  
   - `Service`クラスのインスタンスをコンストラクタで受け取り、HTTP GETリクエストに応じてメッセージを返します。  
  
  
# まとめ  
  
DIのメリットとデメリットをまとめます。  
  
## DIのメリット  
  
### 依存関係の解決の責務を分離できる  
  
DIを使用することで、依存関係の解決をクラス自身から分離できます。これにより、クラスは自身の主な責務に集中でき、依存関係の管理は外部に委ねられます。  
  
### 環境による実装の切り替えが楽になる  
  
DIを使用すると、異なる環境や状況に応じて簡単に実装を切り替えることができます。例えば、開発環境ではモックサービス、本番環境では実際のサービスを注入することが可能です。  
  
### 複雑なテストの実装が楽になる  
  
DIにより、依存関係を簡単にモックに置き換えることができるため、テストの実装が容易になります。これにより、ユニットテストやインテグレーションテストの効率が向上します。  
  
## DIのデメリット  
  
### テストを実装しな場合、冗長になる  
  
DIを適切に使用しない場合、あまりうま味がありません。  
インターフェイスの実装が増えるので、かえって冗長になる可能性があります。  
  
### オーバーヘッド  
  
小規模なプロジェクトでは、DIの導入によるオーバーヘッドがパフォーマンスやコードのシンプルさに影響を与えることがあります。  
  
### 学習コスト  
  
DIパターンやフレームワークの使い方を理解するための学習コストが発生します。特に、新しいメンバーがプロジェクトに参加する際には、その学習が必要になります。  
  
# FAQ  
  
よくありそうな質問と回答内容を記載します。  
  
## jest.mockとDIによるテストクラスのinjectionの使い分けは  
  
依存関係の多い複雑なクラスのテストではテストクラスのinjectionのほうが適しています。  
また、テストクラスは再利用できるので、同じモックを複数のテストで使いまわしたい場合に有用です。  
  
## DIを使うべきでない場合はありますか？  
  
小規模なプロジェクトや単純なアプリケーションでは、DIのオーバーヘッドがデメリットになる場合があります。  
DIを使用しなくても依存関係が簡単に管理できる場合には、DIの導入は不要かもしれません。  
  
# 参考資料  
  
- [Introduction to Dependency Injection（DI）の整理とそのメリット](https://speakerdeck.com/revcomm_inc/introduction-to-dependency-injection-di-nozheng-li-tosonomerituto?slide=3)  
- [Dependency Injection in TypeScript with Nest.js](https://qiita.com/uhooi/items/03ec6b7f0adc68610426)  
- [「依存性逆転の原則」と「依存性の注入」を完全に理解した](https://qiita.com/uhooi/items/03ec6b7f0adc68610426)  
