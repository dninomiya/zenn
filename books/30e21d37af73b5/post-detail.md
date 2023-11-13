---
title: 記事詳細画面の実装
---

```tsx:app/posts/[id]/page.tsx
import { db } from '@/app/actions/lib';
import { notFound } from 'next/navigation';

export default async function Page({
  params: { id },
}: {
  params: { id: string; }
}) {
  const post = await db.post.findUnique({
    where: {
      id,
    },
  });

  if (!post) {
    notFound();
  }

  return (
    <div>
      {post ? (
        <article>
          <h1>{post.title}</h1>
          <p>{post.body}</p>
        </article>
      ) : (
        <p>
          記事はありません
        </p>
      )}
    </div>
  );
}
```
