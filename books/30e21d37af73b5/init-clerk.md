---
title: Clerk の準備
---

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
    <ClerkProvider afterSignInUrl="/" afterSignUpUrl="/onboarding">
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

## 主要コンポーネントの解説

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

![](/images/clerk-account-portal.png)

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

# サーバーアクション用に認証ガードを作る

以下のファイルを作成します。これを使うことでサーバーアクションごとに認証ガードを実装できます。

```ts:app/actions/auth.ts
'use server';

import { auth } from '@clerk/nextjs';

export const authGuard = () => {
  const { userId } = auth();
  if (!userId) {
    throw new Error('You must be signed in to add an item to your cart');
  }

  return userId;
};
```
