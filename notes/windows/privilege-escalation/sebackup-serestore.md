---
title: SeBackupPrivilege / SeRestorePrivilege の悪用
date: 2026-10-17
category: Windows
topic: 権限昇格
tags: [windows, privilege-escalation, sebackupprivilege, serestoreprivilege]
summary: バックアップ特権を使うと、通常のファイル ACL を無視して SAM/SYSTEM レジストリハイブなど任意のファイルを読み取れる
---

## 概要

`SeBackupPrivilege` はファイルシステムのバックアップを行うための特権で、これが有効なアカウントは、通常のファイル ACL による読み取り制限を無視して、任意のファイルを読み取れる（`SeRestorePrivilege` は逆に書き込み側で同様のことができる）。

この特権を悪用すると、本来は SYSTEM 権限でしかアクセスできない SAM ハイブ（ローカルアカウントのハッシュ）と SYSTEM ハイブ（SAM の復号に使うブートキー）を、ACL を無視してそのままコピーできてしまう。取り出したハイブはオフラインで解析し、ローカルアカウントのハッシュを抽出できる。

## 手順

1. `whoami /priv` で `SeBackupPrivilege`（または `SeRestorePrivilege`）が有効か確認する。
2. バックアップ特権を使って、SAM と SYSTEM のレジストリハイブをファイルとして保存する。
3. 保存したハイブをオフラインで解析し、ローカルアカウントのハッシュを抽出する（例えば `secretsdump.py` にローカルファイルとして渡す）。

## コマンド

```
:: SAM と SYSTEM をバックアップ特権で吸い出す
reg save hklm\sam  C:\temp\sam.save
reg save hklm\system C:\temp\system.save
```

```bash
# 手元でハイブからローカルアカウントのハッシュを取り出す
secretsdump.py -sam sam.save -system system.save LOCAL
```

## 注意点

- この特権は、ファイルサーバーやバックアップ運用を行うアカウントに付与されていることが多い。付与対象を必要最小限に絞り、監査ログで `reg save` のような操作が記録されているか確認するとよい。
- 通常のホストで取り出せるのはローカルアカウントのハッシュ。ただし DC 上でこの特権を持っていれば、同じ要領で `ntds.dit` をコピーしてドメイン全体のハッシュを取れる（使用中のファイルなのでシャドウコピー経由で取り出す）。
