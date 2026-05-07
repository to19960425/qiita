---
title: 【mPDF】useSubstitutions でフォント未収録の文字をフォールバックさせる
tags:
  - PHP
  - PDF
  - mPDF
  - フォント
  - 文字化け
private: false
updated_at: ''
id: null
organization_url_name: null
slide: false
ignorePublish: false
---

PHPでPDFを生成する [mPDF](https://mpdf.github.io/) には、**メインフォントに無い文字を別フォントから自動で拾ってきてくれる「フォント代替機能（substitution）」** がある。

ただこの機能、`useSubstitutions` という1つのフラグだけでは動かず、**3つの設定が揃ってはじめて有効化される**という少しクセのある仕様になっている。
請求書PDFで「鱻」のような JIS外の漢字が文字化けして悩んだのをきっかけに調べたので、機能の仕組みと正しい設定方法をまとめておく。

---

## TL;DR

mPDFのフォント代替を有効にするには、以下の **3つの設定が同時に必要**。

```php
'useSubstitutions' => true,                     // ① 機能のON/OFFスイッチ
'backupSubsFont'   => ['notosanscjkjp'],        // ② 通常領域(BMP)の代替フォント（配列）
'backupSIPFont'    => 'notosanscjkjp',          // ③ 拡張領域(SIP)の代替フォント（単一）
```

加えて、各 `fontdata` の中に **`sip-ext` キー**を書いておかないと SIP 領域の代替が働かない、という地味な落とし穴もある。

以下、それぞれの役割と動作を順に見ていく。

---

## 前提: なぜフォント代替が必要なのか

### PDFの文字描画は「フォントにその文字が入っているか」が全て

PDFに文字を描画するとき、**フォントファイルにその文字の形（グリフ）が入っていない**と描画できない。
代わりに `?` や白い四角（豆腐 🟦）として出力される。これが文字化けの正体。

たとえば日本語PDFで定番の **IPAex ゴシック**（`ipaexg.ttf`）には、JIS第一・第二水準＋一部の第三・第四水準までしか入っていないため、人名・地名で使われる JIS 外の漢字（例: `鱻` U+9C7B）はカバー外。

```
表示したかった: 株式会社 鱻 商店
PDFの実物    : 株式会社 ? 商店
```

### 解決策: 「無かったら別のフォントから借りる」

そこで mPDF の **フォント代替機能** の出番。
**メインフォントに無い文字だけ、サブのフォントから自動で拾って描画する**仕組みで、これを有効にすれば文字化けを防げる。

```
「鱻」を描きたい
    ↓
メインフォント (IPAex ゴシック) を探す → 無い
    ↓
代替フォント (Noto Sans CJK JP) を探す → あった
    ↓
代替フォントの「鱻」で描画 ✅
```

この機能を制御するのが、本記事の主役 **`useSubstitutions`** とその仲間たち。

---

## 主役の3設定を深掘りする

### ① `useSubstitutions` — 代替機能のメインスイッチ

```php
'useSubstitutions' => true,
```

**フォント代替機能を有効にするかどうかの ON/OFF スイッチ**。
デフォルトは `false` なので、明示的に `true` にしないと代替は一切働かない。

ハマりやすいのが、**この設定だけONにしても何も起きない**こと。
このフラグはあくまで「代替を有効にする」という宣言で、**「どのフォントを代替先に使うか」は別の設定で指定する必要がある**。それが次の②③。

### ② `backupSubsFont` — BMP（普通の文字）用の代替フォント

```php
'backupSubsFont' => ['notosanscjkjp'],
```

メインフォントで **BMP 領域の文字**が描けないときに参照する代替フォント。

> 💡 **BMP（基本多言語面）とは**
> Unicode の住所のうち `U+0000` 〜 `U+FFFF` のエリア。
> 普段使う文字のほぼ全部（日本語の漢字も大半）はここに住んでいる。今回の `鱻`（U+9C7B）もBMP。

**重要なポイントは、値が「配列」であること**。
複数のフォントを優先順位付きで指定でき、先頭から順に試される。

```php
// 例: 中国語フォント → 韓国語フォント → 絵文字フォント、と段階的に探す
'backupSubsFont' => ['mychinese', 'mykorean', 'myemoji'],
```

1つで十分なら、配列に1つだけ入れる形でOK。

### ③ `backupSIPFont` — SIP（超レアな漢字）用の代替フォント

```php
'backupSIPFont' => 'notosanscjkjp',
```

SIP 領域の文字を描くときに使う代替フォント。

> 💡 **SIP（追加漢字面）とは**
> Unicode の住所のうち `U+20000` 〜 `U+2FFFF` のエリア。
> CJK統合漢字拡張Bなど、**さらに超レアな漢字**が住んでいる。

`backupSubsFont` と違い、こちらは **配列ではなく単一のフォント名を文字列で指定**する。
（mPDFの実装上、SIP領域は1つの代替フォントしか登録できない仕様）

### なぜBMPとSIPで分かれているのか

mPDFは内部で **Unicode の面（plane）ごとに描画ロジックを分けている**ため、代替フォントの設定もそれに対応して分かれている。

| 面 | Unicode範囲 | mPDFの設定キー | 値の型 |
|---|---|---|---|
| BMP（基本多言語面） | U+0000 〜 U+FFFF | `backupSubsFont` | **配列**（複数指定可） |
| SIP（追加漢字面） | U+20000 〜 U+2FFFF | `backupSIPFont` | **文字列**（単一指定） |

業務システムで日本語を扱う範囲なら、**両方を同じフォントに指定しておけばまず問題ない**。

---

## 隠れた4つ目の設定: `sip-ext` キー

公式ドキュメントでも見落としがちなのがこれ。

`fontdata` 配列の中で各フォントを定義するとき、**`sip-ext` キーに値を入れないと、そのフォントは SIP 領域の代替候補として認識されない**。

```php
'fontdata' => [
    'notosanscjkjp' => [
        'R'  => 'NotoSansCJKjp-Regular.ttf',
        // ...
        'sip-ext' => 'NotoSansCJKjp-Regular.ttf',  // ← これが必要！
        'type' => 'TrueType',
    ],
],
```

**意味**: 「このフォントは SIP 領域もカバーしていますよ」という mPDFへの申告。値にはファイル名を書く。

これを書き忘れると、`backupSIPFont` でフォントを指定していても **mPDF が「SIP対応フォントは無い」と判断**してしまい、SIP領域の文字が文字化けする。
代替フォント側はもちろん、**メインフォント側にも書いておくこと**（メインで SIP がカバーされていれば、そもそも代替に行かなくて済む）。

---

## 完成形の設定

ここまでの内容を全部入れた `Mpdf` 初期化コードがこちら。

```php
$mpdf = new \Mpdf\Mpdf([
    'fontDir' => [__DIR__ . '/resources/font'],

    'fontdata' => [
        // メインフォント: IPAex ゴシック
        'ipa' => [
            'R'  => 'ipaexg.ttf',
            'B'  => 'ipaexg.ttf',
            'I'  => 'ipaexg.ttf',
            'BI' => 'ipaexg.ttf',
            'sip-ext' => 'ipaexg.ttf',           // ← 忘れずに
            'type' => 'TrueType',
            'useOTL' => 0xFF,
            'useKashida' => 75,
        ],
        // 代替フォント: Noto Sans CJK JP（カバレッジ広め）
        'notosanscjkjp' => [
            'R'  => 'NotoSansCJKjp-Regular.ttf',
            'B'  => 'NotoSansCJKjp-Regular.ttf',
            'I'  => 'NotoSansCJKjp-Regular.ttf',
            'BI' => 'NotoSansCJKjp-Regular.ttf',
            'sip-ext' => 'NotoSansCJKjp-Regular.ttf',  // ← 忘れずに
            'type' => 'TrueType',
            'useOTL' => 0xFF,
            'useKashida' => 75,
        ],
    ],

    'default_font' => 'ipa',

    // 🎯 フォント代替の3点セット
    'useSubstitutions' => true,
    'backupSubsFont'   => ['notosanscjkjp'],
    'backupSIPFont'    => 'notosanscjkjp',
]);
```

---

## 代替フォントの選び方のコツ

### カバレッジが広いフォントを選ぶ

代替フォントの役目は「メインで足りない部分を埋める」こと。なるべく**収録文字数が多い**フォントが向いている。
**[Noto Sans CJK JP](https://fonts.google.com/noto/specimen/Noto+Sans+JP)** は CJK統合漢字拡張まで広くカバーしており、定番。

> 💡 Noto = "**No** more **to**fu"（豆腐をなくそう）が名前の由来。まさにこの用途のためのフォント。

### 書体（ゴシック／明朝）を本文と揃える

技術的には何でもいいが、**本文と代替で書体が混ざると見栄えが崩れる**。

```
株式会社 鱻 商店
        ↑ ここだけ明朝になって浮く
```

最初は IPAmj 明朝を代替に使っていたが、本文（IPAexゴシック）と書体が混ざって見栄えが悪く、ゴシック系の Noto Sans CJK JP に差し替えた。

### OTFは TTFに変換が必要

mPDFが受け付けるのは **TrueType（TTF）形式のみ**。
Noto Sans CJK JP を含む多くのフォントは OTF（OpenType / CFFアウトライン）で配布されているので、`fonttools` の `otf2ttf` コマンドで変換する。

```bash
pip install fonttools
otf2ttf NotoSansCJKjp-Regular.otf
# → NotoSansCJKjp-Regular.ttf が生成される
```

---

## まとめ

mPDF のフォント代替機能を使う上で押さえるべき**4つのチェックリスト**:

- [ ] `useSubstitutions => true` で機能をON
- [ ] `backupSubsFont` に **配列**でBMP用代替フォントを指定
- [ ] `backupSIPFont` に **文字列**でSIP用代替フォントを指定
- [ ] 各 `fontdata` の中に **`sip-ext` キー**を書く（メイン・代替の両方！）

「`useSubstitutions` を `true` にしただけでは何も起きない」という最初のハマりどころと、「`sip-ext` を書き忘れて SIP代替が効かない」という地味な落とし穴さえ把握しておけば、業務PDFの文字化け問題はほぼ片付く。

---

## 参考

- [mPDF Documentation - Fonts & Languages](https://mpdf.github.io/fonts-languages/fonts-in-mpdf-7-x.html)
- [mPDF Documentation - Default Config (useSubstitutions)](https://mpdf.github.io/reference/mpdf-variables/usesubstitutions.html)
- [Noto Sans CJK JP](https://fonts.google.com/noto/specimen/Noto+Sans+JP)
- [fonttools (otf2ttf)](https://github.com/fonttools/fonttools)
