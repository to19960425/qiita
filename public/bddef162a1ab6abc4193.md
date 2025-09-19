---
title: 【EC-CUBE4.3】受注一覧を特定のステータスで絞り込んだ状態で表示する方法
tags:
  - PHP
  - EC-CUBE4
private: false
updated_at: '2025-06-12T12:27:50+09:00'
id: bddef162a1ab6abc4193
organization_url_name: null
slide: false
ignorePublish: false
---
## はじめに
EC-CUBE4.3の管理画面で、受注一覧ページを開いたときに「新規受付（order\_status\_id=1）」の注文だけを初期表示させたい、という要件がありました。

## 要件概要

* 管理画面の「注文一覧」メニューをクリックした際に、URL末尾に `?order_status_id=1` を自動で付加したい
* コントローラー側の処理を変更せず、ナビゲーション設定だけで実現したい

---

## 方法

### 1. ナビゲーション定義ファイルの場所

EC-CUBEの管理画面のナビゲーションは、以下のようなYAMLファイルで定義されています。

```
app/config/eccube/packages/eccube_nav.yaml
```

この中で、受注一覧（注文一覧）に対応する設定は通常このようになっています：

```yaml
order_master:
    name: admin.order.order_list
    url: admin_order
```

### 2. クエリパラメータ付きに書き換え

クエリパラメータを追加します。

```yaml
order_master:
    name: admin.order.order_list
    url: admin_order
    param: { order_status_id: 1 }
```

### 3. コントローラーでのパラメータ受け取り

`app/Customize/Controller/Admin/Order/OrderController.php` の `index()` メソッドでは、以下のように `order_status_id` パラメータを受け取り、検索条件に反映しています：

```php
$page_no = 1;
$viewData = [];

if ($statusId = (int) $request->get('order_status_id')) {
    $viewData = ['status' => [$statusId]];
}

$searchData = FormUtil::submitAndGetData($searchForm, $viewData);
```

このため、URLに `?order_status_id=1` が含まれていれば、自動的に新規受付（ステータスID=1）の注文がフィルタされた状態で一覧が表示されます。

---

## まとめ

管理画面ナビゲーションのYAML定義にパラメータを追加するだけで、注文一覧ページの初期状態を「新規受付のみ」に変更できます。
簡単かつ保守性も高いため、要件にマッチする場合はこの方法がおすすめです。

※ こちらの記事はChatGPTにまとめてもらいました。
