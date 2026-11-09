---
title: rootfs（コンテナのルートファイルシステム）
date: 2026-10-31
category: Cloud
topic: コンテナ
tags: [cloud, container, linux, rootfs]
summary: rootfs はコンテナの中から / として見えるディレクトリツリー。コンテナイメージを展開したもので、/bin や /etc などを一式持つ
---

## 概要

rootfs（root filesystem）は、`/` を頂点とするファイルシステム全体のこと。コンテナの文脈では、コンテナの中のプロセスから `/` として見えるディレクトリを指す。中身は `/bin`、`/etc`、`/lib`、`/usr` などを一式そろえたディレクトリで、コンテナイメージを展開すると得られる。

コンテナは、ホスト上のこのディレクトリをプロセスのルートに切り替えて動かす（切り替えには `chroot` や `pivot_root` を使う）。そのため、コンテナの中ではホストとは別の OS の環境に見えるが、カーネルはホストと共有している。

## 手順（rootfs を用意して中に入る例）

```
$ mkdir rootfs
$ docker export $(docker create alpine) | tar -x -C rootfs
$ sudo chroot rootfs /bin/sh
```

## 注意点

- rootfs に入っているのはユーザー空間のファイルだけで、カーネルは含まれない。コンテナの中で `uname -r` を実行すると、ホストのカーネルのバージョンが表示される。
