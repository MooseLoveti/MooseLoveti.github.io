---
title: winPEAS によるローカル権限昇格の糸口の自動探索
date: 2026-10-17
category: Windows
topic: 権限昇格
tags: [windows, privilege-escalation, winpeas]
summary: 弱いサービス権限、保存された資格情報、未パッチ、特権トークンなど権限昇格の糸口をまとめて洗い出す定番ツール
---

## 概要

winPEAS は、Windows ホスト上で権限昇格につながりそうな設定不備や情報を自動的に洗い出すツール。手動で一つひとつ確認していく代わりに、まずこれを実行して出力をざっと眺めるのが、Windows ローカル権限昇格を試す際の定石の第一歩になる。

洗い出される代表的な項目：

- 弱いサービス権限（Weak Service Permissions、Unquoted Service Path など）
- 保存された資格情報（レジストリやファイルに平文で残っているパスワードなど）
- 未パッチの脆弱性（OS バージョンに紐づく既知の権限昇格脆弱性）
- 現在のユーザーが持つ特権トークン（`SeImpersonatePrivilege` など、強力な特権が付与されていないか）

## 手順

1. 対象ホストに winPEAS を持ち込んで実行する。
2. 出力の中で色付け・強調表示されている項目（特に赤系統）を優先的に確認する。
3. 見つかった項目（弱いサービス、特権トークンなど）ごとに、対応する個別の権限昇格手法へつなげる。

## コマンド

```powershell
# 全項目を調べ、結果をファイルに保存する
.\winPEASx64.exe > winpeas.txt

# 調べる項目を絞って速く回す
.\winPEASx64.exe quiet systeminfo userinfo servicesinfo

# .exe が使えない環境では bat 版を使う
.\winPEAS.bat
```

## 注意点

- winPEAS はあくまで「糸口の自動列挙」であり、実際に悪用可能かどうかは個々の手法（Potato、Unquoted Service Path など）を手動で確認・実行する必要がある。
- 出力が非常に大量になるため、全部を読み込もうとせず、まず `whoami /priv` で見えるような明らかに強い特権（`SeImpersonatePrivilege`、`SeBackupPrivilege` など）がないかを優先してチェックすると効率がよい。
