# notes/ の書き方

`notes/` に Markdown ファイルを置くと、`https://takihana.dev/notes/<ファイル名>.html` として公開されます。

## front matter

ファイル先頭に必ず `---` で囲んだ front matter を書きます。これがないと Jekyll が変換せず、生のテキストとして配信されます。

```yaml
---
title: ノートのタイトル
date: 2026-01-01
tags: [web, memo]
summary: 一覧などに表示する一行の説明
---
```

| キー | 内容 |
| --- | --- |
| `title` | ノート名（`<title>` と見出しに使用） |
| `date` | 日付（`YYYY-MM-DD`） |
| `tags` | タグの配列 |
| `summary` | 概要 |

レイアウト（`layout: note`）は `_config.yml` の `defaults` で自動適用されるため、書く必要はありません。

このファイルは `_config.yml` の `exclude` に指定しているため、サイトには公開されません。
