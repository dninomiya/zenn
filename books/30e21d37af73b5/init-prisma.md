---
title: Prismaの準備
---

# Prisma の設定

```bash
bun add prisma @prisma/client
```

プロジェクトルートに以下のファイルを作成

```ts:schema.prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url = env("POSTGRES_PRISMA_URL")
  directUrl = env("POSTGRES_URL_NON_POOLING")
}

model User {
  // ID は Clerk のユーザーIDを使うのでデフォルトの乱数IDは使わない
  id        String      @id
  name      String
  createdAt DateTime @default(now())
  posts Post[]
}

model Post {
  // ID はサムネイル画像で使う都合マニュアル生成するので乱数IDは使わない
  id           String   @id
  thumbnailURL String?
  title        String   @db.VarChar(140)
  body         String   @db.VarChar(140)
  author       User     @relation(fields: [authorId], references: [id], onDelete: Cascade)
  authorId     String
  createdAt    DateTime @default(now())
}
```

prisma のマイグレーション

```bash
bun prisma migrate dev --name init
```

以後 `schema.prisma` を変更したら `bun prisma migrate dev` でマイグレーションを実行します。

通常 prisma のインストール時に schema を generate してくれますが、現在 Bun ではその動作が発生しないため以下を追記して明示的に generate するようにします。

```json:package.json
{
  "scripts": {
    "postinstall": "prisma generate"
    // ...
  }
}
```

# Prisma を簡単に使えるようにする

```ts:lib/prisma.ts
import { PrismaClient } from '@prisma/client';

declare global {
  var prisma: PrismaClient | undefined;
}

const prisma = global.prisma || new PrismaClient();

if (process.env.NODE_ENV === 'development') global.prisma = prisma;

export const db = prisma;
```

これにより、以後他のファイルから以下のようにデータベースにアクセスできます。

```ts
import { db } from '@/lib/prisma';

const posts = await db.post.findMany();
```
