---
title: Sysmon のプロセスアクセス（イベント ID 10）と CallTrace の読み方
date: 2026-11-02
category: Forensics
topic: Sysmon
tags: [forensics, sysmon, log-analysis, calltrace]
summary: CallTrace は、あるプロセスが別のプロセスのハンドルを開いた瞬間の呼び出し経路。親が子を起動しただけでも記録される点と、末尾が UNKNOWN のときの意味
---

## 概要

Sysmon のイベント ID 10（ProcessAccess）は、あるプロセス（Source）が別のプロセス（Target）のハンドルを開いたときに記録される。主な項目は次のとおり。

| 項目 | 内容 |
|---|---|
| `SourceImage` | ハンドルを開いた側のプロセス |
| `TargetImage` | 開かれた側のプロセス |
| `GrantedAccess` | 要求したアクセス権（ビットマスク） |
| `CallTrace` | ハンドルを開いたときの呼び出し経路 |

`CallTrace` は、その操作が起きた瞬間に、どのモジュールのどの位置からどの位置へと呼び出されてきたかを、一本の経路として切り取ったもの。`|` 区切りで、`モジュールのパス+オフセット` が呼び出しの深い順（API 側が先頭、呼び出し元が後ろ）に並ぶ。

```
CallTrace: C:\Windows\SYSTEM32\ntdll.dll+9d4c4|C:\Windows\System32\KERNELBASE.dll+2c13e|...|UNKNOWN(000001A2B3C40000)
```

## 読み方

- 親が子を起動しただけでも記録される: 例えば PowerShell がメモ帳を起動すると、プロセスを作った側には子プロセスのハンドルが渡されるので、`SourceImage` が PowerShell、`TargetImage` がメモ帳のイベントが残る。イベント ID 10 があるだけで「不正なアクセス」と判断しない。
- 末尾が `UNKNOWN(...)`: 呼び出し経路のどこかが、ディスク上のどの DLL や EXE にも属さないメモリから来ている。メモリ上に置かれたシェルコードからの呼び出しを示し、プロセスインジェクションが成功してシェルコードが動いた証拠になる。
- 経路に並ぶモジュールを見ると、どのプログラムのどの処理からアクセスされたかが分かる。

## 注意点

- JIT コンパイルを使うプログラム（.NET や JavaScript エンジンなど）は、正規の動作でもファイルに属さないメモリ上でコードを実行するので、`UNKNOWN` が出ることがある。`UNKNOWN` だけで断定せず、`SourceImage` や前後のイベントと合わせて判断する。
