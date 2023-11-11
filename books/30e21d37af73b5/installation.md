---
title: '環境構築'
---

# Bun のインストール

npm/pnpm/yarn より早いので [Bun](https://bun.sh) をパッケージマネージャとしてインストールします。

```bash
npm i -g bun
```

# Next.js プロジェクトの作成

```bash
bunx create-next-app@latest
```

質問の回答は以下（すべてデフォルトなので Enter 連打で OK）

```bash
What is your project named? my-app
Would you like to use TypeScript? Yes
Would you like to use ESLint? Yes
Would you like to use Tailwind CSS? Yes
Would you like to use `src/` directory? No
Would you like to use App Router? (recommended) Yes
Would you like to customize the default import alias (@/*)? Yes
What import alias would you like configured? @/*
```

これにより [Tailwind CSS](https://tailwindcss.com) も初期化されます。

# shadcn/ui の追加

UI の整備が楽になるので [shadcn/ui](https://ui.shadcn.com/docs/installation/next) を追加します。(シャッドシーエヌユーアイと発音します)

- ライブラリではないのでフルカスタム可能
- ライブラリではないので依存せず、ロックされない
- コンポーネント単位でスニペットが丸ごとプロジェクトディレクトリにコピペされる形になる
- テーマカラーなのが CSS 変数化されているので、UI 全体で色や角丸などを一元管理できる
- ダークモード対応
- [Radix UI](https://www.radix-ui.com) & [Taiwlind CSS](https://tailwindcss.com) ベース

```bash
bunx shadcn-ui@latest init
```

質問は以下のように回答。基本デフォルトで良いのですが、 `tailwind.config.js` は `tailwind.config.ts` に変更してください。こうしないと `tailwind.config.js` が重複して作成されます。[近日修正入るようです。](https://github.com/shadcn-ui/ui/pull/1247)

```bash
Would you like to use TypeScript (recommended)? yes
Which style would you like to use? › Default
Which color would you like to use as base color? › Slate
Where is your global CSS file? › › app/globals.css
Do you want to use CSS variables for colors? › yes
Where is your tailwind.config.js located? › tailwind.config.ts
Configure the import alias for components: › @/components
Configure the import alias for utils: › @/lib/utils
Are you using React Server Components? › yes
```

以後[コンポーネント一覧](https://ui.shadcn.com/docs/components/accordion)から欲しいコンポーネントを選んでインストールします。たとえば Button を使いたい場合は

```bash
shadcn-ui@latest add button
```

として、以下のように使います。

```tsx
import { Button } from '@/components/ui/button';

export default function Home() {
  return (
    <div>
      <Button>Click me</Button>
    </div>
  );
}
```

細かい使い方は各コンポーネントのページを参照。[拡張機能](https://marketplace.visualstudio.com/items?itemName=SuhelMakkad.shadcn-ui)を使うと直感的にインストールや Doc の参照ができるのでおすすめです。

# Clerk の初期化

[Clerk](https://clerk.com) を使えば認証機能を簡単に実装できます。

- ユーザー管理画面あり
- ログイン、ログアウト画面（機能あり）
- 外部アカウント連携（追加、削除）あり
- パスワードリセット、メール認証、電話認証あり
- LINE 含め大抵のプロバイダーに対応
- 無料枠あり
- 将来的にデータを抜いて離脱できるのでロックインが緩い
- Vercel Conf でしばしばフューチャーされる（スポンサーなのもあるが）

これを使わない場合 [Auth.js](https://authjs.dev) を使ってください。

## インストール

```bash
bun add @clerk/nextjs
```

Clerk ダッシュボードでプロジェクトを作成（無料）し、認証キーを取得し、`.env.local` に追加します。

```ini:.env.local
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=your_publishable_key
CLERK_SECRET_KEY=your_secret_key
```

## プロバイダーの設置

`<ClerkProvider>`配下でのみ認証機能が使えます。通常はルートレイアウトに設置します。

```tsx:app/layouts.tsx
import { ClerkProvider } from '@clerk/nextjs'

export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <ClerkProvider>
      <html lang="ja">
        <body>{children}</body>
      </html>
    </ClerkProvider>
  )
}
```

## middleware の作成

```ts:middleware.ts
import { authMiddleware } from "@clerk/nextjs";

export default authMiddleware({
  // 公開ルートを指定（利用規約やログイン画面などログイン不要でアクセスできるルート）
  publicRoutes: ["/", '/terms'],
});

export const config = {
  matcher: ['/((?!.+\\.[\\w]+$|_next).*)', '/', '/(api|trpc)(.*)'],
};
```

これだけで認証ガードの実装は完了です。

これにより未ログイン時に publicRoutes 以外にアクセスすると Clerk のログイン画面にリダイレクトされます。カラーやロゴなどカスタムしてブランドに寄せることが可能です。あるいはログイン画面をヘッドレス的にスクラッチし、アプリ内に設置することも可能です。この辺りは Stripe Payments と Stripe Checkout の違いに似ています。

## 各種 UI の設置

ログイン、ログアウトにより表示の切り替えはもちろん、よく見る右上のユーザーメニューやログイン、ログアウトボタンが用意されています。カスタマイズ可能です。あるいはヘッドレス的に完全に自分で作って機能をアタッチすることも可能です。

```tsx
import {
  ClerkProvider,
  SignedIn,
  SignedOut,
  SignInButton,
  UserButton,
} from "@clerk/nextjs";

<SignedIn>
  {/* ログイン中に表示されるブロック */}
  <UserButton />
</SignedIn>
<SignedOut>
  {/* ログアウト中に表示されるブロック */}
  <SignInButton />
  <SignUpButton />
</SignedOut>
```

## プロフィール画面へのアクセス

- SNS 連携の管理（追加、削除）
- アバターの変更
- ログイン用メールアドレスの変更
- アカウント削除

など一般的なアカウント管理機能を備えたスタンドアロンのユーザープロフィールポータルが用意されています。ダッシュボードのサイドメニューから「Account Portal」を選択肢、ユーザープロフィールの URL を環境変数にセットしてアプリ内からリンクさせます。以下実装例。

```ini:.env.local
NEXT_PUBLIC_CLERK_USER_PROFILE=***
```

```tsx
<a href={process.env.NEXT_PUBLIC_CLERK_USER_PROFILE as string} target="_blank">
  アカウント設定
</a>
```

こちらもヘッドレス的にスクラッチし、アプリ内に実装することも可能です。そのあたりは割愛するので[ドキュメント](https://clerk.com/docs)を参照してください。

# Vercel の準備

Vercel プロジェクトを作成の上、設定画面から Function Region を **Tokyo or Osaka, Japan** にします。同時に Storages で Vercel Postgres を作成し、今回作成したプロジェクトと紐付けます。（UI を見ればわかりそうなので割愛します）

![](/images/vercel-region.png)

## DB アクセスに必要な環境変数の取得

プロジェクトディレクトリで以下を実行

```bash
# Vercel CLI がない場合
bun add -g vercel@latest

vercel env pull .env.development.local
```

これにより `.env.development.local` が作成されるので、その中から以下をコピーして `.env.local` に貼り付けます。

```ini:.env.local
POSTGRES_URL=
POSTGRES_PRISMA_URL=
POSTGRES_URL_NON_POOLING=
POSTGRES_USER=
POSTGRES_HOST=
POSTGRES_PASSWORD=
POSTGRES_DATABASE=
```

終わったら `.env.development.local` は削除します。

# Prisma の設定

```bash
bun add prisma @prisma/client
```

schema の作成

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

以上で環境構築は終了です。
