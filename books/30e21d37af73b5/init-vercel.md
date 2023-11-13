---
title: Vercelの準備
---

# Vercel の準備

Vercel プロジェクトを作成の上、設定画面から Function Region を **Tokyo or Osaka, Japan** にします。同時に Storages で Vercel Postgres を作成し、今回作成したプロジェクトと紐付けます。（UI を見ればわかりそうなので割愛します）

![](/images/vercel-region.png)

## DB アクセスに必要な環境変数の取得

プロジェクトディレクトリで以下を実行

```bash
# Vercel CLI がない場合
bun add -g vercel@latest

vercel env pull .env.development.local
```

これにより `.env.development.local` が作成されるので、その中から以下をコピーして `.env.local` に貼り付けます。

```ini:.env.local
POSTGRES_URL=
POSTGRES_PRISMA_URL=
POSTGRES_URL_NON_POOLING=
POSTGRES_USER=
POSTGRES_HOST=
POSTGRES_PASSWORD=
POSTGRES_DATABASE=
```

終わったら `.env.development.local` は削除します。
