---
title: Cloudflare R2 の準備
---

# Cloudflare R2 の準備

[Cloudflare](http://cloudflare.com/) でアカウントを作成し、 Cloudflare R2 のバケットを作成します。

![](/images/cloudflare-create.png)

次に、バケット設定画面からサブドメインを有効にします。

![](/images/cloudflare-enable-subdomain.png)

## 環境変数の設定

バケットの画面より

```ini:.env.local
CLOUDFLARE_ENDPOINT=xxx
CLOUDFLARE_ACCESS_KEY_ID=xxx
CLOUDFLARE_ACCESS_KEY=xxx

# サブドメインの公開 URL
IMAGE_HOST_URL=xxx
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

export const putImage = async (dataUrl: string, pathname: string) => {
  const file = dataURLtoBuffer(dataUrl);

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
