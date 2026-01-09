---
title: 【WooCommerce】Freshdeskサポートフォームを注文詳細ページから非表示にする方法
tags:
  - PHP
  - WordPress
  - WooCommerce
  - Freshdesk
private: false
updated_at: '2026-01-09T13:08:26+09:00'
id: 63ca4c970d0e0c8fbce8
organization_url_name: null
slide: false
ignorePublish: false
---

## やりたいこと

WooCommerce Freshdeskプラグインが注文詳細ページ（マイアカウント → 注文履歴 → 注文詳細）に自動追加する「サポート問い合わせフォーム」を非表示にしたい。
- 作業の備忘録になります。
- あくまでご参考になれば幸いです。

## 方法

`functions.php` に以下を追加：

```php
/**
 * 注文詳細ページからFreshdeskサポートフォームを削除
 */
function remove_freshdesk_ticket_form()
{
    $integrations = WC()->integrations->get_integrations();

    if (isset($integrations['freshdesk']) && is_object($integrations['freshdesk'])) {
        $freshdesk = $integrations['freshdesk'];
        remove_action(
            'woocommerce_view_order',
            array($freshdesk, 'view_order_create_ticket'),
            40
        );
    }
}
add_action('woocommerce_init', 'remove_freshdesk_ticket_form');
```

## フック情報

| 項目       | 値                              |
| ---------- | ------------------------------- |
| フック名   | `woocommerce_view_order`        |
| メソッド名 | `view_order_create_ticket`      |
| 優先度     | 40                              |
| 提供元     | WooCommerce Freshdeskプラグイン |

- 将来的にプラグインのバージョンアップでフック構成が変更される可能性有

## 参考

- [WooCommerce Freshdesk Integration](https://woocommerce.com/products/woocommerce-freshdesk/)
