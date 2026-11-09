---
title: PowerShell の Out-String（コマンドの出力を 1 つの文字列にする）
date: 2026-10-29
category: Windows
topic: PowerShell
tags: [windows, powershell]
summary: Out-String はオブジェクトや外部コマンドの出力を、画面表示と同じ形の 1 つの文字列に変換する
---

## 概要

`Out-String` は、パイプラインで受け取ったオブジェクトを、画面に表示されるのと同じ見た目の文字列に変換するコマンドレット。外部の実行ファイル（`.exe`）の出力やコマンドレットの結果を、文字列として検索・加工・保存したいときに使う。

外部コマンドの出力は、PowerShell では 1 行ずつの文字列の配列として受け取られる。`Out-String` を通すと、それが改行を含む 1 つの文字列にまとまる。

## コマンド

```
# 外部コマンドの出力全体を 1 つの文字列として扱う
PS> $out = <command>.exe | Out-String
PS> $out -match "<pattern>"

# コマンドレットの結果（オブジェクト）を表示どおりの文字列にする
PS> $text = Get-Process | Out-String

# 1 行ずつの文字列の配列にしたいとき
PS> Get-Process | Out-String -Stream
```

## 注意点

- 配列のまま `-match` を使うと、マッチした要素（行）が返るだけで `$Matches` は設定されない。出力全体に対して正規表現を当てたいときは、先に `Out-String` で 1 つの文字列にする。
- 変換後はただの文字列なので、元のオブジェクトのプロパティ（`.Id` など）は使えなくなる。プロパティで絞り込む処理は `Out-String` の前に済ませる。
