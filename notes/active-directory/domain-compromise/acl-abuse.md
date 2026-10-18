---
title: AD の ACL/ACE 悪用（パスワード変更・グループ追加）
date: 2026-10-17
category: Active Directory
topic: ドメイン侵害
tags: [active-directory, acl, privilege-abuse]
summary: オブジェクトに対する強い ACE（ForceChangePassword、GenericAll、GenericWrite など）を持つと、パスワード変更やグループ追加で直接権限を奪える
---

## 概要

AD の各オブジェクト（ユーザー、グループ、コンピュータなど）には、誰が何をできるかを定義する ACL（アクセス制御リスト）と、その個々のエントリである ACE が設定されている。正規の委任設定や過去の設定ミスによって、一般ユーザーに想定以上に強い ACE が付与されていることがある。

代表的な悪用パターン：

- `ForceChangePassword` や `GenericAll` を持つ → 対象ユーザーのパスワードを現在のパスワードを知らなくても強制的に変更できる
- `GenericWrite`（特にグループに対して）や `AddMember` を持つ → 対象グループに自分自身を追加できる（例えば「Domain Admins」グループなど）

BloodHound で収集したグラフ上では、これらは「侵害されたアカウントから目的のアカウント／グループへの最短経路」としてそのまま表示されるため、まず ACL のエッジを辿るのが定石になる。

## 手順

1. BloodHound などで、自分が制御できるアカウントから強い ACE が張られているオブジェクトを探す。
2. ACE の種類に応じて、パスワード変更またはグループメンバー追加を実行する。
3. 変更後のパスワードでログインする、または追加されたグループの権限を使って次の操作に進む。

## コマンド

```
# 対象ユーザーのパスワードを強制変更（ForceChangePassword / GenericAll）
net rpc password 'victim' 'NewPass123!' -U lab.local/user%'Password123' -S <DC_IP>
# またはリモートで
bloodyAD --host <DC_IP> -d lab.local -u user -p 'Password123' set password victim 'NewPass123!'

# 対象グループへ自分を追加（GenericWrite/AddMember）
bloodyAD --host <DC_IP> -d lab.local -u user -p 'Password123' add groupMember 'Domain Admins' user
```

## 注意点

- ACL 悪用は特別な脆弱性ではなく、AD の正規の権限委任機能をそのまま使っている。
- パスワードを強制変更すると、対象ユーザー本人がログインできなくなる。それを避けたいときは、グループ追加や Shadow Credentials（証明書ベースのなりすまし）を使う。
