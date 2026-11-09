---
title: PowerShell の Split-Path（パスから親フォルダーやファイル名を取り出す）
date: 2026-10-29
category: Windows
topic: PowerShell
tags: [windows, powershell]
summary: Split-Path はパスの最後の要素を取り除いて親フォルダーを返す。-Leaf を付けると逆に最後の要素だけを返す
---

## 概要

`Split-Path` は、パス文字列を分割して一部だけを取り出すコマンドレット。オプションなしで実行すると、パスから最後の要素を取り除いた親フォルダーを返す（`-Parent` を付けたのと同じ）。

## コマンド

```
PS> Split-Path "C:\Tools\bin\app.exe"
C:\Tools\bin

PS> Split-Path "C:\Tools\bin\app.exe" -Leaf
app.exe

# 実行中のスクリプトがあるフォルダーを得る
PS> Split-Path -Parent $MyInvocation.MyCommand.Path
```

| オプション | 返すもの |
|---|---|
| なし / `-Parent` | 親フォルダー |
| `-Leaf` | 最後の要素（ファイル名やフォルダー名） |
| `-Qualifier` | ドライブ部分（`C:`） |

## 注意点

- 文字列としてパスを分割するだけなので、そのパスが実在するかは確認しない。
- PowerShell 3.0 以降では、スクリプトのフォルダーは自動変数 `$PSScriptRoot` でも取れる。
