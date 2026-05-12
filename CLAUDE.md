# OPERATION HUB

社内システム統合ポータル。サイドバーから各システム（CLOSING AUTO / MEDIA-MGR / NEXUS OPS / LINE PDF / TEMPLATE 等）を選択し、ウェルカム画面のカード or iframe で 1 画面から呼び出すための入口アプリ。

## ディレクトリ構成

```
operation-hub/
├── index.html              # 単一ファイルアプリ（HTML + CSS + JS + システム定義）
├── favicon.svg             # ファビコン
├── vercel.json             # Vercel 設定（github.silent = true）
├── design-spec/
│   └── SPEC.md             # WorkLedger デザイン仕様書（カラー・コンポーネント定義の正本）
└── CLAUDE.md               # 本ファイル
```

ビルドステップなし。`index.html` 1 枚にすべて同梱されており、編集 → push → Vercel 自動デプロイ。

## 技術スタック

- 純粋な静的サイト（HTML / CSS / 素の JavaScript）
- フレームワーク・バンドラ・パッケージマネージャ **なし**
- ホスティング: Vercel（静的配信、main マージで自動デプロイ）
- フォント: Google Fonts（Inter / JetBrains Mono / Noto Sans JP）
- アイコン: インライン SVG（外部アイコンライブラリ不使用）

## システム定義

`index.html` 内 `const SYSTEMS = [...]` がサイドバー・カード・iframe 遷移先のすべての出元。
新システムを追加する場合はここに `{ id, name, sub, desc, url, icon, children? }` を 1 件追加するだけ。

- `children` を持つシステムはサイドバーに子メニューが展開される（`path` で iframe 遷移先のサブパス指定）
- `url` は本番ドメイン（例 `https://lpd.utinc.dev`）。ローカル開発用 URL は埋め込まない

## デザイン規約（design-spec/SPEC.md 抜粋）

- **色は CSS 変数で参照する**。hex を直書きしない。
  - 主要トークン: `--bg --surface --surface-2 --border --text --text-2 --blue --green --amber --red --purple` 等
- 数値表示は `class="num"` を付与（JetBrains Mono + tabular-nums）
- 角丸は `--radius-sm 4px / --radius 6px / --radius-lg 8px`
- AI っぽさを避ける: 過度なグラデーション・絵文字アイコン・カラフルカード禁止
- 情報密度を優先（業務ツールの想定）
- 詳細は `design-spec/SPEC.md` 参照

## コーディング規約

- TypeScript ではなく素の JS。strict は適用されないが、`const` / `let` を使い `var` 不使用
- 関数はトップレベルに散在させず、コメントの仕切り (`// ============ X ============`) で区切ってグルーピング
- DOM 要素取得はファイル先頭〜該当セクションでまとめて取得（再取得を避ける）
- イベントリスナは `addEventListener` を使用（インライン `onclick=` 禁止）
- 日本語コメント（コミットメッセージも日本語）

## よく使うコマンド

`package.json` がないため、ビルド・テストコマンドは存在しない。

```bash
# ローカル確認（任意の静的サーバ。Python の例）
python -m http.server 8000
# → http://localhost:8000

# 本番デプロイは main への merge で自動。手動デプロイは不要
```

## 環境変数

なし。`.env` は使用していない。すべての遷移先 URL は `index.html` の `SYSTEMS` 配列にハードコード。

## Git ワークフロー

グローバル CLAUDE.md（`~/.claude/CLAUDE.md`）の方針に従う：

- `main` 直コミット禁止。`lc/<種別>/<内容>` ブランチで作業
- push と PR 作成は同一セッションでセット（孤立ブランチ禁止）
- マージは `gh pr merge --squash --delete-branch`
- Vercel プレビュー URL は push 後に必ず提示

## 注意事項

- **iframe 埋め込みできないシステムへの対応**: 子フレームで CSP `frame-ancestors` / `X-Frame-Options: DENY` を返すサイトは表示できない。`index.html` 側に 6 秒タイムアウトのフォールバック UI（「新しいタブで開く」）を実装済み。新規システム追加時は埋め込み可否を確認すること。
- **Vercel Bot コメント抑制**: `vercel.json` の `github.silent: true` を維持。GUI 側の `gitComments.onPullRequest` も OFF にしてあるはず（再設定不要）。
- **バージョン表記**: サイドバー右下 `v2.2.0 · © 2026` を `index.html:532` で更新。手動メンテ。

## TODO / 未対応

- システム数が増えた場合のサイドバー検索 / フィルタ
- `SYSTEMS` 配列を別 JSON に分離するか（現状は HTML 直書き）
- `iframe` 埋め込み不可サイトの自動検知（現状は 6 秒タイムアウト推測）
