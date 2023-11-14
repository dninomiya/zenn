---
title: 投稿、編集画面の実装
---

![](/images/post-form-tree.png)

# フォームコンポーネントと関連アクションの実装

記事作成画面と編集画面で共有するフォームコンポーネントを作成します。

```tsx:app/components/post-form/action.tsx
'use server';

import { authGuard } from '@/app/actions/auth';
import { db, deleteImage, putImage } from '@/app/actions/lib';
import { Prisma } from '@prisma/client';
import { randomUUID } from 'crypto';
import { revalidatePath } from 'next/cache';
import { redirect } from 'next/navigation';
import { z } from 'zod';

const PostSchema = z.object({
  body: z.string().max(140),
});

export const createPost = async (formData: FormData) => {
  const authorId = authGuard();
  const id = randomUUID();
  const validatedData = PostSchema.parse({
    body: formData.get('body'),
  });
  const newData: Prisma.PostUncheckedCreateInput = {
    ...validatedData,
    id,
    authorId,
  };

  const thumbnailDataURL = formData.get('thumbnail') as string;

  if (thumbnailDataURL) {
    newData.thumbnailURL = await putImage(
      thumbnailDataURL,
      `posts/${id}/thumbnail.png`
    );
  }

  await db.post.create({
    data: newData,
  });

  revalidatePath('/');
  redirect('/');
};

export const updatePost = async (id: string, formData: FormData) => {
  const authorId = authGuard();
  const validatedData = PostSchema.parse({
    body: formData.get('body'),
  });
  const newData: Prisma.PostUncheckedUpdateInput = {
    body: validatedData.body,
  };

  const thumbnailDataURL = formData.get('thumbnail') as string;
  const thumbnailAction = formData.get('thumbnail-action') as string;
  const imagePath = `posts/${id}/thumbnail.png`;

  if (thumbnailDataURL && thumbnailAction === 'save') {
    newData.thumbnailURL = await putImage(thumbnailDataURL, imagePath);
  } else if (thumbnailAction === 'delete') {
    newData.thumbnailURL = null;
    await deleteImage(imagePath);
  }

  await db.post.update({
    where: {
      id,
      authorId,
    },
    data: newData,
  });

  revalidatePath('/');
};

export const deletePost = async (id: string, imageURL?: string | null) => {
  const uid = authGuard();

  await db.post.delete({
    where: {
      id,
      authorId: uid,
    },
  });

  if (imageURL) {
    deleteImage(imageURL.replace(process.env.IMAGE_HOST_URL as string, ''));
  }

  revalidatePath('/');
  redirect('/');
};

export const getOwnPost = (editId: string) => {
  const uid = authGuard();

  return db.post.findUnique({
    where: {
      id,
      authorId: uid,
    },
  });
}
```

```tsx:app/components/post-form/index.tsx
import {
  createPost,
  deletePost,
  getOwnPost,
  updatePost,
} from './action';
import ImageCropper from '@/app/components/image-cropper';
import SubmitButton from '@/app/components/submit-button';
import { Label } from '@/components/ui/label';
import { Textarea } from '@/components/ui/textarea';

export default async function PostForm({ editId }: { editId?: string }) {
  const oldPost = editId ? await getOwnPost(editId) : null;

  const defaultValue = oldPost
    ? {
        title: oldPost.title,
        body: oldPost.body,
      }
    : {
        title: '',
        body: '',
      };

  return (
    <div>
      <form action={editId ? updatePost.bind(null, editId) : createPost}>
        <div>
          {oldPost?.thumbnailURL && (
            <div>
              <input type="checkbox" id="thumbnail-action" name="thumbnail-action" value="delete" className="peer" />
              <label htmlFor="thumbnail-action">
                削除
              </label>
              <img className="peer-checked:hidden" src={oldPost.thumbnailURL} alt="" />
            </div>
          )}
          <input type="file" name="thumbnail" accept="images/png,images/jpeg" />
        </div>

        <label htmlFor="body">本文*</label>
          <textarea
            maxLength={140}
            name="body"
            placeholder=""
            defaultValue={defaultValue.body}
            id="body"
            required
          />

          <button>{editId ? '更新' : '作成'}</button>****
      </form>

      {editId && oldPost && (
        <form
          action={deletePost.bind(null, editId, oldPost.thumbnailURL)}
        >
          <button>記事を削除</button>
        </form>
      )}
    </div>
  );
}
```

# 投稿画面の実装

```tsx:app/create/page.tsx
import PostForm from '@/app/components/post-form';

export default function Page() {
  return <PostForm />;
}
```

# 編集画面の実装

```tsx:app/posts/[id]/edit/page.tsx
import { getPost } from '@/app/actions/post';
import PostForm from '@/app/components/post-form';
import { notFound } from 'next/navigation';

export default async function Page({
  params: { id },
}: {
  params: {
    id: string;
  };
}) {
  const authorId = authGuard();

  const post = await db.post.findUnique({
    where: {
      id,
      authorId,
    },
  });;

  if (!post) {
    notFound();
  }

  return <PostForm editId={id} />;
}
```
