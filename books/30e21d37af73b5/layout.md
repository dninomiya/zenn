---
title: 'レイアウト'
---

_実装の解説において、スタイルの部分は省略し機能的な部分に焦点を絞ります。_

# ヘッダーを作成

```tsx:app/components/header.tsx
import { Button } from '@/components/ui/button';

export default function Header() {
  return (
    <header>
      <span>ロゴ</span>
      <SignedIn>
        <UserButton />
      </SignedIn>
      <SignedOut>
        <SignInButton>
          <Button variant="outline">ログイン</Button>
        </SignInButton>
        <SignUpButton>
          <Button>新規登録</Button>
        </SignUpButton>
      </SignedOut>
    </header>
  );
}
```

# レイアウトに設置

```tsx:app/layouts.tsx
<ClerkProvider>
  <body lang="ja">
    <Header />
    <main>{children}</main>
  </body>
</ClerkProvider>
```

以後全画面でヘッダーが表示され、ログイン、ログアウト機能も実装されました。

# Clerk のカスタマイズについて

今回の例では Clerk が用意する UI やページをそのまま利用しましたが、Headless UI 的にフルカスタマイズも可能です。詳細は[公式ドキュメント](https://docs.clerk.dev/)を参照してください。

また、現在 Clerk が用意するログイン、登録ページは英語 UI ですがゆくゆく日本語対応される見込みです。
