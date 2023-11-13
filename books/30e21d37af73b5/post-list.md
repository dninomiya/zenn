---
title: 記事一覧画面の実装
---

```tsx:app/page.tsx
import { db } from '@/app/actions/lib';
import Link from 'next/link';

export default async function Page() {
  const posts = await db.post.findMany();

  return (
    <div>
      {posts.length > 0 ? (
        <ul>
          {posts.map((post) => {
            return (
              <li key={post.id}>
                <Link href={`/posts/${post.id}`}>{post.title}</Link>
              </li>
            );
          })}
        </ul>
      ) : (
        <p>
          記事はありません
        </p>
      )}
    </div>
  );
}
```
