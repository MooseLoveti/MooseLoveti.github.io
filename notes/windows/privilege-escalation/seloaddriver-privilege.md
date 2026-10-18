---
title: SeLoadDriverPrivilege の悪用
date: 2026-10-17
category: Windows
topic: 権限昇格
tags: [windows, privilege-escalation, seloaddriverprivilege]
summary: カーネルドライバをロードできる特権。脆弱な署名済みドライバと組み合わせるとカーネルレベルのコード実行につながる
---

## 概要

`SeLoadDriverPrivilege` は、カーネルモードのデバイスドライバをロード・アンロードできる特権。ドライバはカーネル空間（最も高い権限レベル）で実行されるため、この特権があれば任意のドライバを読み込ませることで、理論上は OS の最高権限を得られる。

単に自作の未署名ドライバをロードしようとしても、ドライバ署名の強制（Driver Signature Enforcement）によって通常は弾かれる。そのため実際の悪用では、正規に署名されているが脆弱性を持つドライバ（Bring Your Own Vulnerable Driver, BYOVD と呼ばれる手法）をロードし、そのドライバが提供する機能（任意メモリ読み書きなど）を使ってカーネルレベルの操作を行う、という形が一般的。

## 手順

1. `whoami /priv` で `SeLoadDriverPrivilege` が有効になっているか確認する。
2. 脆弱性が知られている署名済みドライバを用意する。
3. その特権でドライバをロードする。
4. ドライバが提供する機能（カーネルメモリの読み書きなど）を使って、権限昇格やセキュリティ機能の無効化（EDR のプロセス保護解除など）につなげる。

## コマンド

```powershell
# SeLoadDriverPrivilege を持っているか確認する
whoami /priv

# 特権を有効にしてドライバを登録・ロードする（例: 脆弱な署名済みドライバ Capcom.sys）
.\EoPLoadDriver.exe System\CurrentControlSet\<service-name> C:\Users\<user>\Capcom.sys

# ロードしたドライバの機能を使って SYSTEM 権限でコマンドを実行する
.\ExploitCapcom.exe
```

## 注意点

- BYOVD は、Defense Evasion（検知回避）の文脈でもよく使われる。EDR のプロセスを保護しているカーネルドライバを、別の脆弱なドライバで無力化する、という形の悪用がその一例。
- 対策としては、脆弱性が報告されているドライバをブロックリスト（Microsoft の脆弱なドライバのブロックリストなど）で拒否する設定を有効にしておくことが重要。
