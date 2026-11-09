---
title: シェルの $$（自分の PID）
date: 2026-10-31
category: Dev
topic: シェル
tags: [dev, shell, bash, linux]
summary: $$ は現在のシェル自身のプロセス ID に展開される特殊変数。サブシェルの中でも元のシェルの PID のままなので注意
---

## 概要

`$$` は、現在のシェル自身のプロセス ID（PID）に展開される特殊変数。

```
$ echo $$
30540
$ ps
    PID TTY          TIME CMD
  30540 pts/5    00:00:00 bash
  39262 pts/5    00:00:00 ps
```

`ps` の結果の `bash` の PID と一致する。

## 使いどころ

- 自分の情報を `/proc` から見る: `cat /proc/$$/status`
- 一時ファイル名を他のプロセスと重ならないようにする: `/tmp/work.$$`
- コンテナや namespace の検証で、今いるシェルの PID がどう見えているかを確認する（新しい PID namespace の中では小さな番号になる）

## 注意点

- `( ... )` などのサブシェルの中でも、`$$` は元のシェルの PID のまま。サブシェル自身の PID が欲しいときは、bash なら `$BASHPID` を使う。
- 直前にバックグラウンドで起動したプロセスの PID は `$!`。
