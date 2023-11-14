---
title: メタ、OG Image
---

# Meta

以下が最小構成です。

```tsx:app/layout.tsx
import { Metadata } from 'next';

export const metadata: Metadata = {
  title: {
    default: 'サイトタイトル',
    template: `%s | サイトタイトル`,
  },
  description: '',
  openGraph: {
    title: {
      default: 'サイトタイトル',
      template: `%s | サイトタイトル`,
    },
    description: '',
  },
  twitter: {
    card: 'summary_large_image',
  },
};

// ...
```

各ページでは以下のように設定

```tsx:app/pages/index.tsx
export const metadata: Metadata = {
  title: 'ページタイトル',
}
```

こうすることで、トップページでは `サイトタイトル` が、それ以外のページでは `ページタイトル | サイトタイトル` が `<title>` に設定されます。

## 動的ページの場合

```tsx:app/posts/[slug]/page.tsx
export async function generateMetadata(
  {
    params: { slug },
  }: {
    params: { slug: string };
  },
  parent: ResolvingMetadata
) {
  const doc = await getPost(slug); // DB から記事を取得
  const parentMeta = await parent;

  if (!doc) {
    return {};
  }

  return {
    title: doc.title,
    description: doc.description || parentMeta.description,
    openGraph: {
      ...parentMeta.openGraph,
      title: doc.title,
      description: doc.description || parentMeta.description,
      type: 'article',
    },
  };
}
```

# OG Image

ベストなサイズは諸説ありますがそもそもサイトごとに勝手にトリミングされたりするので深く考えず、16:9 とかで作っておけば OK です。端スレスレに枠を置いたりトリミングされたら崩れるようなデザインは避けましょう。

作った画像を opengraph-image.png & twitter-image.png という名前でそれぞれ作成し、`/app` に設置します。（複製で OK）

最終的にこのような状態があれば OK です。

![](/images/meta-files.png)
