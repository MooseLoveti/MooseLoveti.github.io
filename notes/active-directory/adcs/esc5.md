---
title: ESC5（CA への ACL が緩く PKI 全体を掌握できる）
date: 2026-10-17
category: Active Directory
topic: AD CS
tags: [active-directory, adcs, esc5]
summary: CA オブジェクトや CA サーバー自体への ACL が緩い場合、PKI インフラ全体を支配できてしまう
---

## 概要

ESC5 は、CA（証明書発行局）に関連する AD オブジェクト（CA サーバーのコンピュータオブジェクト、CA オブジェクト自体、CA が使う設定コンテナなど）に対する ACL が緩く設定されている場合に成立する。これらのオブジェクトへの強い権限（例えば CA サーバーのコンピュータアカウントに対する `GenericAll` など）を持てると、CA サーバー自体を掌握したり、CA の設定を書き換えたりできてしまい、結果として PKI インフラ全体を支配できる。

ESC1〜ESC4 が「テンプレート単位」の設定不備なのに対し、ESC5 は「CA のインフラそのもの」に対するアクセス制御の不備という違いがある。

## 手順

1. CA サーバーのコンピュータオブジェクトや CA 関連の AD オブジェクトへの ACL を確認する。
2. 強い権限（`GenericAll` など）を持てる経路があれば、CA サーバー自体の掌握（マシンの乗っ取り）や CA 設定の書き換えを行う。
3. CA を掌握した状態になれば、任意のテンプレートを有効化したり、ESC7 のような CA 権限の悪用に直接つなげられる。

## コマンド

```bash
# PKI 関連オブジェクトの ACL を含めて、悪用できる設定を列挙する
certipy find -u user@lab.local -p 'Password123' -dc-ip <DC_IP> -vulnerable -stdout
```

```powershell
# Public Key Services コンテナ配下の ACL を確認する
dsacls "CN=Public Key Services,CN=Services,CN=Configuration,DC=lab,DC=local"
```

## 注意点

- BloodHound で、CA サーバーのコンピュータオブジェクトや CA 関連オブジェクトに届く権限エッジがないかを探すのが実務的な調べ方。
