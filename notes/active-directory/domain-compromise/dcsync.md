---
title: DCSync によるドメイン全体のハッシュ抜き取り
date: 2026-10-17
category: Active Directory
topic: ドメイン侵害
tags: [active-directory, dcsync, domain-compromise]
summary: ディレクトリ複製権限（DS-Replication-Get-Changes 系）を悪用し、DC に成りすましてドメイン内全アカウントのハッシュを取得する手法
---

## 概要

AD のドメインコントローラは、互いにディレクトリ情報を複製し合う仕組みを持っている。DCSync は、この複製プロトコルを悪用し、通常のドメインコントローラではない端末が「自分も DC である」かのように振る舞って、ドメイン内の全アカウント（`krbtgt` を含む）のパスワードハッシュを取得する手法。

成立条件は、対象アカウントが `DS-Replication-Get-Changes` と `DS-Replication-Get-Changes-All` という複製権限を持っていること。この権限はドメイン管理者やドメインコントローラのグループには標準で付与されているが、設定ミスで一般アカウントに付与されてしまっていることがある。

`krbtgt` のハッシュが取得できれば、Golden Ticket（偽造 TGT）を作り放題になり、ドメイン全体を永続的に支配できてしまう。

## 手順

1. 対象アカウントが複製権限（`DS-Replication-Get-Changes`/`-All`）を持っているか確認する（BloodHound で確認できる）。
2. 複製権限を持つアカウントの資格情報で、DCSync を実行する。
3. 取得したハッシュ（特に `krbtgt`）を使って、Golden Ticket などの永続化手法につなげる。

## コマンド

```
secretsdump.py lab.local/administrator@<DC_IP> -just-dc
# 特定ユーザーのみ
secretsdump.py -just-dc-user krbtgt lab.local/administrator@<DC_IP>
```
