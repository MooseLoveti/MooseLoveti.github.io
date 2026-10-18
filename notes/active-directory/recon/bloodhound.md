---
title: BloodHound / SharpHound による AD の攻撃経路の可視化
date: 2026-10-17
category: Active Directory
topic: 偵察
tags: [active-directory, bloodhound, recon]
summary: AD オブジェクトの関係を収集し、ドメイン管理者への最短経路をグラフで洗い出す定石の偵察手法
---

## 概要

BloodHound は、AD オブジェクト（ユーザー、グループ、ACL、セッション、委任設定、信頼関係など）を収集し、それらの関係をグラフとして可視化するツール。収集した情報から「Shortest Path to Domain Admins」のようなクエリを投げることで、侵害の起点から攻撃経路（Attack Path）を機械的に洗い出せる。AD 環境に足がかりができたら、まずこれを回すのが定石になっている。

収集側のツールは Linux からは Impacket ベースの `bloodhound-python`、Windows 上からは `SharpHound` を使う。どちらも収集結果を BloodHound の GUI（グラフ DB）に取り込んで解析する。

## 手順

1. 一般ドメインユーザーの認証情報を 1 つ入手する。AD では認証済みユーザーに読み取り権限が広く付与されているのが既定のため、資格情報さえあればほぼ常に収集が成立する。
2. 収集ツールを実行し、AD オブジェクトと ACL の情報を出力する（`.json` または `.zip`）。
3. 出力を BloodHound の GUI に取り込み、「Shortest Path to Domain Admins」などの定型クエリでグラフを辿る。
4. グラフ上で `GenericAll`、DCSync 権限（`DS-Replication-Get-Changes` 系）、Kerberoastable（SPN 付きユーザー）などのマーカーが付いたノードを探し、そこを起点に次の攻撃手法（ACL 悪用、Kerberoasting、DCSync など）へつなげる。

## コマンド

```
# Linux から（Impacket ベースの収集）
bloodhound-python -u 'user' -p 'Password123' -d lab.local -ns <DC_IP> -c All

# Windows 上から SharpHound
.\SharpHound.exe -c All --zipfilename loot
```

## 注意点

- BloodHound はあくまで「権限関係の可視化」であり、実際に悪用できるかは個々の権限・サービスの状態に依存する。グラフで見つけたエッジ（`GenericAll` → ShadowCredential 攻撃、DCSync 権限 → secretsdump など）は、対応する攻撃手法のノートを別途参照して実行する。
- 収集は AD 全体のオブジェクトを読み取るため、検知ログ（イベントログや EDR）に痕跡が残りやすい。収集範囲（`-c` オプション）を絞ることで、ノイズを減らせる場合がある。
