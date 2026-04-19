# WorkLedger デザイン仕様書

> 業務報告書・日報からのスタッフ分析、給与計算、履歴推移のための業務管理システム向けデザインシステム。Claude Code が開発者への直接のハンドオフを想定しています。

**対象:** 経理・給与担当/管理者 / **デバイス:** PC (管理画面) / **言語:** 日本語 / **トーン:** Notion/Linear寄り・高密度業務ツール

> ※ 本ファイルは operation-hub プロジェクトへの適用用に、指示書のコア部分（トークン・コンポーネント定義）を抽出保存したもの。元資料の画像・詳細は別途管理。

---

## 1. 設計原則

| # | 原則 | 説明 |
|---|---|---|
| 1 | **情報第一、装飾最小** | 業務ユーザーが1画面で多くの情報を把握できることを最優先 |
| 2 | **数値の視認性を最大化** | 金額・率・件数など数値は等幅フォント(JetBrains Mono)で tabular-nums |
| 3 | **1画面1目的** | 画面のトップに「この画面で何ができるか」を明確にしたタイトル+サマリー |
| 4 | **色は意味を持つ** | 青=情報/主アクション、緑=良好/達成、橙=注意、赤=警告/破棄 |
| 5 | **既存データ構造を尊重** | デザイン変更はUI層に留める |
| 6 | **AIっぽい視覚を避ける** | 過度なグラデーション、丸すぎる角、絵文字アイコン、カラフルカードは使わない |

---

## 2. カラー

全ての色はCSSカスタムプロパティとして `:root` に定義。**ハードコードされた hex 値は禁止。**

### ニュートラル

```css
--bg:              #FAFBFC;  /* ページ背景 */
--surface:         #FFFFFF;  /* カード・テーブル */
--surface-2:       #F4F6F8;  /* th・hover・入力背景 */
--surface-3:       #EEF1F4;  /* バー・バッジ背景 */
--border:          #E4E8EC;  /* 標準ボーダー (1px) */
--border-strong:   #D0D6DC;  /* 強調ボーダー・軸線 */
--text:            #1A1F24;  /* 本文・見出し */
--text-2:          #5B6672;  /* 補助テキスト */
--text-3:          #8C96A1;  /* ラベル・メタ */
--text-mute:       #B0B8C1;  /* プレースホルダ・無効 */
```

### ブランドカラー (プライマリ)

```css
--blue:        #2563EB;  /* 主要アクション・リンク */
--blue-2:      #1D4ED8;  /* hover・押下・強調テキスト */
--blue-soft:   #EFF4FE;  /* 選択状態背景・バッジ */
--blue-border: #C9DAF8;  /* フォーカスリング・強調枠 */
```

### 意味を持つ色 (ステータス)

```css
--green:       #10A86B;  /* 達成・増加・良好 */
--green-soft:  #E6F7EF;
--amber:       #D97706;  /* 注意・確認待ち */
--amber-soft:  #FEF3E2;
--red:         #DC2626;  /* 警告・減少・破棄 */
--red-soft:    #FDECEC;
--purple:      #7C3AED;  /* 補助系列・サブ指標 */
--purple-soft: #F3EDFE;
```

> **色の使用ルール:** 1画面で使う主要色は **青(主アクション) + 2色以下のステータス色** に抑える。

### 色の使い分け早見表

| コンテキスト | 使う色 | NGな使い方 |
|---|---|---|
| ボタン(主)・リンク | `--blue` | 緑や赤で「成功/危険」を示す以外に使わない |
| 数値の増加 | `--green` | 「良い/悪い」の判定なしで使わない |
| 数値の減少 | `--red` | 売上減=赤、コスト減=緑など意味で使い分け |
| 確認待ち・未処理 | `--amber` | 単なる装飾で使わない |
| 破棄・警告 | `--red` | 強度の高い色を多用しない |
| 補助系列(グラフ) | `--purple`, `--amber` | 6系列超の同時表示は避ける |

---

## 3. タイポグラフィ

欧文 Inter + 和文 Noto Sans JP + 数値 JetBrains Mono の3フォント構成。**数値は必ず等幅** にして桁揃え。

### フォントファミリー

```css
--font-sans: "Inter", "Noto Sans JP", -apple-system, BlinkMacSystemFont,
             "Hiragino Kaku Gothic ProN", "Yu Gothic Medium", sans-serif;
--font-mono: "JetBrains Mono", "SF Mono", Menlo, Consolas, monospace;
```

### タイプスケール

