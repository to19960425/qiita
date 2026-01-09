# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 概要

Qiita CLI を使用した技術記事管理リポジトリ。Markdown + YAML フロントマターで記事を作成し、GitHub Actions で自動公開される。

## コマンド

```bash
# Qiita CLI インストール
npm install -g @qiita/qiita-cli

# ローカルプレビュー（localhost:8888）
npx qiita preview

# 新規記事作成
npx qiita new <記事ID>

# 記事公開（通常は GitHub Actions 経由）
npx qiita publish --all
```

## ディレクトリ構成

```
public/                    # 公開記事（git管理対象）
├── <記事ID>.md           # YAML フロントマター付き記事
└── .remote/              # リモート状態キャッシュ（gitignore）

.github/workflows/
└── publish.yml           # main/master プッシュ時に自動公開

qiita.config.json         # CLI設定（port, host, includePrivate）
```

## 記事フォーマット

`public/` 内の各記事は以下の形式:

```markdown
---
title: 記事タイトル
tags:
  - タグ1
  - タグ2
private: false
updated_at: '2025-01-07T15:05:46+09:00'
id: <qiita記事ID>
organization_url_name: null
slide: false
ignorePublish: false
---

本文（Markdown）
```

## 公開フロー

1. `public/` 内で記事を作成・編集
2. `main` または `master` ブランチにプッシュ
3. GitHub Actions が `increments/qiita-cli/actions/publish@v1` で自動公開
4. リポジトリ設定で `QIITA_TOKEN` シークレットが必要

## 注意事項

- `docs/` は gitignore 対象（ローカル専用）
- `.remote/` は Qiita API レスポンスキャッシュ。手動編集不可
