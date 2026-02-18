---
title: 【WordPress】内部リンクの末尾スラッシュが統一されていないと余計な301リダイレクトが発生する話
tags:
  - WordPress
  - URL
  - SEO
  - パフォーマンス
private: false
updated_at: '2026-02-18T10:46:45+09:00'
id: 03d4fad5f671965bca47
organization_url_name: null
slide: false
ignorePublish: false
---

当たり前のことかもしれないが、WordPressの案件に携わった際に改めて気づいたので、備忘録として残しておく。

---

## 起きていたこと

WordPressサイトの内部リンクを確認していたところ、一部のリンクで**末尾スラッシュが付いていない**ものがあった。

ブラウザの開発者ツールで確認すると、スラッシュなしのURLにアクセスするたびに **301リダイレクト** が発生していた。

```
GET /products/item01       → 301 Moved Permanently
GET /products/item01/      → 200 OK
```

見た目上はページが正常に表示されるので気づきにくいが、リンクをクリックするたびに余分なHTTPリクエストが1往復増えている状態だった。

---

## 原因

WordPressの **パーマリンク構造** に `/%postname%/` のように末尾スラッシュありの形式を設定している場合、スラッシュなしのURLにアクセスすると `redirect_canonical` という関数によって自動的に末尾スラッシュ付きのURLへ301リダイレクトされる。

つまり、パーマリンク構造で「末尾スラッシュあり」と定義しているのに、テンプレート側のリンクが「スラッシュなし」で書かれていると、毎回リダイレクトが挟まることになる。

---

## 具体例

```html
<!-- 末尾スラッシュが統一されていない例 -->
<a href="/products/item01">商品ページ</a>
<a href="/blog/my-first-post">ブログ記事</a>
<a href="/shop?cat=5">カテゴリ一覧</a>

<!-- パーマリンク構造に合わせて統一した例 -->
<a href="/products/item01/">商品ページ</a>
<a href="/blog/my-first-post/">ブログ記事</a>
<a href="/shop/?cat=5">カテゴリ一覧</a>
```

クエリパラメータ付きのURLも同様で、`/shop?cat=5` ではなく `/shop/?cat=5` のようにパス部分の末尾にスラッシュを入れる必要がある。

---

## 末尾スラッシュが不要なケース

すべてのURLに末尾スラッシュを付ければいいわけではなく、以下のようなケースではスラッシュを付けない（付けると逆に問題になる場合がある）。

- **静的ファイル**: CSS / JS / 画像ファイルへのパス
- **外部API・REST APIのエンドポイント**: 別のルーティング管轄のパス
- **外部サイトへのリンク**: 他サイトのURL規則に依存する

---

## まとめ

- WordPressのパーマリンク構造に末尾スラッシュがある場合、内部リンクもそれに合わせて統一する
- 統一されていないと、ユーザーには見えない301リダイレクトが毎回発生する
- 1回のリダイレクトの影響は小さいが、サイト全体で積み重なるとパフォーマンスやSEOに影響が出る可能性がある
- テンプレートやJSでURLをハードコードする際は、パーマリンク構造を意識してスラッシュの有無を確認する

WordPressに限った話ではなく、サイトのURL構造に合わせてリンクの末尾スラッシュを統一するのは基本的なことだが、テンプレートを手書きしていると意外と見落としがちなポイントだと思った。

---

## 参考

- [WordPress: redirect_canonical()](https://developer.wordpress.org/reference/functions/redirect_canonical/)
- [WordPress: パーマリンク設定](https://ja.wordpress.org/support/article/using-permalinks/)
