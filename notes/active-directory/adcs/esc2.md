---
title: ESC2（EKU が Any Purpose の証明書テンプレート）
date: 2026-10-17
category: Active Directory
topic: AD CS
tags: [active-directory, adcs, esc2, certipy]
summary: テンプレートの EKU が Any Purpose になっていると、クライアント認証が許可されていないテンプレートからでもクライアント証明書を発行できてしまう
---

## 概要

証明書の EKU（Extended Key Usage、拡張キー使用法）は、その証明書が「クライアント認証用」「コード署名用」など、どの用途に使えるかを制限するフィールド。ESC2 は、テンプレートの EKU が Any Purpose（あらゆる用途に使える）に設定されている場合に成立する。

本来はクライアント認証の EKU を持たないテンプレートであっても、EKU が Any Purpose であれば、発行された証明書をクライアント認証（PKINIT）に流用できてしまう。これにより、ESC1 と同様の悪用（SAN 指定による他ユーザーへのなりすまし）につながることがある。

## 手順

1. テンプレートの EKU が Any Purpose になっているものを探す。
2. そのテンプレートに対して証明書要求を送る。
3. 発行された証明書を、クライアント認証を含む任意の用途に流用する。

## コマンド

```
certipy req -u user@corp.local -p 'Password123' -ca 'CORP-CA' \
  -target <CA_HOST> -template 'AnyPurposeTemplate'
```

## 注意点

- EKU が Any Purpose、あるいは EKU が一切指定されていない（Any Purpose とほぼ同義に扱われる）テンプレートは、意図せずクライアント認証用途に使われるリスクがあるため、必要な EKU だけに絞って設定する。
- ESC1 と組み合わせて、SAN をクライアントが指定できる設定も重なっていると、さらに悪用しやすくなる。
