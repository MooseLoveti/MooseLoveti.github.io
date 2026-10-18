---
title: Potato 系ツールによる SeImpersonatePrivilege の悪用
date: 2026-10-17
category: Windows
topic: 権限昇格
tags: [windows, privilege-escalation, seimpersonateprivilege, potato]
summary: SeImpersonatePrivilege を持つサービスアカウントから、トークン偽装の仕組みを悪用して SYSTEM 権限を得る一連の手法
---

## 概要

Potato 系（PrintSpoofer、GodPotato、JuicyPotato など）は、`SeImpersonatePrivilege` という特権を悪用して SYSTEM 権限を取得するための定番の権限昇格手法。IIS や SQL Server などのサービスアカウントは、この特権を標準で持っていることが多く、見つけたら「ほぼ即座に権限昇格できる」くらい刺さりやすい。

仕組みとしては、SYSTEM 権限で動作する何らかの RPC/COM サービスに、攻撃者が用意した通信相手へ接続させ、そのときに渡ってくる SYSTEM の偽装トークンを `SeImpersonatePrivilege` で奪って利用する、という流れが共通している。具体的な誘導方法（どの COM インターフェースや RPC を使うか）がツールごとに異なり、対応する Windows バージョンも変わってくる。

## 手順

1. `whoami /priv` で `SeImpersonatePrivilege` が有効になっているか確認する。
2. 対象の Windows バージョンに合った Potato 系ツールを選ぶ（新しめの環境では GodPotato、Windows 10/Server 2019 系では PrintSpoofer が定番）。
3. ツールを実行し、SYSTEM 権限のコマンドプロンプトや任意コマンドの実行を得る。

## コマンド

```
:: まず特権確認
whoami /priv

:: PrintSpoofer（Win10/2019 等で定番）
PrintSpoofer.exe -i -c cmd

:: GodPotato（新しめの環境で広く刺さる。.NET 指定）
GodPotato-NET4.exe -cmd "cmd /c whoami"

:: JuicyPotato（古い環境向け。CLSID 指定が要る）
JuicyPotato.exe -l 1337 -p c:\windows\system32\cmd.exe -a "/c whoami" -t * -c {<CLSID>}
```

## 注意点

- `SeImpersonatePrivilege` はサービスアカウントの既定の特権として付与されていることが多く、「見かけたら疑う」くらいの頻出パターン。`whoami /priv` を最初に確認する癖をつけておくと見逃しにくい。
- どの Potato 系ツールが刺さるかは Windows のバージョンやパッチ状況に依存するため、1 つがダメでも別のツールを試す価値がある。
