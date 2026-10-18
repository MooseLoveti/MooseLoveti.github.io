---
title: ESC4（証明書テンプレートへの書き込み権限の悪用）
date: 2026-10-17
category: Active Directory
topic: AD CS
tags: [active-directory, adcs, esc4, certipy]
summary: テンプレート自体への WriteDacl / WriteOwner / GenericWrite 権限があると、テンプレートを脆弱な設定に書き換えて他の ESC 手法に持ち込める
---

## 概要

ESC4 は、証明書テンプレートオブジェクト自体に対する強い書き込み権限（`WriteDacl`、`WriteOwner`、`GenericWrite` など）を持っている場合に成立する。テンプレートの設定そのものを書き換えられるということは、ESC1 のような「クライアントが SAN を自由に指定できる」設定へテンプレートを作り変えてしまえる、ということ。

つまり ESC4 は単独の悪用手法というより、「他の ESC（主に ESC1）を成立させるための入り口」として位置づけられる。

## 手順

1. 書き込み権限を持つ証明書テンプレートを特定する（BloodHound で `GenericWrite`/`WriteDacl`/`WriteOwner` のエッジとして見える）。
2. そのテンプレートの設定を、クライアントが SAN を指定できる（ESC1 と同じ）設定に書き換える。
3. 書き換え後のテンプレートに対して、ESC1 と同じ手順で証明書を要求する。

## コマンド

```
# テンプレートを脆弱化（SAN 指定可能等に書き換え）
certipy template -u user@corp.local -p 'Password123' -template 'VulnTemplate' \
  -dc-ip <DC_IP> -write-default-configuration
# 以降は ESC1 と同手順で悪用
```

## 注意点

- テンプレートへの書き込み権限は、テンプレートの作成・保守を行う運用グループに限定すべきで、一般ユーザーグループに広く付与されていないか定期的に確認する。
- 悪用後は `certipy` で設定を元に戻すオプションもあるため、侵害調査の際はテンプレートの変更履歴（監査ログ）も確認するとよい。
