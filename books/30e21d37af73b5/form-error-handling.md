---
title: '付録: フォームのエラーハンドリング'
---

本編ではバリデーションを行いつつもエラーハンドリングを割愛しました。実際のプロダクトでは「文字数が多すぎます」などのエラーをフィードバックする必要があります。また、送信後は Toast（ポップアップ通知）などで成功のフィードバックを行うことも多いでしょう。

# Toast のインストールと初期化

まずは shadcn/ui から Toast をインストールします。

```bash
bunx shadcn-ui@latest add toast
```

アプリケーション全体で Toast を使えるようにします。

```diff tsx:app/layout.tsx
+ import { Toaster } from "@/components/ui/toaster"
...
+  <Toaster />
 </body>
```

# Zod でエラーメッセージをセットする

まずは Zod のスキーマを調整してエラーメッセージを調整します。なお、本編のコードとは無関係です。

```tsx:action.ts
'use server';

import { authGuard } from '@/app/actions/auth';
import { db } from '@/app/actions/lib';
import { Prisma } from '@prisma/client';
import { randomUUID } from 'crypto';
import { z } from 'zod';

// 返却ステータスの型を定義
export type FormState =
  | {
      status: 'success';
      message: string; // 成功時のメッセージ
    }
  | {
      status: 'error';
      fieldErrors: {
        body?: string[] | undefined;
      };
    }
  | {
      status: 'idle';
    };

// エラーメッセージのセット
const PostSchema = z.object({
  body: z
    .string()
    .min(1, {
      message: '本文を入力してください',
    })
    .max(140, {
      message: '本文は140文字以内で入力してください',
    }),
});

export const createPost = async (formData: FormData): Promise<FormState> => {
  const authorId = authGuard();
  const id = randomUUID();

  // safeParse に変更
  const validatedData = PostSchema.safeParse({
    body: formData.get('body'),
  });

  if (!validatedData.success) {
    // エラー時にエラーステータス＆エラーメッセージを返却
    return {
      status: 'error',
      fieldErrors: validatedData.error.flatten().fieldErrors,
    };
  }

  const newData: Prisma.PostUncheckedCreateInput = {
    ...validatedData.data,
    id,
    authorId,
  };

  await db.post.create({
    data: newData,
  });

  return {
    status: 'success',
    messages: '保存しました'
  };
};
```

# フォームでエラーと Toast を表示する

送信アクションを `useFormState` に変更します。そのため フォームコンポーネント（or ページ）をクライアントコンポーネントにする必要があります。

```tsx
'use client';

import { useFormState } from '@/hooks/useFormState';
import { FormState } from '@/action';

export const Form = () => {
  const [formState, handleSubmit] = useFormState<FormState>(
    (_: FormState, data: FormData) => {
      return createPost(data).then((state) => {
        if (state.status === 'success') {
          // 成功時に Toast を表示
          toast(state.messages);
        } else if (state.status === 'error') {
          // エラー時に Toast を表示
          toast('入力内容を確認してください');
        }

        return state;
      });
    }
  );

  return (
    <form onSubmit={handleSubmit}>
      <textarea name="body" />
      <!-- フォームの下にエラーメッセージを表示 -->
      {formState.status === 'error' &&
        formState.fieldErrors.body?.map((error) => (
          <p key={error} className="text-red-500">
            {error}
          </p>
        ))}
      <button type="submit">送信</button>
    </form>
  );
};
```
