---
title: 'オンボーディングフロー'
---

はじめて利用するユーザーがアカウント等を作成し、利用可能な状態になるためのフローをオンボーディングフロートいいます。

# リダイレクト

現状はログインしているかしていないかを軸にリダイレクトが自動でなされますが、これをオンボーディングの完了ステータスで判断するよう変更します。

```ts:middleware.ts
export default authMiddleware({
  publicRoutes: ['/'],
  async afterAuth(auth, req) {
    if (!auth.userId && auth.isPublicRoute) {
      return;
    }

    // 未ログインかつ非公開ルートへのアクセスはログイン画面へリダイレクト
    if (!auth.userId && !auth.isPublicRoute) {
      return redirectToSignIn({ returnBackUrl: req.url });
    }

    // セッションにオンボーディングの完了ステータスがあるか確認
    let onboarded = auth.sessionClaims?.onboarded;

    if (!onboarded) {
      // セッションになければClerkユーザー情報からステータスを取得
      const user = await clerkClient.users.getUser(auth.userId!);
      onboarded = user.publicMetadata.onboarded;
    }

    // オンボーディング前ならオンボーディングページへリダイレクト
    if (!onboarded && req.nextUrl.pathname !== '/onboarding') {
      const orgSelection = new URL('/onboarding', req.url);
      return NextResponse.redirect(orgSelection);
    }

    // オンボーディング済みでオンボーディングページへアクセスしたらトップページへリダイレクト
    if (onboarded && req.nextUrl.pathname === '/onboarding') {
      const orgSelection = new URL('/', req.url);
      return NextResponse.redirect(orgSelection);
    }
  },
});
```

sessionClaims を受け取るためにアプリのダッシュボードより `onboarded` というキーのデータをセッションに含めるよう設定します。

![](/images/clerk-customize-session-token.png)

## リダイレクトはどう制御するのが良いのか？

middleware で一元管理的に制御するアプローチをお勧めします。各ページコンポーネントで状態を取得してリダイレクトさせることもできますが、その場合全体の関係性が把握しづらく、意図せず無限リダイレクトを発生させる可能性があります。

現時点で　 Prisma が Edge に対応していないので DB に基づいたリダイレクトはできません。そのため Clerk のメタデータを介してリダイレクトを制御しています。

# オンボーディングページの作成

```tsx:app/onboarding/page.tsx
export default async function UserForm() {
  return (
    <form action={createUser}>
      <label htmlFor="name">名前</label>
      <input type="text" id="name" defaultValue="" required name="name" />
      <button>作成</button>
    </form>
  );
}
```

```ts:app/onboarding/action.ts
'use server';

import { db } from '@/app/actions/lib';
import { Prisma } from '@prisma/client';
import { revalidatePath } from 'next/cache';
import { redirect } from 'next/navigation';
import { z } from 'zod';

const UserSchema = z.object({
  name: z.string().max(240),
});

export const createUser = async () => {
  'use server';

  const id = authGuard();
  const validatedData = UserSchema.parse({
    name: formData.get('name'),
  });

  const data: Prisma.UserUncheckedCreateInput = {
    name: validatedData.name,
    id,
  };

  // DBにユーザーを作成
  await db.user.create({
    data,
  });

  // Clerkのユーザーメタデータにオンボーディング完了ステータスをセット
  await clerkClient.users.updateUserMetadata(id, {
    publicMetadata: {
      onboarded: true,
    },
  });

  // キャッシュをクリア
  revalidatePath('/');

  // トップページへリダイレクト
  redirect('/');
};
```
