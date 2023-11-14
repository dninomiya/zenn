---
title: ファビコン
---

FaviconGenerator で作成し、出力ファイルから favicon.ico を取得して `/app` に設置して完了です。

https://realfavicongenerator.net

## 元画像をアップロード（正方形でシンプルな画像）

![](/images/favicon-generator-1.png)

## ページ下部より生成開始

![](/images/favicon-generator-2.png)

## 出力ファイルをダウンロード

![](/images/favicon-generator-3.png)

## favicon.ico を取得し、`/app` に設置

![](/images/favicon-generator-4.png)

## 備考

- Favicon はキャッシュが残って反映されなかったりしますが気にしないで OK です。そのうち反映されます。
- SVG ファビコンが理想ですが Safari が非対応なので今回のアプローチを取ります。条件分岐して SVG ファビコンを反映させることも可能です。
- 無理やり SVG ファビコンを Safari に対応させるハックがあるのですが、他の共有系 SNS で正しく表示されないケースなどもあるため ico が無難です。
