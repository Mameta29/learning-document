# Redis を使ったセッション管理の構築手順

## Redis とは

Redis（Remote Dictionary Server）は、高速で柔軟なオープンソースのインメモリデータストア。データベース、キャッシュ、メッセージブローカーとして利用でき、キーと値のペアを使ってデータを保存する。特に、高速な読み書き性能を活かしてセッション管理やキャッシュとして広く利用されている。

## 前提条件

以下のツールがインストールされていることを前提とする：

- Node.js
- npm
- Redis サーバー

## セッション管理の構築手順

### ステップ 1: 必要なパッケージのインストール

```sh
npm install fastify ioredis bcrypt
```

### ステップ 2: Redis クライアントのセットアップ

`redis.ts`というファイルを作成し、以下のコードを記述：

```typescript
import Redis from 'ioredis';

const redis = new Redis({
  host: process.env.REDIS_HOST || '127.0.0.1',
  port: Number(process.env.REDIS_PORT) || 6379,
  password: process.env.REDIS_PASSWORD || undefined,
});

export default redis;
```

### ステップ 3: ユーザー登録機能の実装

`UserService.ts`を作成し、以下のコードを記述：

```typescript
import { hash } from 'bcrypt';
import { dbInstance as db } from '../../database';

class UserService {
  async createUser(
    username: string,
    email: string,
    password: string
  ): Promise<void> {
    const hashedPassword = await hash(password, 10);
    await db
      .insertInto('user')
      .values({ username, email, password: hashedPassword })
      .execute();
  }

  async findUserByEmail(
    email: string
  ): Promise<{
    id: number;
    username: string;
    email: string;
    password: string;
  } | null> {
    const user = await db
      .selectFrom('user')
      .selectAll()
      .where('email', '=', email)
      .executeTakeFirst();
    return user as unknown as {
      id: number;
      username: string;
      email: string;
      password: string;
    } | null;
  }
}

export const userService = new UserService();
```

### ステップ 4: セッション管理の実装

`SessionService.ts`を作成し、以下のコードを記述：

```typescript
import redis from '../../redis';
import { randomBytes } from 'crypto';

class SessionService {
  async createSession(userId: number): Promise<string> {
    const sessionId = randomBytes(16).toString('hex');
    const now = new Date().toISOString();

    await redis.set(
      sessionId,
      JSON.stringify({ userId, createdAt: now, updatedAt: now }),
      'EX',
      60 * 60 * 24
    ); // 24時間有効

    return sessionId;
  }

  async findSessionById(
    sessionId: string
  ): Promise<{ userId: number; createdAt: string; updatedAt: string } | null> {
    const sessionData = await redis.get(sessionId);
    if (!sessionData) {
      return null;
    }
    return JSON.parse(sessionData);
  }
}

export const sessionService = new SessionService();
```

### ステップ 5: 認証コントローラーの実装

`AuthController.ts`を作成し、以下のコードを記述：

```typescript
import { FastifyRequest, FastifyReply } from 'fastify';
import { userService } from '../services/UserService';
import { sessionService } from '../services/SessionService';
import { compare } from 'bcrypt';

export class AuthController {
  static async login(request: FastifyRequest, reply: FastifyReply) {
    const { email, password } = request.body as {
      email: string;
      password: string;
    };

    const user = await userService.findUserByEmail(email);
    if (!user || !(await compare(password, user.password))) {
      return reply.code(401).send({ message: '認証に失敗しました。' });
    }

    const sessionId = await sessionService.createSession(user.id);
    reply.setCookie('sessionId', sessionId, { httpOnly: true, path: '/' });
    reply.send({ message: 'ログインに成功しました。' });
  }
}
```

### ステップ 6: Redis サーバーの起動

```sh
redis-server
```

### ステップ 7: 環境変数の設定

`.env`ファイルに Redis の接続情報を追加：

```env
DATABASE_URL="file:./data/dev.db"
PORT=3000
REDIS_HOST=127.0.0.1
REDIS_PORT=6379
REDIS_PASSWORD=yourpassword
```

### ステップ 8: サーバーの起動

```sh
npm run dev
```

### ステップ 9: ログイン API のテスト

curl コマンドを使用してログイン API をテスト：

```sh
curl -X POST http://localhost:3000/login \
-H "Content-Type: application/json" \
-d '{
  "email": "sample@example.com",
  "password": "password"
}'
```

この API が正常に動作している場合、セッション ID が Cookie に設定され、Redis にセッションデータが保存される。
