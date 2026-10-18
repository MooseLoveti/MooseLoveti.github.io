---
title: AS-REP Roasting（事前認証無効アカウントのオフラインクラック）
date: 2026-10-17
category: Active Directory
topic: Kerberos
tags: [active-directory, kerberos, as-rep-roasting]
summary: Kerberos の事前認証が無効なアカウントに対して AS-REP の暗号化部分を取得し、オフラインでクラックする手法
---

## 概要

通常の Kerberos 認証（AS-REQ/AS-REP）では、クライアントは自分のパスワードハッシュで暗号化したタイムスタンプを事前認証として KDC に送り、それが検証されて初めて TGT（AS-REP）が発行される。これにより、他人のパスワードを知らないと TGT を要求できない仕組みになっている。

ところが、アカウントの設定で「Kerberos の事前認証を要求しない（Do not require Kerberos preauthentication）」が有効になっていると、誰でもそのユーザー名を指定するだけで AS-REP を取得できる。AS-REP の一部はそのユーザーのパスワードハッシュで暗号化されているため、取得した AS-REP をオフラインでクラックすればパスワードが割り出せる。Kerberoasting と違い、事前に有効な資格情報を持っていなくても実行できる点が特徴。

## 手順

1. 事前認証が無効なユーザーを特定する（既知のユーザー名リストで試すか、認証情報がある場合は全ユーザーを自動列挙する）。
2. 対象ユーザー宛てに事前認証なしで AS-REQ を送り、AS-REP を取得する。
3. 取得した AS-REP のハッシュ部分をオフラインでクラックする。

## コマンド

```
# 認証情報あり（全ユーザーを列挙して対象を自動抽出）
GetNPUsers.py lab.local/user:'Password123' -dc-ip <DC_IP> -request

# 認証情報なし（ユーザー名リストで総当り）
GetNPUsers.py lab.local/ -dc-ip <DC_IP> -usersfile users.txt -no-pass
```

取得したハッシュは hashcat（モード 18200）などでオフラインクラックする。

## 注意点

- 事前認証を無効にする設定は、古いシステムとの互換性のために残っていることがある。守る側は、このフラグが立っているアカウントを定期的に棚卸しし、不要なら無効化（＝事前認証を要求する設定に戻す）する。
- Kerberoasting と同様、対象アカウントのパスワードが十分に強力であればオフラインクラックは成立しにくい。
