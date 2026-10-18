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
2. `ManageCA` 権限があれば、`EDITF_ATTRIBUTESUBJECTALTNAME2` などの危険フラグを有効化し、以降は ESC6 と同じ手順で悪用する。
3. `ManageCertificates` 権限があれば、拒否されたはずの証明書要求（SAN に管理者を指定したものなど）を手動で承認して発行させる。

## コマンド

```
# ManageCA 権限で危険フラグを有効化（ESC6 化）
certipy ca -u user@corp.local -p 'Password123' -ca 'CORP-CA' -dc-ip <DC_IP> \
  -enable-template 'SubCA'
```

## 注意点

- `ManageCA`/`ManageCertificates` は CA の運用担当者に必要な強い権限であり、付与対象は厳密に絞るべき。BloodHound 等で、これらの権限が想定外のグループやユーザーに付与されていないか確認する。
- ESC6 を無効化していても、ESC7 の権限があれば攻撃者が自分でフラグを立てられてしまうため、フラグの状態だけでなく「誰がそれを変更できるか」まで含めて評価する必要がある。
