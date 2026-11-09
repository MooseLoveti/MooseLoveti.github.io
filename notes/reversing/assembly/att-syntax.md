---
title: AT&T 記法と Intel 記法の読み分け
date: 2026-11-04
category: Reversing
topic: アセンブリ
tags: [reversing, pwn, assembly, gdb, x86-64]
summary: AT&T 記法はオペランドが「元, 先」の順で、レジスタに %、即値に $ が付く。GDB や objdump の既定は AT&T なので、Intel 記法と取り違えない
---

## 概要

x86 のアセンブリには 2 つの書き方があり、オペランドの順番が逆になる。

| | AT&T 記法 | Intel 記法 |
|---|---|---|
| オペランドの順番 | `命令 元, 先` | `命令 先, 元` |
| レジスタ | `%rax` | `rax` |
| 即値 | `$0x10` | `0x10` |
| メモリ参照 | `-0x8(%rbp)` | `[rbp-0x8]` |
| サイズ | 命令の接尾辞（`movq`、`movl`） | `QWORD PTR` など |

同じ命令の比較:

```
AT&T:  mov    %rax,-0x8(%rbp)          ; rax の値を rbp-0x8 に書き込む
Intel: mov    QWORD PTR [rbp-0x8],rax  ; 同じ意味
```

## 表示の切り替え

```
# GDB（既定は AT&T）
(gdb) set disassembly-flavor intel

# objdump（既定は AT&T）
$ objdump -d -M intel <binary>
```

## 注意点

- GDB の既定は AT&T 記法。gdb-peda などの拡張は Intel 記法に切り替えることが多いので、使う環境によって見た目が変わる。読む前に `%` の有無で記法を判断する。
- `mov %rax,-0x8(%rbp)` を Intel 記法の感覚で読むと「rbp-0x8 の値を rax に読む」と逆に解釈してしまう。書き込み先と読み込み元を取り違えると、カナリアやバッファの位置を誤る。
