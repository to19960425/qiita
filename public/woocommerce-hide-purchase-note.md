---
title: 【WooCommerce】注文詳細ページの「購入メモ」を非表示にする方法
tags:
  - PHP
  - WordPress
  - WooCommerce
private: false
updated_at: ''
id: null
organization_url_name: null
slide: false
ignorePublish: false
---

## やりたいこと

WooCommerceの注文詳細ページ（マイアカウント → 注文履歴 → 注文詳細）に表示される「購入メモ（Purchase Note）」を非表示にしたい。
- 作業の備忘録になります。
- あくまでご参考になれば幸いです。

## 購入メモとは

WooCommerceの標準機能で、商品購入後に顧客に表示されるメモです。

**設定場所**: 商品編集画面 → 商品データ → 高度（Advanced） → 購入メモ

## 方法

`functions.php` に以下を追加：

```php
/**
 * 注文詳細ページの購入メモを非表示にする
 *
 * @param string $value 購入メモの内容
 * @param WC_Product $product 商品オブジェクト
 * @return string 空文字列（非表示）または元の値
 */
function hide_purchase_note_on_view_order($value, $product)
{
    if (is_wc_endpoint_url('view-order')) {
        return '';
    }
    return $value;
}
add_filter('woocommerce_product_get_purchase_note', 'hide_purchase_note_on_view_order', 10, 2);
```

## フック情報

| 項目           | 値                                        |
| -------------- | ----------------------------------------- |
| フィルター名   | `woocommerce_product_get_purchase_note`   |
| 出力テンプレート | `templates/order/order-details-item.php` |
| CSSクラス      | `product-purchase-note`                   |
| データ保存先   | `_purchase_note` メタキー                 |
| 提供元         | WooCommerce本体                           |

## 別の方法

### CSSで非表示

```css
.product-purchase-note {
    display: none;
}
```

※HTMLは出力されるが表示されない

### 全ページで非表示

```php
add_filter('woocommerce_product_get_purchase_note', '__return_empty_string');
```

※注文完了ページ（thankyou）やメールでも非表示になる

## 注意事項

- 今回の実装は注文詳細ページ（view-order）でのみ非表示にします
- 注文完了ページ（thankyou）やメールでは引き続き表示されます

## 参考

- [WooCommerce - Action and Filter Hook Reference](https://woocommerce.github.io/code-reference/hooks/hooks.html)
