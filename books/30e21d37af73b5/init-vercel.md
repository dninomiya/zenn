---
title: Vercelの準備
---

# GitHub リポジトリの作成

エディター左下の雲マークをクリックするとリポジトリを聞かれるので、名前を決めて作成します。private（非公開） or public（公開） はお好みで。

![](/images/github-create-repository.png)

# Vercel プロジェクトの作成

Vercel ダッシュボードからプロジェクトを作成します。作成時に先ほどのリポジトリをインポート対象に指定し、環境変数を設定する画面でプロジェクトの `.enc.local` を全体コピーしてペーストします。（一括で入力されます）

![](/images/vercel-env.png)

作成後、設定画面から Function Region を **Tokyo or Osaka, Japan** にします。

![](/images/vercel-region.png)

## デプロイについて

連携後は GitHub にプッシュすると自動的にデプロイされます。main ブランチにデプロイした場合本番 URL に反映され、それ以外はデモ用の URL に反映されます。

# Vercel Postgres の作成

Storages タブから Vercel Postgres を作成し、今回作成したプロジェクトと紐付けます。

![](/images/vercel-postgres-1.png)
![](/images/vercel-postgres-2.png)

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

終わったら `.env.development.local` は削除します。なお、この内容は Vercel プロジェクトに自動でセットされているので Vercel 管理画面から手動でセットする必要はありません。
