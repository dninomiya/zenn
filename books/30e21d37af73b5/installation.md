---
title: '環境構築'
---

# Bun のインストール

npm/pnpm/yarn より早いので [Bun](https://bun.sh) をパッケージマネージャとしてインストールします。

```bash
npm i -g bun
```

# Next.js プロジェクトの作成

```bash
bunx create-next-app@latest
```

質問の回答は以下のようにしてください。最初の初期化以降は前回の値が初期値として使われるので Enter 連打で OK です。

```bash
What is your project named? my-app
Would you like to use TypeScript? Yes
Would you like to use ESLint? Yes
Would you like to use Tailwind CSS? Yes
Would you like to use `src/` directory? No
Would you like to use App Router? (recommended) Yes
Would you like to customize the default import alias (@/*)? Yes
What import alias would you like configured? @/*
```

これにより [Tailwind CSS](https://tailwindcss.com) も初期化されます。

# shadcn/ui の追加

UI の整備が楽になるので [shadcn/ui](https://ui.shadcn.com/docs/installation/next) を追加します。(シャッドシーエヌユーアイと発音します)

- ライブラリではないのでフルカスタム可能
- ライブラリではないので依存せず、ロックされない
- コンポーネント単位でスニペットが丸ごとプロジェクトディレクトリにコピペされる形になる
- テーマカラーなのが CSS 変数化されているので、UI 全体で色や角丸などを一元管理できる
- ダークモード対応
- [Radix UI](https://www.radix-ui.com) & [Taiwlind CSS](https://tailwindcss.com) ベース

```bash
bunx shadcn-ui@latest init
```

質問は以下のように回答。基本デフォルトで良いのですが、 `tailwind.config.js` は `tailwind.config.ts` に変更してください。こうしないと `tailwind.config.js` が重複して作成されます。[近日修正入るようです。](https://github.com/shadcn-ui/ui/pull/1247)

```bash
Would you like to use TypeScript (recommended)? yes
Which style would you like to use? › Default
Which color would you like to use as base color? › Slate
Where is your global CSS file? › › app/globals.css
Do you want to use CSS variables for colors? › yes
Where is your tailwind.config.js located? › tailwind.config.ts
Configure the import alias for components: › @/components
Configure the import alias for utils: › @/lib/utils
Are you using React Server Components? › yes
```

以後[コンポーネント一覧](https://ui.shadcn.com/docs/components/accordion)から欲しいコンポーネントを選んでインストールします。たとえば Button を使いたい場合は

```bash
shadcn-ui@latest add button
```

として、以下のように使います。

```tsx
import { Button } from '@/components/ui/button';

export default function Home() {
  return (
    <div>
      <Button>Click me</Button>
    </div>
  );
}
```

細かい使い方は各コンポーネントのページを参照。[拡張機能](https://marketplace.visualstudio.com/items?itemName=SuhelMakkad.shadcn-ui)を使うと直感的にインストールや Doc の参照ができるのでおすすめです。
