# CLAUDE.md — mytasks

自分専用タスク管理アプリ。仕様は [SERVICE_SPEC.md](SERVICE_SPEC.md) を参照。

## 構成

```
mytasks/
├── index.html       # アプリ本体（HTML/CSS/JS すべてこの1ファイル）
├── SERVICE_SPEC.md
└── CLAUDE.md
```

**ビルド不要・依存ゼロ。** npm も package.json もない。`index.html` を直接編集する。

## 起動

ブラウザで `index.html` を開くだけ。ただし `file://` だと localStorage が使えないブラウザがあるため、
確実に保存したい場合はローカルサーバー経由で開く。

```bash
python -m http.server 8945 --directory .
```

## コードの構造（index.html 内 IIFE）

| セクション | 役割 |
|---|---|
| storage | `load()` / `save()` — localStorage キー `mytasks.v1` |
| dates | `iso` `today` `addDays` `addMonths` `diffDays` `dueLabel` `parseDate` |
| quick-add parsing | `parseInput()` — `#プロジェクト` `@期限` `!`/`!!` `*毎日` を抽出 |
| crud | `addTask` `find` `toggle` `remove` `nextDue` |
| filter & sort | `visible()` `cmp()` `bucket()` — ビュー別の抽出・並び・グルーピング |
| render | `render()` `taskHTML()` `renderStats()` `renderProjects()` `emptyHTML()` |
| events | イベント委譲。`#list` に click / input / change / keydown を1つずつ |
| import/export | JSON 書き出し・読み込み、Markdown コピー、古い完了の整理 |

## 実装ルール

- **全描画は `render()` 経由。** DOM を部分的に書き換えない（`renderStats()` だけは編集中のちらつき回避で例外）。
- **イベントは委譲で書く。** タスクは毎回 innerHTML で作り直すので、個別要素へのリスナー登録は禁止。
- **`data-act` 属性でアクションを表す。** クリック処理は `e.target.closest('[data-act]')` で分岐。
- **ユーザー入力は必ず `esc()` を通す。** `innerHTML` に生の値を入れない。
- **localStorage アクセスは必ず try/catch。** シークレットウィンドウ等で例外になる。
- 外部CDN・フォント・ライブラリを追加しない（オフライン動作が売り）。
- UIテキストは日本語。`console.log` を残さない。

## データ形式を変える場合

`task` にフィールドを増やすときは `newTask()` のデフォルトを必ず埋める（既存データに無いフィールドは
`undefined` になるため）。破壊的変更が必要なら localStorage キーを `mytasks.v2` に上げ、`load()` で
v1 からのマイグレーションを書く。

## 動作確認の観点

ブラウザで開いて以下を通す（自動テストはない）。

1. `テスト #検証 @明日 !!` を追加 → タグが `#検証` `明日` `最優先` になる
2. ビュー 今日 / 今週 / すべて / 完了 を切り替えて、期限バケットの見出しが正しい
3. `*毎日` のタスクを完了 → 翌日分が自動生成される
4. 削除 → トーストの「元に戻す」で復活する
5. リロード → 件数と内容が保持されている
6. テーマ切替が両方向で効く／スマホ幅で1カラムになる