| 役割 | サイズ / weight / tracking | 用途 |
|---|---|---|
| Display | 32px / 700 / -0.025em | ドキュメントタイトル |
| Page title | 22px / 700 / -0.02em | ページ見出し `.page__title` |
| Section | 18px / 600 / -0.015em | セクション見出し |
| Card title | 12.5px / 600 | カード見出し `.card__title` |
| KPI value | 22px / 600 / -0.02em | KPI数値 `.kpi__value` |
| Body | 13px / 400 / 1.5 | 本文 (基礎) |
| Small | 12px / 400 | 補助テキスト |
| Caption | 11px / 500 | メタ情報 |
| Label | 10.5-11px / 600 / UC 0.08em | サイドバー見出し等 |
| Number | tabular-nums | 全数値表現 `.num` |

> **必須ルール:** すべての金額・件数・日付・パーセント値には `className="num"` を付与する（厳守）。`font-variant-numeric: tabular-nums` が適用される。

```css
.num {
  font-family: var(--font-mono);
  font-variant-numeric: tabular-nums;
  letter-spacing: -0.01em;
}
```

---

## 4. スペーシング

4px グリッドがベース。**カード内側 14px、カード間 12px、ページ外側 24px** が標準。

### スケール

| px | トークン | 用途 |
|---|---|---|
| 2 | space-0.5 | アイコン内部・微調整 |
| 4 | space-1 | バッジ内・細かい間隔 |
| 6 | space-1.5 | ボタングループ間 |
| 8 | space-2 | アイコンとラベルの間 |
| 12 | space-3 | カード間・要素グループ間 |
| 14 | space-3.5 | カード内側パディング |
| 16 | space-4 | セクション間 |
| 20 | space-5 | 見出し下 |
| 24 | space-6 | ページ左右パディング |
| 40 | space-10 | 大セクション区切り |

### テーブル・カードの密度

| 要素 | 高さ/Padding | 用途 |
|---|---|---|
| テーブル行 (`td`) | `9px 12px` | 高密度・10行以上を見渡す |
| テーブルヘッダ (`th`) | `8px 12px` | sticky, 背景 `--surface-2` |
| カード `.card__head` | `11px 14px` | 下に1px border |
| カード `.card__body` | `14px` | テーブル埋込時は `0` |
| ボタン (標準) | height `28px` / padding `0 12px` | フォント 12px |
| ボタン (小) | height `24px` / padding `0 8px` | 補助アクション |
| 入力フィールド | height `30px` | 検索・フィルタ |

---

## 5. 角丸・影

角丸は **控えめ** に、影は極力使わずボーダーで面を区切る。

### 角丸スケール

| px | トークン | 用途 |
|---|---|---|
| 3 | - | タグ/dot |
| 4 | `--radius-sm` | バー/ボタン(小) |
| 5 | - | ボタン/入力 |
| 6 | `--radius` | カード |
| 8 | `--radius-lg` | モーダル/ドロップダウン |
| full | - | アバター |

### 影

- **境界なし(ボーダーで代替)** → カードはこれ
- **`--shadow-sm`** → セグメント型・微細
- **フロート影** → Tooltip / Modal / Popover のみ

> **注意:** カードに `box-shadow` は付けない。`1px solid var(--border)` で面を区切る。影を使うのは **浮遊する要素**(ドロップダウン、モーダル、トースト) のみ。

---

## 6. レイアウト

3カラム構造(サイドバー固定 + トップバー固定 + 可変コンテンツ)を基本とする。

### アプリケーションシェル

- **Sidebar**: 224px 固定、`position: sticky; top: 0; height: 100vh;`
- **Topbar**: 48px 固定、`position: sticky; top: 0;`
- **Page**: `padding: 18px 24px 40px;`

### コンテンツグリッド

| クラス | 構成 | 用途 |
|---|---|---|
| `.grid.g-4` | 4等分 | KPIストリップ |
| `.grid.g-3` | 3等分 | メトリクス・概要 |
| `.grid.g-2` | 2等分 | チャート + リスト |
| `grid-template-columns: 1fr 340px` | メイン + サイド | メインチャート + リスクパネル |
| `grid-template-columns: 1fr 320px` | テーブル + 情報 | 運用ビュー |

> **コンテンツ幅:** ページ本体は最大 1200-1400px、それ以上広がる場合は `max-width` を設定する。高密度に情報を詰め込むため最小は 1280px で動作保証。

---

## 7. コンポーネント

### Button (`.btn`)

バリアント: `primary` / `default` / `ghost` × サイズ: default(28px) / sm(24px)

```html
<button class="btn btn--primary">確定する</button>
<button class="btn">キャンセル</button>
<button class="btn btn--ghost">…</button>
<button class="btn btn--sm">小サイズ</button>
```

> **1画面のプライマリボタンは1つまで。**

### Badge (`.badge`)

