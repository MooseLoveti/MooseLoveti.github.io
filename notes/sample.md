---
title: サンプルノート
date: 2026-10-14
tags: [sample]
summary: ノート基盤の動作確認用サンプルです。
---

このノートはサンプルです。ノート基盤の表示確認のために置いています。学習内容は含まれていません。

## 表示確認

### 長いコード行

インラインコードは `GET /index.php?id=1' UNION SELECT NULL,NULL,NULL-- -` のように長くても折り返されます。

```
curl -s -X POST 'https://example.com/api/v1/login' -H 'Content-Type: application/json' -d '{"user":"admin","password":"aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa"}'
```

### 表

| 項目 | 内容 | 備考 |
| --- | --- | --- |
| 脆弱性の種類 | クロスサイトスクリプティング | サンプルのため実在の事例ではありません |
| 確認環境 | ローカルの検証用環境 | 外部サービスには実施していません |

### 画像

![サイト運営者のアイコン画像（表示確認用）](/icon/icon.jpg)

> 引用の表示確認です。

- リスト項目 1
- リスト項目 2（[リンク](https://takihana.dev)）
