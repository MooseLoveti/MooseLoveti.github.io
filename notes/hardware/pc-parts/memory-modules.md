---
title: メモリモジュールの種類（UDIMM・SO-DIMM・RDIMM・ECC など）
date: 2026-10-30
category: Hardware
topic: PC パーツ
tags: [hardware, memory, dimm, ecc]
summary: UDIMM・SO-DIMM・CUDIMM・ECC UDIMM・ECC RDIMM・MRDIMM・HBM が、それぞれどの機器で使われるか
---

| 種類 | 主な用途 |
|---|---|
| UDIMM | 一般のデスクトップ PC |
| SO-DIMM | ノート PC・小型 PC |
| CUDIMM | 高速クロック対応のデスクトップ |
| ECC UDIMM | 一部の小規模ワークステーション・サーバー |
| ECC RDIMM | 本格的なワークステーション・サーバー |
| MRDIMM | 対応する高性能サーバープラットフォーム |
| HBM | 主に GPU・AI アクセラレータ用の高帯域メモリ |

## 用語

- ECC（Error Correcting Code）: メモリ上のビット誤りを検出・訂正する機能。長時間止められないサーバーやワークステーションで使う。
- RDIMM（Registered DIMM）: メモリコントローラとチップの間にレジスタ（バッファ）を挟んだモジュール。大容量・多枚数を安定して載せられる。
- CUDIMM: モジュール上にクロックを整えるチップ（クロックドライバ）を載せた UDIMM。高いクロックでも安定して動かせる。

## 注意点

- 種類ごとに対応するマザーボード・CPU が決まっていて、基本的に混ぜて使えない。RDIMM は一般向けのデスクトップでは動かず、UDIMM はサーバー向けのボードで使えないことが多い。
