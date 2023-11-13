---
title: アナリティクス
---

![](/images/analytics.webp)

[Vercel Analytics](https://vercel.com/docs/analytics/quickstart) は GA と違いクッキーを使用しないのでクッキーブロッカーの影響を受けずにアクセス数を計測できます。

```bash
bun add @vercel/analytics
```

```diff tsx:app/layout.tsx
+ import { Analytics } from '@vercel/analytics/react';

...

+   <Analytics />
  </body>
```

以後 Vercel ダッシュボードからアクセス数を確認できます。

特定のクリックイベントなどを計測したい場合以下のようにします。

```diff tsx
+ import { track } from '@vercel/analytics';

function SignupButton() {
  return (
    <button
      onClick={() => {
+         track('Signup');
        // ... other logic
      }}
    >
      Sign Up
    </button>
  );
}
```
