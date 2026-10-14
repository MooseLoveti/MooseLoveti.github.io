# MooseLoveti.github.io

Takihana Shota のセキュリティポートフォリオサイト。https://takihana.dev で公開している。

## 構成

- `index.html` と `style.css` だけの静的サイト。ビルド工程、パッケージ、JavaScript フレームワークはない。
- GitHub Pages が `main` ブランチのルートをそのまま公開する（`CNAME` は独自ドメイン設定なので変更しない）。
- ページ内のセクション: hero / `#cves` / `#projects` / `#achievements` / `#articles` / `#focus`。
- 色・余白・角丸などは `style.css` 冒頭の `:root` の CSS 変数で管理している。新しい色を直書きせず変数を使う。
- レスポンシブのブレークポイントは 900px と 600px。

## 作業ルール

- 1 issue = 1 PR。issue の範囲外の変更を混ぜない。
- ページ上の文章は日本語。CVE 番号・リンク先・実績の日付など事実情報は、issue で明示されない限り変更しない。
- 外部リンクは `target="_blank" rel="noreferrer"` を付ける。
- 画像には意味のある `alt` を付ける。
- コミットメッセージと PR の説明は日本語で書く。
