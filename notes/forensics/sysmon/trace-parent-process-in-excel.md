---
title: Excel で開いたログのタイムラインから親プロセスを遡る
date: 2026-11-06
category: Forensics
topic: Sysmon
tags: [forensics, sysmon, log-analysis, excel, hayabusa]
summary: 親の PID（できれば PGUID）で検索し、Shift を押しながら「次を検索」で上方向に遡って、親のプロセス作成ログを見つける
---

## 手順

時系列順に並んだログ（Hayabusa の CSV 出力など）を Excel で開いている前提。

1. 調べたい行の親プロセスの PID（可能なら `ParentPGUID`）を確認する。
2. `Ctrl + F` で検索ダイアログを開き、その値を入力する。
3. **Shift を押しながら**「次を検索」をクリックすると、上方向（時間をさかのぼる方向）に検索できる。
4. ヒットした行の `RuleTitle` が `Proc Exec`（プロセス作成のイベント）なら、それが親の起動ログ。さらにその行の親を同じ手順で辿っていく。

## 注意点

- 親は子より前に起動しているので、上方向に探すのが速い。下方向に検索すると、親が起動した別の子プロセスのログばかりがヒットする。
- PID は再利用されるので、PID で探すと関係のないプロセスに当たることがある。ログに PGUID（ProcessGuid）があれば、そちらで検索する。
