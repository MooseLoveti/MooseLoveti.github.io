---
title: 補助ベクトル（Auxiliary Vector）と getauxval
date: 2026-11-05
category: Pwn
topic: Linux
tags: [pwn, linux, auxv, getauxval, canary]
summary: カーネルがプロセス起動時に渡す補助情報。getauxval(type) で取得でき、AT_RANDOM はカナリアの元になる 16 バイトの乱数のアドレスを返す
---

## 概要

補助ベクトル（Auxiliary Vector、auxv）は、Linux のカーネルがプロセスの起動時に、引数や環境変数と一緒にスタックに積んで渡す補助情報の一覧。動的リンカや libc の初期化に使われる。

`getauxval(type)` は、`type` に番号を指定して対応する値を取得する関数。返り値は数値またはメモリアドレス。

| 番号 | 名前 | 内容 |
|---|---|---|
| 3 | `AT_PHDR` | ELF プログラムヘッダのアドレス |
| 6 | `AT_PAGESZ` | メモリのページサイズ |
| 7 | `AT_BASE` | 動的リンカのベースアドレス |
| 9 | `AT_ENTRY` | 実行ファイルのエントリーポイントのアドレス |
| 15 | `AT_PLATFORM` | CPU プラットフォーム名の文字列へのポインタ |
| 16 | `AT_HWCAP` | CPU のハードウェア機能を示すビットマスク |
| 23 | `AT_SECURE` | セキュア実行モード（setuid など）かどうか |
| 25 | `AT_RANDOM` | 16 バイトの乱数が格納されたアドレス。スタックカナリアの生成に使われる |
| 31 | `AT_EXECFN` | 実行ファイルのパス文字列へのポインタ |
| 33 | `AT_SYSINFO_EHDR` | vDSO のベースアドレス |

## コマンド

```
# 補助ベクトルを一覧表示する
$ LD_SHOW_AUXV=1 /bin/true

# 実行中のプロセスの補助ベクトル（バイナリ形式）
$ xxd /proc/<pid>/auxv
```

## pwn での使いどころ

- `AT_RANDOM`: 返るのはアドレスで、その先の 16 バイトが乱数。前半 8 バイトの先頭（最下位）1 バイトを `00` にしたものがスタックカナリアになるので、このアドレスの中身を読めればカナリアが分かる。
- `AT_BASE`、`AT_SYSINFO_EHDR`、`AT_ENTRY`、`AT_PHDR`: 動的リンカ、vDSO、実行ファイルのアドレスが分かるので、ASLR や PIE の回避に使える。
- `AT_RANDOM`、`AT_PLATFORM`、`AT_EXECFN` はスタック上のアドレスを返すので、スタックのアドレスの漏洩にもなる。

## 注意点

- `getauxval` が返す値がアドレスか数値かは番号によって違う。アドレスが返る番号では、値そのものではなく、その先のメモリを読む必要がある。
