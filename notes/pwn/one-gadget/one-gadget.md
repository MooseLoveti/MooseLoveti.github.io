---
title: one_gadget（1 回のジャンプでシェルを起動する libc 内のガジェット）
date: 2026-11-01
category: Pwn
topic: one_gadget
tags: [pwn, libc, one-gadget, posix-spawn]
summary: 任意のアドレスに 1 回飛べるだけの状況で、libc 内の execve("/bin/sh") や posix_spawn に至るガジェットを one_gadget で探し、制約を確かめて使う
---

## 概要

one_gadget は、libc の中から「そこに飛ぶだけで `/bin/sh` が起動するコード片（ガジェット）」を探すツール。関数ポインタの上書きなどで任意のアドレスに 1 回だけ飛べるが、引数は用意できない、という状況で使う。

各ガジェットには制約（constraints）が付いていて、飛んだ時点のレジスタやスタックの値がその条件を満たしていないと失敗する。

```
0xebc48 execve("/bin/sh", rsi, rdx)
constraints:
  address rbp-0x78 is writable
  [rsi] == NULL || rsi == NULL || rsi is a valid argv
  [rdx] == NULL || rdx == NULL || rdx is a valid envp
```

`execve` 型のほかに、`posix_spawn` 型のガジェットも出てくる。`posix_spawn()` は新しいプロセスを生成して指定したプログラムを実行する関数で、ここに `/bin/sh` を渡せばシェルが起動する。

## 手順

1. libc のアドレスを 1 つ漏洩させ、libc のベースアドレスを求める（漏洩したアドレス − その関数の libc 内オフセット）。
2. `one_gadget` でガジェットの候補と制約を一覧する。
3. 飛ぶ直前でブレークポイントを張り、各ガジェットの制約に出てくるレジスタやメモリの値を確認する。
4. 制約を満たすガジェットのアドレス（ベース + オフセット）に飛ばす。

## コマンド

```
$ one_gadget <libc.so.6>
$ one_gadget -l 1 <libc.so.6>     # 制約の厳しいものも含めて、より多くの候補を出す
```

```
# 飛ぶ直前でレジスタとメモリを確認する（GDB）
info registers rax rsi rdx r9 r10 r12 r13 rbp
x/gx $rbp-0x78
```

## 注意点

- 最初の一覧で全部の制約が満たせなくても、すぐ諦めない。`-l 1` で候補を増やすと、使えるガジェットが見つかることがある。
- ガジェットの手前の命令も逆アセンブルして見る（`x/6i <libc_base + offset - 8>` など）。制約に引っかかっているレジスタを初期化する命令（`xor eax, eax` など）がすぐ手前にあれば、表示されたアドレスより数バイト手前から飛ぶことで制約を満たせる場合がある。
- 手元と配布環境で libc が違うと、オフセットがずれてガジェットが一切刺さらない。問題で配布された libc（または同じ Docker イメージの libc）に対して `one_gadget` を実行する。
- RELRO が Full でも GOT を書き換えられないだけで、関数ポインタを直接呼ばせる形なら one_gadget は使える。