```html
<span class="badge badge--blue">スタッフ</span>
<span class="badge badge--green">達成</span>
<span class="badge badge--amber">確認待</span>
<span class="badge badge--red">要注意</span>
<span class="badge badge--gray">シニア</span>
<span class="badge badge--purple">新人</span>

<!-- dot を付けると、より強い状態表現 -->
<span class="badge badge--green"><span class="dot dot--green"></span>良好</span>
```

### Card (`.card`)

```html
<div class="card">
  <div class="card__head">
    <div>
      <div class="card__title">今月の売上</div>
      <div class="card__sub">2026/04 · 全社</div>
    </div>
    <button class="btn btn--ghost btn--sm">…</button>
  </div>
  <div class="card__body">
    <div class="num" style="font-size:22px; font-weight:600;">¥3,284万</div>
    <div class="kpi__delta delta-up">▲ 4.2% vs 前月</div>
  </div>
</div>
```

### KPI Card (`.kpi`)

必須要素: `label` / `value(.num)` / `delta`

```html
<div class="card kpi">
  <div class="kpi__label">総売上</div>
  <div class="kpi__value num">¥3,284万</div>
  <div class="kpi__delta delta-up">▲ 4.2% vs 前月</div>
</div>
```

delta クラス: `.delta-up`(緑・増加) / `.delta-down`(赤・減少) / `.delta-flat`(グレー・変化なし)

### Data Table (`.tbl`)

- `th` は大文字小 + sticky + 背景 `--surface-2`
- `td` は padding `9px 12px`
- 数値列は `.num` で等幅・右寄せ
- `tbody tr:hover` で `--surface-2` 背景

### Toggle Switch (`.toggle`)

設定項目のON/OFF、機能の有効化、フィルタ切替に使う。

```html
<label class="toggle">
  <input type="checkbox" checked/>
  <span class="toggle__track"><span class="toggle__thumb"></span></span>
  <span class="toggle__label">通知</span>
</label>
```

---

## 8. データ表現

### 数値のフォーマット規則

| 対象 | 表記 | 備考 |
|---|---|---|
| 金額(通常) | `¥1,234,567` | カンマ区切り・¥マーク付き |
| 金額(短縮) | `¥324万` / `¥1.2億` | 一覧・KPI値で使う |
| パーセント | `94.2%` | 小数第1位まで |
| 前月比 | `▲ 4.2%` / `▼ 2.1%` / `─` | 色で正負を表現 |
| 日付 | `2026/04/19` / `04/19` | YYYY/MM/DD 区切り |
| 時刻 | `10:30–19:00` | 全角ダッシュ |
| 件数 | `182件` | 単位を後置 |

---

## 9. パターン

### Do / Don't

**✓ 推奨**
- 数値は必ず等幅フォントで右寄せ
- カード間に 12px、カード内に 14px の余白
- 1画面のプライマリボタンは1つまで
- テーブル行は hover で強調 (`--surface-2`)
- ステータスは「色 + テキスト」両方で伝える
- 空状態はイラストではなく、短いテキスト + アクションで

**✗ 非推奨**
- グラデーション背景を広面積で使う
- カードに box-shadow を付ける
- 数値をプロポーショナルフォントで表示
- 絵文字アイコンを使う
- 装飾的な斜線・波線・パターン
- 角丸 12px 以上のカード
- 複数の彩度の高い色を同時に使う

---

## 10. 移行手順

### STEP 1: トークン導入
CSSカスタムプロパティを `:root` に定義。

### STEP 2: タイポ・フォント切替
Inter + Noto Sans JP + JetBrains Mono を読込。`body` に `var(--font-sans)`、数値には `.num` クラス。

### STEP 3: 共通コンポーネントの置換
`.btn .badge .card .tbl .toggle` を既存命名規則に合わせて移植。

### STEP 4: 画面単位の刷新
優先度順に画面を刷新。1画面=1PR が推奨。

---

## 11. チェックリスト

- [ ] **色**: ハードコード hex が残っていない / 1画面の色数が5色以内 / ステータス色が意味通り
- [ ] **タイポ**: すべての数値に `.num` / 見出しがスケール通り
- [ ] **余白**: カード間 12px / カード内 14px / ページ左右 24px / 4px グリッドに沿っている
- [ ] **レイアウト**: 1280px で横スクロールなし / sticky 要素が適切に効く
- [ ] **コンポーネント**: primary ボタンが1つ / カードに影なし / テーブルヘッダが sticky
- [ ] **データ**: 空状態の表示 / 0件・異常値の扱い / 前月比の±を色で表現
- [ ] **Toggle**: ON/OFF切替には toggle を使用 / ラベルで状態が明確
- [ ] **アクセシビリティ**: キーボード操作可能 / フォーカスリングあり / 色だけに依存しない情報伝達

---

**WorkLedger Design System · v1.0 · 2026/04**
