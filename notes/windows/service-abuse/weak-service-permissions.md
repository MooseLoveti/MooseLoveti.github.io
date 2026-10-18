---
title: Weak Service Permissions（サービス構成の書き換えによる権限昇格）
date: 2026-10-17
category: Windows
topic: サービス悪用
tags: [windows, privilege-escalation, service]
summary: サービスの構成（binPath など）を一般ユーザーが変更できる場合、実行コマンドを書き換えて任意コードを実行させる手法
---

## 概要

Windows サービスには、それぞれ「誰がそのサービスの構成を変更できるか」を定めた ACL が設定されている。本来はこの ACL は管理者にのみ変更権限を与えるべきだが、設定ミスにより一般ユーザーでもサービスの `binPath`（実行ファイルパスと引数）を書き換えられることがある。

`binPath` を書き換えられると、サービスが起動・再起動されたタイミングで任意のコマンドを実行させられる。ただし、そのサービスが LocalSystem のような高い権限で動作していなければ、権限昇格としての効果はない。

## 手順

1. `accesschk` などのツールで、自分が変更権限を持つサービスがないか確認する。
2. 対象サービスが LocalSystem などの高い権限で動作していることを確認する。
3. サービスの `binPath` を、任意コマンドを実行するものに書き換える。
4. サービスを再起動する。

## コマンド

```
:: アクセス権確認（accesschk）
accesschk.exe /accepteula -uwcqv user *

:: binPath の書き換え
sc config VulnService binPath= "cmd.exe /c net localgroup administrators attacker /add"
sc stop VulnService & sc start VulnService
```

## 注意点

- このサービスが LocalSystem などの権限で動いていなければ、構成を書き換えられても得られる権限はそのサービスの実行権限止まりで、権限昇格にはならない。悪用前に対象サービスの実行アカウントを必ず確認する。
- 構成を書き換えるには、サービスの ACL で `SERVICE_CHANGE_CONFIG`（SDDL では `DC`）が必要。`WRITE_DAC`（`WD`）や `WRITE_OWNER`（`WO`）があれば、ACL を書き換えてこの権限を自分に付けられる。ACL は `sc sdshow <SERVICE_NAME>` で確認できる。
