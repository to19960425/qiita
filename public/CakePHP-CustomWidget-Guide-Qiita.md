---
title: 【CakePHP 4.x】カスタムウィジェットの作り方
tags:
  - PHP
  - CakePHP
  - form
  - widget
private: false
updated_at: '2026-01-07T14:58:08+09:00'
id: 587d2ff07c9b4c9a34b6
organization_url_name: null
slide: false
ignorePublish: false
---

# CakePHP 4.x カスタムウィジェット作成ガイド

フォームの入力要素をカスタマイズする**ウィジェット**の作成方法を、パスワード表示切替ウィジェットを例に解説します。

## ファイル構成

```
src/View/
├── AppView.php                    # ウィジェット登録
└── Widget/
    └── PasswordToggleWidget.php   # ウィジェット本体

config/
└── app_form.php                   # HTMLテンプレート定義
```

## 実装手順

### 1. ウィジェットクラス作成

```php
// src/View/Widget/PasswordToggleWidget.php
<?php
declare(strict_types=1);

namespace App\View\Widget;

use Cake\View\Form\ContextInterface;
use Cake\View\StringTemplate;
use Cake\View\Widget\WidgetInterface;

class PasswordToggleWidget implements WidgetInterface
{
    protected StringTemplate $_templates;

    public function __construct(StringTemplate $templates)
    {
        $this->_templates = $templates;
    }

    public function render(array $data, ContextInterface $context): string
    {
        $data += [
            'name' => '',
            'val' => null,
            'escape' => true,
            'templateVars' => [],
        ];

        unset($data['type']);

        // CakePHP内部の 'val' → HTML の 'value' に変換
        if (($value = $data['val']) !== null && $value !== '') {
            $data['value'] = $value;
        }
        unset($data['val']);

        return $this->_templates->format('passwordToggle', [
            'name' => $data['name'],
            'type' => 'password',
            'templateVars' => $data['templateVars'],
            'attrs' => $this->_templates->formatAttributes(
                $data,
                ['name', 'type', 'templateVars']
            ),
        ]);
    }

    public function secureFields(array $data): array
    {
        return isset($data['name']) && $data['name'] !== ''
            ? [$data['name']]
            : [];
    }
}
```

### 2. AppView.php で登録

```php
// src/View/AppView.php
public function initialize(): void
{
    $this->loadHelper('Form', [
        'templates' => 'app_form',
        'widgets' => [
            'passwordToggle' => ['App\\View\\Widget\\PasswordToggleWidget'],
        ],
    ]);
}
```

### 3. HTMLテンプレート定義

```php
// config/app_form.php
<?php
return [
    'passwordToggle' => '<div class="password-wrap">' .
        '<input type="{{type}}" name="{{name}}"{{attrs}}/>' .
        '<button type="button" class="js-password-toggle">' .
        '<i class="fa fa-eye"></i>' .
        '</button>' .
        '</div>',
];
```

## 使用方法

```php
echo $this->Form->control('password', [
    'type' => 'passwordToggle',
    'label' => 'パスワード',
    'class' => 'form-control',
    'required' => true,
]);
```

## ポイント整理

| 項目 | 説明 |
|------|------|
| `val` → `value` | CakePHP内部キーをHTML属性に変換 |
| `formatAttributes()` | 配列をHTML属性文字列に変換（第2引数で除外指定可） |
| `secureFields()` | CSRF保護対象フィールドを返す（必須メソッド） |
| テンプレート | `app_form.php`で定義、`format()`の第1引数名で紐づく |

## 参考

- [CakePHP 4.x - Creating Custom Widgets](https://book.cakephp.org/4/en/views/helpers/form.html#creating-custom-widgets)
