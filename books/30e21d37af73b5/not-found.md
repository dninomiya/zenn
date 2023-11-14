---
title: 404ページ
---

デフォルトで Next.js が用意してくれていますが、 `not-found.tsx` を `/app` に作成するとことでオリジナルの 404 ページを作成できます。

```tsx:app/not-found.tsx
export default function NotFound() {
  return <div>404</div>;
}
```
