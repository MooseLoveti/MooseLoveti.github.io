---
title: consent.exe（UAC の確認画面を出すプログラム）とログでの昇格の見分け方
date: 2026-11-06
category: Windows
topic: UAC
tags: [windows, uac, consent, integrity-level, log-analysis]
summary: consent.exe は昇格の要求があったときに AppInfo サービスが起動する確認・資格情報入力の画面。直後に高い整合性レベルのプロセスが出ていれば昇格した証拠
---

## 概要

`consent.exe`（`C:\Windows\System32\consent.exe`）は、管理者権限が必要な操作の要求があったときに、確認画面を表示するプログラム。昇格の要求を受けた AppInfo サービス（`svchost.exe` の中で動く）が、SYSTEM 権限で `consent.exe` を起動し、セキュリティで保護されたデスクトップ（画面が暗くなる状態）に表示させる。

`consent.exe` が出す画面は 2 種類ある。

- 同意プロンプト: 管理者のユーザーが「はい / いいえ」を選ぶ画面。
- 資格情報プロンプト: 標準ユーザーが、管理者のユーザー名とパスワードを入力して昇格させる画面。

ユーザーが許可すると、AppInfo サービスが管理者権限のトークンで目的のプロセスを作る。このとき、親プロセスは `consent.exe` や AppInfo ではなく、昇格を要求した元のプロセスとして記録される。

## ログでの見分け方

| 見るもの | 意味 |
|---|---|
| `consent.exe` の起動 | 昇格の要求があった（許可されたかどうかは、これだけでは分からない） |
| 直後に `HIGH_INTEGRITY` のプロセスが起動している | 昇格が許可された証拠 |
| `MandatoryLabel: HIGH_INTEGRITY` | 整合性レベルが高（SID `S-1-16-12288`）。管理者権限で動いている |
| `TokenElevationType: ELEVATED_TOKEN` | 昇格したトークン（TokenElevationTypeFull）で動いている |

`MandatoryLabel` と `TokenElevationType` は、Windows のセキュリティログのプロセス作成イベント（イベント ID 4688）に含まれる項目。生のログでは `TokenElevationType` は `%%1936`（既定。UAC の対象外）、`%%1937`（昇格済み）、`%%1938`（制限付き。UAC で権限を落とした状態）のように記録される。

## 注意点

- `consent.exe` が起動しているのに、その後に高い整合性レベルのプロセスがなければ、ユーザーが拒否したか、画面を放置した可能性がある。
- 自動昇格（auto-elevate）が設定された Windows の一部のプログラムは、確認画面を出さずに昇格する。そのため、`consent.exe` の記録がないまま高い整合性レベルのプロセスが現れることがあり、`consent.exe` がない ＝ 昇格していない、とは言えない。
- UAC を「常に通知しない」に設定している環境や、UAC 自体を無効にしている環境でも、確認画面は出ない。
