---
title: 【PHP】array_mergeで数値キーが振り直される問題と + 演算子による解決法
tags:
  - PHP
  - array
  - 配列
private: false
updated_at: '2026-01-29T11:12:38+09:00'
id: f4d778aa8577a7fb5396
organization_url_name: null
slide: false
ignorePublish: false
---

実務で配列操作をしていて、今更ながら `array_merge()` の挙動を正しく理解できていなかったので、備忘録として残します。

---

## やりたかったこと

数値キーを持つ連想配列に、別の数値キーで要素を追加したかった。

```php
$array = [1 => 'a', 2 => 'b', 3 => 'c'];

// ここに [7 => 'g'] を追加したい
// 期待する結果: [1 => 'a', 2 => 'b', 3 => 'c', 7 => 'g']
```

---

## 起きた問題

`array_merge()` を使ったところ、キーが 0 から振り直されてしまった。

```php
$array = [1 => 'a', 2 => 'b', 3 => 'c'];
$array2 = array_merge($array, [7 => 'g']);

var_dump($array2);
// array(4) {
//   [0] => string(1) "a"
//   [1] => string(1) "b"
//   [2] => string(1) "c"
//   [3] => string(1) "g"   ← キーが 7 ではなく 3 になっている
// }
```

---

## 文字列キーにしてみたが…

数値キーがダメなら文字列キーにすればいいのでは？と思い試してみた。

```php
$array = [1 => 'a', 2 => 'b', 3 => 'c'];
$array2 = array_merge($array, ['7' => 'g']);

var_dump($array2);
// 結果は同じ…キーが振り直されてしまう
```

PHP では、配列のキーに数値形式の文字列（`'7'` など）を指定すると、**内部的に整数キーとして扱われる**ため、この方法では解決しなかった。

---

## 原因：array_merge() の仕様

[PHP公式マニュアル](https://www.php.net/manual/ja/function.array-merge.php) によると：

> 入力配列が数値添字を持っていた場合、その数値添字は 0 から始まる連番に置き換えられます。

つまり、`array_merge()` は**数値キーを保持しない仕様**になっている。

---

## 解決策：+ 演算子を使う

配列の `+` 演算子（配列結合演算子）を使えば、元のキーを保持したまま配列を結合できる。

```php
$array = [1 => 'a', 2 => 'b', 3 => 'c'];
$array2 = $array + [7 => 'g'];

var_dump($array2);
// array(4) {
//   [1] => string(1) "a"
//   [2] => string(1) "b"
//   [3] => string(1) "c"
//   [7] => string(1) "g"   ← キーが保持されている！
// }
```

---

## array_merge と + 演算子の違い

| 項目 | `array_merge()` | `+` 演算子 |
|:--|:--|:--|
| 数値キー | 0から振り直し | 保持 |
| 文字列キー | 後の配列で上書き | **先の配列を優先** |
| 空配列との結合 | インデックス配列に変換 | 元の配列をそのまま返す |

### 注意：キー重複時の挙動が異なる

```php
$a = [1 => 'a'];
$b = [1 => 'x'];

// array_merge: 後から追加（キーは振り直し）
array_merge($a, $b);  // [0 => 'a', 1 => 'x']

// + 演算子: 先の配列を優先（後は無視）
$a + $b;  // [1 => 'a']
```

---

## まとめ

- `array_merge()` は数値キーを 0 から振り直す仕様
- 数値キーを保持したい場合は `+` 演算子を使う
- ただし `+` 演算子はキー重複時に先の配列を優先するので注意
- 用途に応じて使い分けることが大切

---

## 参考

- [PHP: array_merge - Manual](https://www.php.net/manual/ja/function.array-merge.php)
- [PHP: 配列演算子 - Manual](https://www.php.net/manual/ja/language.operators.array.php)
