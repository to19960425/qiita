---
title: 【EC-CUBE4.3】プラグインを開発してみる
tags:
  - PHP
  - Symfony
  - doctrine
  - twig
  - EC-CUBE4
private: false
updated_at: '2025-04-21T15:56:13+09:00'
id: d17b6c7cadc24e1ea6a3
organization_url_name: null
slide: false
ignorePublish: false
---
## 動機
- EC-CUBEをカスタマイズする方法はある程度理解できたつもり。
- プラグインについて構造などまだよくわかっていない。
- 既存のプラグインをいじる必要が有りそうなのでこれを機会に実際に触ってみる。

## やりたいこと
- まずは、簡単なプラグインをイチから自分で開発してみる
- プラグイン内容
    - マイページ購入履歴から、直接受注問合せができるようにする

## 本記事でわかること
- プラグインの生成からインストール完了の確認まで
- 作成したプラグイン：https://github.com/to19960425/order-inquiry-plugin/

## 参考
- 公式ドキュメント：https://doc4.ec-cube.net/plugin_development

## 手順
### 1. プラグインの雛形を生成
- プラグイン名は、`OrderInquiryPlugin` とする。
```bash
bin/console eccube:plugin:generate
```
```bash
 name [EC-CUBE Sample Plugin]:
 > OrderInquiryPlugin

 code [Sample]:
 > OrderInquiry

 ver [1.0.0]:
 > # 空
 ```
 
### 2. プラグインをインストール
```
bin/console eccube:plugin:install --code OrderInquiry
```

### 3. プラグインを開発
- 管理画面を確認すると、プラグイン一覧の「ユーザー独自プラグイン」に追加される。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3850968/a7d9a19e-c238-4e0d-8755-2bcbbb17c690.png)
これで準備OKです。
