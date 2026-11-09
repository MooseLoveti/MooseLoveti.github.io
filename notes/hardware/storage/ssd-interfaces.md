---
title: SSD の M.2・NVMe・SATA・PCIe 4.0 の違い
date: 2026-10-29
category: Hardware
topic: ストレージ
tags: [hardware, ssd, nvme, sata, m2, pcie]
summary: M.2 は形状（取り付け規格）、NVMe と SATA は通信の方式。M.2 の形でも中身が SATA の SSD があるので注意
---

SSD の仕様に並ぶ用語は、「形状」と「通信方式」という別々の軸を表している。

| 用語 | 何を表すか | 説明 |
|---|---|---|
| M.2 | 形状（取り付け規格） | マザーボードに直接挿す細長いカード型の規格 |
| NVMe | 通信プロトコル | PCIe 経由で SSD とやり取りするための方式。高速 |
| PCIe 4.0 | 接続の世代 | NVMe SSD がつながる PCIe の第 4 世代。4 レーン（x4）でおよそ 8GB/s |
| SATA | 接続規格 | SSD や HDD をつなぐ従来の規格。最大でおよそ 600MB/s |

例えば「M.2 / PCIe 4.0 / NVMe」と書かれていれば、M.2 の形をしていて、PCIe 4.0 の上で NVMe で通信する SSD という意味になる。

## 注意点

- M.2 は形状の名前なので、M.2 の SSD でも中身が SATA 接続のもの（M.2 SATA）がある。速度は SATA と同じなので、速さを求めるなら NVMe と書かれているかを確認する。
- マザーボードの M.2 スロットが NVMe と SATA のどちらに対応しているか（両方か）も確認する。
- PCIe 5.0 のスロットに 4.0 の SSD を挿しても動くが、速度は 4.0 のまま。
