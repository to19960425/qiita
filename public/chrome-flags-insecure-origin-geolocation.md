---
title: 【Chrome】HTTP環境で位置情報などのセキュアな機能を一時的に使えるようにする方法
tags:
  - Chrome
  - HTTP
  - テスト
  - Geolocation
private: false
updated_at: '2026-02-04T10:57:08+09:00'
id: 96aee84f49ebb6eee93d
organization_url_name: null
slide: false
ignorePublish: false
---

備忘録として記事にしました。

---

## 背景

テスト環境（HTTP）で位置情報を使った処理の動作確認がしたかった。

ローカルに HTTPS 環境を構築する方法もあるが、そのためだけに環境を作るのは手間だったので、もっと手軽な方法を探した。

---

## 解決策：Chrome の flags 設定を使う

Chrome のアドレスバーに以下を入力してアクセスする。

```
chrome://flags/#unsafely-treat-insecure-origin-as-secure
```

表示された設定欄に対象のオリジン（例：`http://test-site.jp`）を入力して有効化すれば、一時的に HTTP 環境でも位置情報機能等のセキュアコンテキストが必要な API を使用できるようになる。

---

## 手順

1. Chrome のアドレスバーに `chrome://flags/#unsafely-treat-insecure-origin-as-secure` を入力
2. 「Insecure origins treated as secure」の入力欄に対象の URL を入力（例：`http://test-site.jp`）
3. プルダウンを「Enabled」に変更
4. 「Relaunch」ボタンで Chrome を再起動

---

## 注意事項

- **Google Chrome 限定**の機能です
- あくまで開発・テスト用途の一時的な設定であり、常用は推奨されません
- 本番環境では必ず HTTPS を使用してください
