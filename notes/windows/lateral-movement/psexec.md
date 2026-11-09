---
title: PsExec（Sysinternals のリモート実行ツール）のオプション
date: 2026-11-02
category: Windows
topic: 横展開
tags: [windows, lateral-movement, psexec, sysinternals]
summary: PsExec でリモートのホストにコマンドを実行させるときの基本的な書き方と、/accepteula・-c などよく使うオプション
---

## 概要

`psexec.exe` は Sysinternals のリモート実行ツール。管理者の資格情報で対象ホストの管理共有（`ADMIN$`）に接続し、サービスを作ってコマンドを実行させる。正規の管理ツールだが、横展開でもよく使われる。

## コマンド

```
psexec.exe \\<TARGET_IP> -accepteula -u lab\administrator -p Password123 cmd.exe
psexec.exe \\<TARGET_IP> -accepteula -c <local_file.exe>
psexec.exe \\<TARGET_IP> -accepteula -s cmd.exe
```

| オプション | 意味 |
|---|---|
| `-accepteula`（`/accepteula`） | 初回起動時の使用許諾ダイアログを出さずに実行する。自動化や攻撃で必ずと言っていいほど付けられる |
| `-c <file>` | 指定した実行ファイルをリモート側へコピーしてから実行する |
| `-s` | リモート側で SYSTEM 権限で実行する |
| `-u`、`-p` | 接続に使うユーザー名とパスワード |
| `-i` | 対話的なデスクトップに表示させる |

## 注意点

- `-accepteula` を付けないと、初回は許諾ダイアログが出て止まる（非対話の実行では応答できず固まる）。
- 実行には対象ホストの管理者権限と、SMB（445/tcp）で管理共有に接続できることが必要。
- Impacket の `psexec.py` は同じ仕組みを Python で実装した別のツールで、ハッシュでの認証（Pass-the-Hash）にも対応している。
