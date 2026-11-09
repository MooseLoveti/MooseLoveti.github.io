---
title: Sysmon のログからプロセスインジェクションを見分ける
date: 2026-11-06
category: Forensics
topic: Sysmon
tags: [forensics, sysmon, log-analysis, process-injection]
summary: StartModule・StartFunction が「-」、CallTrace の末尾が UNKNOWN、親プロセス名が空の変な親子関係など、インジェクションを示すログの特徴
---

## 見分けるポイント

| ログの特徴 | 意味 |
|---|---|
| リモートスレッド作成（イベント ID 8）の `StartModule` / `StartFunction` が「-」 | スレッドの開始位置が、どの DLL にも EXE にも属さないメモリ（シェルコードを置いた場所）にある |
| プロセスアクセス（イベント ID 10）の `CallTrace` の末尾が `UNKNOWN(...)` | ファイルに属さないメモリ（＝シェルコード）から呼ばれた。インジェクションが成功し、シェルコードが動いた証拠 |
| 同じプログラムが同じプログラムを起動する変な親子関係で、`ParentProcessName` が空 | プロセスの複製（Dirty Vanity など）。既存のプロセスを複製して、その複製の中でシェルコードを動かす手法 |

## 補足

- イベント ID 8（CreateRemoteThread）は、あるプロセスが別のプロセスの中にスレッドを作ったときに記録される。正規の DLL の関数から始まるスレッドなら `StartModule` に DLL のパスが、`StartFunction` に関数名が入るが、シェルコードを直接指していると空になる（解析ツールの出力では「-」と表示される）。
- `CallTrace` の読み方は「Sysmon のプロセスアクセス（イベント ID 10）と CallTrace の読み方」のノートを参照。
- 1 つの特徴だけで判断せず、どのプロセスがどのプロセスに対して行ったか（`SourceImage` と `TargetImage`）、前後のイベントの流れと合わせて判断する。
