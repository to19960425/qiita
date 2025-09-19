---
title: 【EC-CUBE4.3】プラグインを開発してみる その６
tags:
  - PHP
  - Symfony
  - doctrine
  - twig
  - EC-CUBE4
private: false
updated_at: '2025-04-21T15:43:47+09:00'
id: 99bffb99d4fa2e77ffb3
organization_url_name: null
slide: false
ignorePublish: false
---
## 本記事で分かること
- プラグインでのメール文章のカスタマイズ

## 手順
### 1. Event クラスにイベント処理を追加
- 下記が、問い合わせメールの Twig テンプレート
```php
// src/Eccube/Resource/template/default/Mail/contact_mail.twig
{% autoescape 'safe_textmail' %}
※本メールは自動配信メールです。

┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
　※本メールは、
　{{ BaseInfo.shop_name }}よりお問い合わせされた方に
　お送りしています。
　もしお心当たりが無い場合は、
　その旨 {# 問い合わせ受付メール #}{{ BaseInfo.email02 }} まで
　ご連絡いただければ幸いです。
┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛

{{ data.name01 }} {{ data.name02 }} 様

以下のお問い合わせを受付致しました。
確認次第ご連絡いたしますので、少々お待ちください。

お名前：{{ data.name01 }} {{ data.name02 }}{% if data.kana01 or data.kana02 %} ({{ data.kana01 }} {{ data.kana02 }}){% endif %} 様
郵便番号：{% if data.postal_code %}〒{{ data.postal_code }}{% endif %}

住所：{% if data.pref.name is defined %} {{ data.pref.name }}{% endif %}{{ data.addr01 }}{{ data.addr02 }}
電話番号：{{ data.phone_number }}
メールアドレス：{{ data.email }}
お問い合わせ内容：

{{ data.contents }}
{% endautoescape  %}
```

- イベントリスナーで、`data.contents` の先頭に 注文番号 を追記する

```php
namespace Plugin\OrderInquiry;

use Eccube\Event\TemplateEvent;
use Symfony\Component\EventDispatcher\EventSubscriberInterface;
use Symfony\Contracts\Translation\TranslatorInterface;

class Event implements EventSubscriberInterface
{

    /**
     * @var TranslatorInterface
     */
    private $translator;

    /**
     * Event constructor.
     *
     * @param TranslatorInterface $translator
     */
    public function __construct(TranslatorInterface $translator)
    {
        $this->translator = $translator;
    }

    public static function getSubscribedEvents(): array
    {
        return [
            'Mypage/history.twig' => 'onHistoryDetail',
            'Contact/index.twig' => 'onContactIndex',
            'Contact/confirm.twig' => 'onContactConfirm',
            'Mail/contact_mail.twig' => 'onContactMail', //追加
            'Mail/contact_mail.html.twig' => 'onContactMail', //追加
        ];
    }

    public function onHistoryDetail(TemplateEvent $event): void
    {
        $twig = '@OrderInquiry/default/Mypage/history.twig';
        $event->addSnippet($twig);
    }

    public function onContactIndex(TemplateEvent $event): void
    {
        $twig = '@OrderInquiry/default/Contact/index.twig';
        $event->addSnippet($twig);
    }

    public function onContactConfirm(TemplateEvent $event): void
    {
        $twig = '@OrderInquiry/default/Contact/confirm.twig';
        $event->addSnippet($twig);
    }

    //追加
    public function onContactMail(TemplateEvent $event): void
    {
        $data = $event->getParameter('data');
        $orderNo = $data['order_no'] ?? null;
        if ($orderNo) {
            $label = $this->translator->trans('order_inquiry.common.order_number');
            $data['contents'] = $label . "【" . $orderNo . "】" . "\n" . $data['contents'];
            $event->setParameter('data', $data);
        }
    }
}
```

### 2. MailCatcher で確認
- HTMLメールの場合
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3850968/fd9f7513-fd39-4537-a077-f01b73717d3c.png)

- テキストメールの場合
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3850968/b391e797-003a-4ddb-8e75-bc28251ef1d9.png)

## まとめ
- ボタンを追加するような簡単なプラグインだが、ある程度どのようにしてプラグインが作られているのか感じることができたので、良かった
- イベントリスナーや、Twig拡張、フォーム拡張などを使う
- コアのファイルを書き換えないようにして作る
- ProductController といった既存コントローラはイベントリスナー等で処理を割り込ませる
