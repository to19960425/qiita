---
title: 【WooCommerce】注文詳細ページの「再注文」ボタンを非表示にする方法
tags:
  - PHP
  - WordPress
  - WooCommerce
private: false
updated_at: '2026-01-09T13:08:25+09:00'
id: b578ed46cf6afc97bee2
organization_url_name: null
slide: false
ignorePublish: false
---

## やりたいこと

WooCommerceの注文詳細ページ（マイアカウント → 注文履歴 → 注文詳細）に表示される「再注文」ボタンを非表示にしたい。
- 作業の備忘録になります。
- あくまでご参考になれば幸いです。

## 方法

`functions.php` に以下を追加：

```php
/**
 * 注文詳細ページから再注文ボタンを削除
 */
function remove_order_again_button()
{
    remove_action(
        'woocommerce_order_details_after_order_table',
        'woocommerce_order_again_button'
    );
}
add_action('woocommerce_init', 'remove_order_again_button');
```

## フック情報

| 項目     | 値                                            |
| -------- | --------------------------------------------- |
| フック名 | `woocommerce_order_details_after_order_table` |
| 関数名   | `woocommerce_order_again_button`              |
| 優先度   | 10（デフォルト）                              |
| 提供元   | WooCommerce本体                               |

## 参考

- [WooCommerce - Action and Filter Hook Reference](https://woocommerce.github.io/code-reference/hooks/hooks.html)
