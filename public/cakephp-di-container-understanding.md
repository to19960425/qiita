---
title: 【CakePHP5】DDDリファクタリングで気づいた「DIコンテナをちゃんと理解していなかった」話
tags:
  - PHP
  - CakePHP
  - DI
  - DDD
  - 設計
private: false
updated_at: ''
id: null
organization_url_name: null
slide: false
ignorePublish: false
---

保守しているCakePHP案件をDDD設計にリファクタリングしようとしたところ、DIコンテナについてちゃんと理解せずに使っていたことに気づいた。
EC-CUBE案件でもサービスクラスを作ってコントローラで呼ぶということはやっていたが、「なぜそうするのか」を深く考えずに雰囲気で書いていた。自分への整理も兼ねてまとめておく。

---

## これまでの自分の使い方

Symfony（EC-CUBE含む）でサービスクラスを作ってコントローラにコンストラクタ注入する、ということは何度もやっていた。

```php
// Symfonyでよくやっていたパターン
class SomeController
{
    public function __construct(
        private ExampleServiceClass $exampleService  // 具象クラスを直接指定
    ) {}
}
```

これでも動くし、DIコンテナが自動的にインスタンスを作って渡してくれるので、「DI使えてるな」と思っていた。

ただ、これは **DIコンテナの「自動インスタンス化」機能を使っていただけ** で、DIコンテナと組み合わせることでより力を発揮する **「インターフェースを介した依存の抽象化」** は全く活用できていなかった。

---

## CakePHP 5 の `services()` メソッド

CakePHP 5でDDDリファクタリングを進めるにあたり、`Application.php` の `services()` メソッドでDIコンテナの設定を行う必要が出てきた。

```php
public function services(ContainerInterface $container): void
{
    // インターフェース → 実装クラスのバインディング
    $container->add(CouponRepositoryInterface::class, CakeCouponRepository::class);
    $container->add(ApiTokenRepositoryInterface::class, CakeApiTokenRepository::class);
    $container->add(ExternalCouponApiInterface::class, HttpExternalCouponApi::class);

    // Domain Service
    $container->add(CouponMergeService::class);

    // UseCase（依存するインターフェースを明示的に指定）
    $container
        ->add(GetDashboardDataUseCase::class)
        ->addArgument(CouponRepositoryInterface::class)
        ->addArgument(ApiTokenRepositoryInterface::class)
        ->addArgument(ExternalCouponApiInterface::class)
        ->addArgument(CouponMergeService::class);
}
```

### `add` と `addArgument` の違い

| メソッド | 役割 |
|---------|------|
| `add(インターフェース, 実装)` | 「このインターフェースが来たらこのクラスを使え」という**登録** |
| `addArgument(型)` | 「作るときにこれをコンストラクタに渡せ」という**引数指定** |

最初は `add` と `addArgument` の違いがピンと来なかったが、要するに:

- `add` → **何を作るかの登録**
- `addArgument` → **作るときに何を渡すかの指定**

`addArgument` でインターフェースを渡すと、コンテナは先に `add` で登録されたバインディングを参照して実装クラスを解決してくれる。つまり `addArgument(CouponRepositoryInterface::class)` と書けば、コンテナが `CakeCouponRepository` のインスタンスを作って渡してくれる。

---

## 利用する側はインターフェースを書くだけ

バインディングを設定すれば、利用する側は **インターフェースを型ヒントに書くだけ** で、具体的な実装クラスを知る必要がなくなる。

```php
// UseCaseは CakeCouponRepository や HttpExternalCouponApi を知らなくていい
class GetDashboardDataUseCase
{
    public function __construct(
        private CouponRepositoryInterface $couponRepository,
        private ExternalCouponApiInterface $externalApi
    ) {}
}
```

```
GetDashboardDataUseCase が CouponRepositoryInterface を要求
        ↓
DIコンテナが「CakeCouponRepository を使えばいいな」と判断
        ↓
CakeCouponRepository のインスタンスを自動生成して注入
```

---

## 「使えていた」と「理解していた」の違い

改めて整理すると、以前の自分は以下の状態だった。

| | 具象クラス直指定（以前） | インターフェース + バインディング |
|--|--|--|
| 自動インスタンス化 | ✅ | ✅ |
| インターフェース定義 | ❌ | ✅ |
| バインディング設定 | ❌ | ✅ |
| テスト時の差し替え | △（手動で渡せば可能） | ✅（コンテナで一括管理） |
| Clean Architecture | ❌ | ✅ |

EC-CUBE案件でサービスクラスをコントローラで使う、ということはやっていたが、インターフェースを定義してバインディングするところまではやっていなかった。「DIっぽく使っていただけで、DIコンテナの力を引き出せていなかった」ということになる。

---

## DIコンテナを使う理由（自分なりの理解）

1. **テスタビリティ**: テスト時にインターフェースをモックに差し替えられる
2. **疎結合**: 具体的な実装ではなくインターフェースに依存する設計になる
3. **自動注入**: コンストラクタで型ヒントを書くだけでインスタンスが渡される
4. **実装の切り替えが容易**: 本番はHTTP通信、テストはスタブ実装など、設定変更だけで切り替えられる

特に3番は以前から恩恵を受けていたが、1・2・4は意識できていなかった。

---

## 補足

- DIコンテナ自体はDDD専用の概念ではなく、汎用的な設計パターン。ただしDDDやClean Architectureとは非常に相性が良い
- Symfonyでは `services.yaml`、CakePHP 5では `Application.php` の `services()` とやり方は違うが、「`new` は自分でしない、コンテナに任せる」という考え方は同じ
- まだ完全に腹落ちしているわけではないので、実際にリファクタリングを進めながら理解を深めていきたい

---

※ この記事はClaude Codeで作成しています。
