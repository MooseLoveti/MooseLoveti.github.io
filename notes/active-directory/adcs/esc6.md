---
title: ESC6（CA の EDITF_ATTRIBUTESUBJECTALTNAME2 フラグ悪用）
date: 2026-10-17
category: Active Directory
topic: AD CS
tags: [active-directory, adcs, esc6, certipy]
summary: CA 側のフラグが有効だと、テンプレートの設定に関わらずクライアントが任意の SAN を指定できてしまう
---

## 概要

ESC6 は、CA（証明書発行局）側の設定フラグ `EDITF_ATTRIBUTESUBJECTALTNAME2` が有効になっている場合に成立する。このフラグが立っていると、個々のテンプレートで SAN のクライアント指定を禁止していても、CA レベルでそれを上書きして、どのテンプレートに対してもクライアントが任意の SAN を指定できるようになってしまう。

つまり ESC1 がテンプレート単位の不備なのに対し、ESC6 は CA 全体に影響する設定不備で、CA 配下の全テンプレートが ESC1 相当の状態になる。

## 手順

1. CA にこのフラグが立っているか確認する。
2. 立っていれば、本来 SAN 指定を許可していないテンプレートに対しても、ESC1 と同じ要領で任意ユーザーの SAN を指定した証明書要求を送る。

## コマンド

```
certipy req -u user@corp.local -p 'Password123' -ca 'CORP-CA' \
  -template 'User' -upn administrator@corp.local
```

## 注意点

- このフラグは古い互換性要件のために有効化されたまま残っていることがある。CA の設定（`certutil -getreg policy\EditFlags` など）で定期的に確認し、不要であれば無効化する。
- ESC6 が有効な環境では、個々のテンプレートの堅牢さに関わらず全体が危険にさらされるため、優先度の高い是正対象になる。
