---
title: Unquoted Service Path によるサービス実行ファイルの差し替え
date: 2026-10-17
category: Windows
topic: サービス悪用
tags: [windows, privilege-escalation, service]
summary: 引用符で囲まれていない、空白を含むサービスの実行パスを悪用し、先に探索される場所へ偽の実行ファイルを置いて実行させる手法
---

## 概要

Windows サービスの実行ファイルパスが引用符で囲まれておらず、かつパスの途中に空白が含まれている場合、Windows はパスをスペース区切りで分割し、手前の短い候補から順に実行可能ファイルを探していく。これを利用し、攻撃者が探索順の早い位置（空白の手前までのパス）に同名の実行ファイルを配置しておくと、サービス起動時に意図したものではなく攻撃者の実行ファイルが実行されてしまう。

例えば、サービスの実行パスが次のように設定されているとする。

```
C:\Program Files\Some Service\svc.exe
```

このパスが引用符で囲まれていないと、Windows は次の候補を順番に試そうとする。

```
C:\Program.exe
C:\Program Files\Some.exe
C:\Program Files\Some Service\svc.exe
```

このうち `C:\Program Files\Some.exe` に書き込み権限があれば、そこに悪意ある実行ファイルを置くことで、サービスが起動するたびにそれが実行される。サービスが SYSTEM やローカル管理者権限で動作していれば、権限昇格につながる。

## 手順

1. 自動起動（Auto）に設定されたサービスの中から、実行パスが引用符で囲まれておらず、かつ `C:\Windows\` 配下ではないものを探す。
2. パスの空白区切りで生成される候補のうち、書き込み可能なディレクトリを特定する。
3. そのディレクトリに、サービスが期待するファイル名で悪意ある実行ファイルを配置する。
4. サービスを再起動する（または次回の自動起動を待つ）。

## コマンド

```
:: 脆弱なサービスの探索
wmic service get name,displayname,pathname,startmode | findstr /i "auto" | findstr /i /v "c:\windows\\" | findstr /i /v """
```

```powershell
# wmic が無い環境（新しい Windows 11 など）では PowerShell で探す
Get-CimInstance Win32_Service | Where-Object { $_.StartMode -eq 'Auto' -and $_.PathName -notmatch '^"' -and $_.PathName -notlike 'C:\Windows\*' } | Select-Object Name, PathName
```

## 注意点

- サービスの実行ファイルがそのサービスを動かしている権限（多くは SYSTEM）で実行されるため、書き込み権限のあるディレクトリに悪意あるファイルを置けるかどうかが成立の鍵になる。
- 対策はシンプルで、サービスの実行パスを必ず二重引用符で囲むこと。これだけでパスの分割解釈自体が起きなくなる。
