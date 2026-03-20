# cutting-machine-wizard

**切断機 電気工事仕様書ウィザード** — 日特機械工業株式会社

機種シリーズ・仕様をステップ形式で選択し、電気工事業者向け仕様書PDF を自動生成するシングルファイル Web アプリ。

---

## 概要

| 項目 | 内容 |
|---|---|
| 対象機種 | NCシリーズ（NCO / NCR / NCA / NCB）、INMシリーズ |
| 出力 | 電気工事仕様書 PDF（A4・1ページ） |
| 技術構成 | シングルファイル HTML（HTML + CSS + JavaScript） |
| 依存ライブラリ | jsPDF 2.5.1、html2canvas 1.4.1（cdnjs CDN） |
| データ保存 | LocalStorage（自動保存・復元） |
| デプロイ | GitHub Pages（`main` ブランチ自動デプロイ） |

---

## ウィザードのフロー

### NC シリーズ（NCO / NCR / NCA / NCB）

```
Step 1  基本情報         機種シリーズ・型番・顧客名・現場名・納品日・担当者
Step 2  カバー・安全装置  アクリル/全カバー有無・安全センサー種別・箇所数・投光器
Step 3  モーター仕様      駆動方式・メインモーター容量・インバーター制御・走行モーター
Step 4  制御盤            盤種別・盤寸法・設置位置・電源仕様・プラグ形状
Step 5  給水・付帯設備    給水方式・キャプタイヤコード・長さ・備考
Step 6  確認・PDF出力
```

### INM シリーズ

```
Step 1  基本情報
Step 2  サイズ・カバー    機種サイズ（20インチ / 26インチ以上）・カバー・安全センサー
Step 3  モーター仕様      ブレードモーター・走行モーター・昇降モーター（容量・型番）
Step 4  制御盤・付帯設備  盤寸法・キャプタイヤコード・備考
Step 5  確認・PDF出力
```

> 機種シリーズ選択後、ルートが自動的に切り替わる。

---

## 機能一覧

### ウィザード操作
- **ステップバー自由ジャンプ** — ヘッダーのステップ番号をクリックして任意のステップへ移動可能
- **バリデーション** — 「次へ」ボタン経由の場合のみ必須項目チェック

### データ保存
- **LocalStorage 自動保存** — 入力・選択の変化を即時保存（キー: `cmw_state`）
- **自動復元** — ページ再読み込み後も入力内容・現在ステップを復元
- **保存対象** — テキストフィールド、ラジオボタン全グループ、機種セレクト、ステップ位置

### 備考・特記事項
- Step 1〜4（NC系）/ Step 1〜4（INM系）の各ステップに備考テキストエリアを設置
- 確認画面・PDF ではセクションごとに別行で表示（空欄は非表示）

### PDF 出力
- html2canvas で確認画面を画像キャプチャ → jsPDF で A4 PDF に変換
- **日本語文字化けなし**（フォントレンダリングをブラウザに委任）
- A4 用紙 1 ページに収まるよう各要素のサイズを最適化
- ファイル名: `電気工事仕様書_{型番}_{顧客名}.pdf`

---

## ファイル構成

```
cutting-machine-wizard/
├── index.html          # 本体（HTML + CSS + JS すべて1ファイル）
└── README.md
```

> バージョン管理用として `index(Nc).html` の形式で手元に保存しておく。
> 現在の最新バージョン: **index(7c).html**

---

## デプロイ手順

```
1. index(Nc).html の内容をコピー
2. GitHub の index.html をWebエディタで開く
3. 全選択 → 貼り付け
4. Commit directly to main
5. 1〜2分後に GitHub Pages で確認
```

GitHub Pages URL: `https://junyoshida727.github.io/cutting-machine-wizard/`

---

## 開発ルール

| ルール | 内容 |
|---|---|
| ファイル命名 | `index(Nc).html`（N を 1 つずつ増やす） |
| 作業ブランチ | `dev` で作業 → `main` へ手動マージ |
| 修正指示 | 「基準は Nc」と毎回明示する |
| 変更範囲 | 「その他は一切変更しないでください」を添える |
| 曖昧な点 | 修正前に質問してから実施 |

---

## バージョン履歴

| バージョン | 日付 | 主な変更 |
|---|---|---|
| 1c | 2026-03-11 | 初期リリース（NC系・INM系フロー、PDF出力） |
| 2c | 2026-03-11 | LocalStorage自動保存、ステップバー制限付きジャンプ |
| 3c | 2026-03-11 | ステップバー全ステップ自由ジャンプに変更 |
| 4c | 2026-03-11 | 機種シリーズ選択肢の括弧付き説明文を削除 |
| 5c | 2026-03-11 | 各ステップに備考欄追加・確認画面/PDFに反映 |
| 6c | 2026-03-11 | PDF文字化け修正（html2canvas方式に切替） |
| 7c | 2026-03-11 | PDF 1ページ収録に最適化 |

---

## 技術メモ

### PDF 日本語対応
jsPDF の組み込みフォントは日本語非対応。`html2canvas` でブラウザ上のレンダリング結果を画像化して PDF に貼り込む方式を採用。

```javascript
html2canvas(wrap, { scale: 2, useCORS: true, backgroundColor: '#fff' })
  .then(canvas => {
    const doc = new jsPDF({ format: 'a4' });
    doc.addImage(canvas.toDataURL('image/jpeg', 0.92), 'JPEG', 0, 0, 210, imgH);
    doc.save('filename.pdf');
  });
```

### LocalStorage 復元の注意点
ラジオボタン復元後は、条件表示UIの再トリガーが必要。

```javascript
if(state['r_ncc'])   onNCCov();
if(state['r_inmsz']) onINMSz();
// ... など
```

### データ追加時の修正箇所（4点セット）
新フィールドを追加する際は以下を必ずセットで修正する。

1. `getData()` — フィールド取得
2. `TEXT_IDS` / `RADIO_NAMES` — LocalStorage保存対象
3. `buildReview()` — 確認画面表示
4. `genPDF()` — PDF出力（wrap の HTML）

---

*日特機械工業株式会社 / NITTOKU KIKAI KOGYO Co., Ltd.*
