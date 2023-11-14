---
title: Cloudflare R2 の準備
---

# Cloudflare R2 の準備

[Cloudflare](http://cloudflare.com/) でアカウントを作成し、 Cloudflare R2 のバケットを作成します。

![](/images/cloudflare-create.png)

次に、バケット設定画面からサブドメインを有効にします。

![](/images/cloudflare-enable-subdomain.png)

## 環境変数の設定

管理画面よりトークンを作成します。

![](/images/cloudflare-create-token-1.png)

API トークンを以下の設定で作成します。Specify buckets は自分のバケットを選択してください。

![](/images/cloudflare-create-token-2.png)

最後の画面で必要な情報をコピーします。この情報は 2 度と表示されないので必ずコピーしてください。同時に、この情報が人に知れるとバケットにアクセスされる可能性があるので注意してください。紛失したら同じフローでトークンを再作成できます。

![](/images/cloudflare-create-token-3.png)

コピーした情報をもとに環境変数を設定します。

```ini:.env.local
CLOUDFLARE_ACCESS_KEY_ID=1の内容
CLOUDFLARE_ACCESS_KEY=2の内容
CLOUDFLARE_ENDPOINT=3の内容

# サブドメインの公開 URL
IMAGE_HOST_URL=冒頭で有効にしたサブドメインのURL
```

## 画像操作関数を作成

```bash
bun add @aws-sdk/client-s3
```

```ts:lib/storage.ts
import {
  S3Client,
  PutObjectCommand,
  PutObjectCommandInput,
  DeleteObjectCommand,
  DeleteObjectCommandInput,
} from '@aws-sdk/client-s3';

const client = new S3Client({
  region: 'auto',
  endpoint: process.env.CLOUDFLARE_ENDPOINT as string,
  credentials: {
    accessKeyId: process.env.CLOUDFLARE_ACCESS_KEY_ID as string,
    secretAccessKey: process.env.CLOUDFLARE_ACCESS_KEY as string,
  },
});

export const putImage = async (file: File, pathname: string) => {
  const uploadParams: PutObjectCommandInput = {
    Bucket: 'next-demo',
    Key: pathname,
    Body: file,
    ContentType: 'image/png',
    ACL: 'public-read',
  };

  const command = new PutObjectCommand(uploadParams);
  await client.send(command);

  return `${process.env.IMAGE_HOST_URL}/${pathname}`;
};

export const deleteImage = async (pathname: string) => {
  const uploadParams: DeleteObjectCommandInput = {
    Bucket: 'next-demo',
    Key: pathname,
  };

  const command = new DeleteObjectCommand(uploadParams);
  return client.send(command);
};
```
