---
title: ProcessGuid（PGUID）で PID の再利用に惑わされずにプロセスを追う
date: 2026-11-06
category: Forensics
topic: Sysmon
tags: [forensics, sysmon, log-analysis, process-guid]
summary: PGUID（ProcessGuid）は起動された個々のプロセスを区別する固有の ID。PID が再利用されても別のプロセスとして区別できる
---

## 概要

PID は OS が使い回すため、ログの期間が長いと、同じ PID が別のプロセスに割り当てられていることがある。PID だけで親子関係を追うと、まったく関係のないプロセスを親と取り違えてしまう。

Sysmon は、起動された個々のプロセスに `ProcessGuid` という固有の ID（`{xxxxxxxx-xxxx-...}` の形式）を付けて記録する。PID が再利用されても `ProcessGuid` は重ならないので、同じプロセスかどうかを確実に区別できる。Hayabusa などの解析ツールの出力では、`ProcessGuid` は `PGUID`、親プロセスの `ParentProcessGuid` は `ParentPGUID` と略されることが多い。

## 使い方

- 親を探すときは、子のイベントの `ParentProcessGuid`（`ParentPGUID`）と同じ値を `ProcessGuid` に持つプロセス作成イベント（イベント ID 1）を探す。
- そのプロセスの以降の行動（ネットワーク接続、ファイル作成など）は、同じ `ProcessGuid` を持つイベントを集めると追える。

## 注意点

- `ProcessGuid` は Sysmon が付ける値で、Windows のセキュリティログ（イベント ID 4688）にはない。4688 だけで追うときは、PID と時刻を組み合わせて、PID の再利用がないか確かめる。
