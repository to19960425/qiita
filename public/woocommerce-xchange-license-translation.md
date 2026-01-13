---
title: 【WooCommerce】xchangemarketのライセンス情報を日本語化する方法
tags:
  - PHP
  - WordPress
  - WooCommerce
private: false
updated_at: '2026-01-13T09:34:45+09:00'
id: 5eb51a5c1a373da1a5dd
organization_url_name: null
slide: false
ignorePublish: false
---

## やりたいこと

WooCommerceの注文詳細ページ（マイアカウント → 注文履歴 → 注文詳細）に表示される「Product Licensing Information」を日本語化したい。
- 作業の備忘録になります。
- あくまでご参考になれば幸いです。

## 現状

ダウンロード商品購入後、以下のような英語のライセンス情報が表示される：

```
Product Licensing Information
Product:    HOLLYWOOD CHOIRS - diamond-download
Serial Number:    AOQJ KBIU JOZD DJIP MTVT
Download:    https://www.eastwestsounds.com/register
Support Contact:    jmedina@eastwestsounds.com
```

## 方法

`functions.php` に以下を追加：

```php
/**
 * xchangemarketのライセンス情報を日本語化
 *
 * @return void
 */
function translate_xchange_license_info()
{
    remove_action('woocommerce_order_details_after_order_table', 'xchange_info_display', 10);
    add_action('woocommerce_order_details_after_order_table', 'xchange_info_display_ja', 10, 1);
}
add_action('init', 'translate_xchange_license_info');

/**
 * ライセンス情報を日本語で表示
 *
 * @param WC_Order $order 注文オブジェクト
 * @return void
 */
function xchange_info_display_ja($order)
{
    $xchangeinfo = get_post_meta($order->get_id(), 'xchange_license_info', true);

    if (empty($xchangeinfo) || substr($xchangeinfo, 0, 3) === 'tx:') {
        return;
    }

    $translations = array(
        'Product Licensing Information' => '製品ライセンス情報',
        '<TH>Product:</TH>'              => '<TH>製品：</TH>',
        '<TD>Product:</TD>'              => '<TD>製品：</TD>',
        '<TD>Serial Number:</TD>'        => '<TD>シリアル番号：</TD>',
        '<TD>Download:</TD>'             => '<TD>ダウンロード：</TD>',
        '<TD>Support Contact:</TD>'      => '<TD>サポート連絡先：</TD>',
    );

    $xchangeinfo = str_replace(array_keys($translations), array_values($translations), $xchangeinfo);

    echo "<p>" . $xchangeinfo . "\n";
}
```

## フック情報

| 項目         | 値                                            |
| ------------ | --------------------------------------------- |
| フック名     | `woocommerce_order_details_after_order_table` |
| 元の関数名   | `xchange_info_display`                        |
| 優先度       | 10                                            |
| メタキー     | `xchange_license_info`                        |
| 提供元       | xchangemarketプラグイン                       |

## 翻訳マッピング

| 英語                            | 日本語           |
| ------------------------------- | ---------------- |
| Product Licensing Information   | 製品ライセンス情報 |
| Product:                        | 製品：           |
| Serial Number:                  | シリアル番号：   |
| Download:                       | ダウンロード：   |
| Support Contact:                | サポート連絡先： |

## 実装方式のポイント

プラグインを直接編集せず、テーマ側で対応：

1. `remove_action()`で元の表示関数を解除
2. `add_action()`で翻訳版の関数を同じフックに登録
3. カスタムフィールドから取得したHTMLを`str_replace()`で翻訳

**メリット**:
- プラグインのアップデートに影響されない
- テーマ側で完結するため管理が容易

## 注意事項

- xchangemarketプラグインのバージョンアップでHTMLの構造やラベルが変更される可能性あり
- その場合は`$translations`配列を更新してください
- メールの日本語化は別途対応が必要です

## 参考

- [WooCommerce - Action and Filter Hook Reference](https://woocommerce.github.io/code-reference/hooks/hooks.html)
