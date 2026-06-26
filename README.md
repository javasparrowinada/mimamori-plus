# solarich 見守りプラス｜メールテンプレート

ソラリッチ見守りプラスのメールテンプレート集と編集UI。GitHub Pagesで公開。

公開URL: https://javasparrowinada.github.io/mimamori-plus/

## ディレクトリ構成

```
mimamori-plus/
├── index.html               トップ（リンク一覧）
├── logic.html               AIレポート仕様
├── notifications.html       通知テンプレート仕様
├── design-b-edit.html       デザインB（編集モード・α版） ★メインの編集UI
├── design.html              デザインA（岩井さん確認用・現在は非表示）
├── design-b.html            デザインB（岩井さん確認用・現在は非表示）
├── design-a-edit.html       デザインA（編集モード・現在は非表示）
├── emails/                  デザインA系メールHTML（11ファイル）
├── emails-b/                デザインB系メールHTML（11ファイル）★こちらを本番採用
└── images/headers/          メールヘッダー画像
    ├── 01-report.png        みまもりレポート用（AIレポート系3通）
    └── 01-report-notice.png みまもり通知用（緊急/通常見守り系8通）
```

## メールテンプレート種別（emails-b/）

| 種別 | ファイル | 用途 |
|---|---|---|
| AIレポート | `AIレポート_01_anshin.html` ほか3通 | 隔週レポート（みまもりレポート） |
| 緊急通知 | `緊急通知_01_停電.html` ほか5通 | 即時アラート（みまもり通知） |
| 通常見守り | `通常見守り_01_AC給電.html` ほか3通 | 日常通知（みまもり通知） |

## 編集モードの使い方（デザイナー / 非エンジニア向け）

[design-b-edit.html](https://javasparrowinada.github.io/mimamori-plus/design-b-edit.html) を開く。

各メールカードに3つのボタン:
- **HTMLダウンロード ↓** : 編集後の完全HTML（base64画像入り）を `.html` ファイルでDL → **エンジニアへ引き継ぐ用**
- **ソースをコピー** : HTMLソースをクリップボードへ（Slack/Notion 等で共有）
- **別画面で開く ↗** : ブラウザでプレビュー表示

編集はブラウザの localStorage に自動保存。「最初に戻す」で初期状態へリセット可能。

## エンジニア向け：メール送信の注意事項

### 1. 送信経路（必須）

**メール送信API経由で送信すること**。Gmail Webコンポーズに貼り付けるとHTMLがsanitizeされ、ダークモード対策などが効かなくなる。

推奨送信サービス:
- SendGrid / Resend / Postmark / Amazon SES など

### 2. 送信時のヘッダー

```
Content-Type: multipart/alternative; boundary="..."
  ├ text/plain  (フォールバック)
  └ text/html; charset=UTF-8  (このリポジトリのHTML)
```

### 3. ドメイン認証（推奨）

SPF / DKIM / DMARC を正しく設定すること。
- 認証されたメールはメールクライアントが信頼し、ダーク変換等の自動調整が緩くなる
- 迷惑メール判定回避にも必須

### 4. 画像の扱い

各メールHTMLでは画像を**相対パス**で参照している:
```html
<img src="../images/headers/01-report.png" alt="みまもりレポート">
```

送信時の選択肢:
- **A. 画像を別ホスト配信**: `https://yourcdn.example.com/headers/01-report.png` に書き換え（推奨）
- **B. base64埋め込み**: HTMLサイズが増えるが、画像ホスト不要

`design-b-edit.html` の「HTMLダウンロード ↓」「ソースをコピー」では、**B（base64埋め込み）**形式で出力される。これをそのまま送信すれば画像ホスト不要。

## ダークモード対応について

### 現状の対策（HTML内に実装済み）

各メールHTMLには以下のライトモード強制宣言が入っている:

```html
<meta name="color-scheme" content="light only">
<meta name="supported-color-schemes" content="light only">
<style>
  :root { color-scheme: light only; }
</style>
<table bgcolor="#FFFFFF" style="background:#FFFFFF;">
  <td bgcolor="#FFFFFF" style="background:#FFFFFF;color:#2D2A26">
```

### メールクライアント別の挙動

| メーラー | ライトモード強制宣言の尊重 | 結果 |
|---|---|---|
| Apple Mail | ✅ 尊重 | ライトモード維持 |
| Outlook (Web / デスクトップ) | ✅ 尊重 | ライトモード維持 |
| Yahoo Mail | ✅ 尊重 | ライトモード維持 |
| Gmail Web | ✅ 尊重 | ライトモード維持 |
| **Gmail iOS アプリ** | ⚠️ 無視することあり | **強制ダーク変換される場合あり** |
| Gmail Android アプリ | ⚠️ 同上 | 同上 |

### Gmail iOSの強制ダーク変換に追加対応する場合（任意）

`<style>` に以下を追記する:

```html
<style>
@media (prefers-color-scheme: dark) {
  .force-light-bg { background:#FFFFFF !important; }
  .force-light-text { color:#2D2A26 !important; }
}
/* Outlook.com 用 */
[data-ogsc] .force-light-bg { background:#FFFFFF !important; }
[data-ogsc] .force-light-text { color:#2D2A26 !important; }
</style>
```

そして対象要素にクラスを追加:
```html
<td class="force-light-bg force-light-text">...</td>
```

ただし Gmail iOS は `@media` も無視することがあるため、完全保証は不可能。最終的にはダーク変換を受容するか、画像化（テキスト部分も含めPNG化）で完全に守るかの選択。

### テスト方法

- **Litmus** / **Email on Acid**（有料）: 全メーラーで一括レンダリング確認
- 実機テスト: Gmail iOS、Apple Mail、Outlook 各環境で受信確認

## 開発・デプロイ

- GitHub Pages による静的ホスティング（`main` ブランチへの push で自動デプロイ）
- ローカル確認: `python3 -m http.server 8000` 等で起動して http://localhost:8000/
