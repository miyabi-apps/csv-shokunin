# テンプレート作成ガイド（JSONリファレンス）

テンプレートは **JSON ファイル 1 つ** で定義します。
`templates` フォルダに JSON ファイルを置いて「再読み込み」を押すだけで、コードを変えずに新しい業務テンプレートを追加できます。

> JSON を書かなくても、画面の「テンプレート管理」→「新規作成」から同じ内容を作成できます。

## 1. 保存場所

| 用途 | 保存場所 |
|---|---|
| 通常（exe 版・開発版とも） | `%LOCALAPPDATA%\csv-shokunin\templates\` |
| 社内の共有フォルダを使いたい場合 | 環境変数 `CSV_SHOKUNIN_TEMPLATES_DIR` にフォルダのパスを設定 |
| 保存領域ごと別の場所にしたい場合（持ち運び用など） | 環境変数 `CSV_SHOKUNIN_DATA_DIR` にフォルダのパスを設定 |

exe の置き場所には関係ありません（exe を移動・更新してもテンプレートはそのまま使えます）。

ファイル名は `ID.json`（例：`sales_summary.json`）です。文字コードは **UTF-8** で保存してください（BOM 付きでも読み込めます）。

## 2. 最小のテンプレート

```json
{
  "id": "my_template",
  "name": "総務_備品一覧",
  "columns": { "output": ["備品名", "数量", "購入日"] },
  "processing": { "remove_unused_columns": true, "convert_dates": true },
  "output": { "format": "excel" }
}
```

省略した項目は初期値になります。

## 3. 全項目

```json
{
  "_comment": "先頭が _ の項目はメモとして自由に書けます（処理には使われません）",
  "schema_version": 1,
  "id": "sales_summary",
  "name": "営業_売上集計",
  "category": "営業",
  "description": "処理内容プレビューに表示される説明",
  "order": 40,

  "columns": {
    "output": ["受注日", "顧客名", "担当者", "売上金額"],
    "remove": ["商品名", "住所", "電話番号"],
    "rename": { "売上金額": "売上（円）" },
    "date_columns": ["受注日"],
    "amount_columns": ["売上金額"],
    "amount_keywords": ["金額", "価格"],
    "aliases": { "顧客名": ["取引先名", "得意先名"] }
  },

  "processing": {
    "remove_unused_columns": true,
    "remove_duplicates": true,
    "trim_blanks": false,
    "convert_dates": true,
    "date_format": "YYYY/MM/DD",
    "normalize_amounts": false
  },

  "output": {
    "format": "excel",
    "encoding": "utf-8-sig",
    "file_name": "売上集計_{date}"
  },

  "excel_summary": {
    "sheet_name": "売上集計",
    "amount_column": "売上金額",
    "group_by": "担当者",
    "chart": true,
    "metric_label": "売上"
  }
}
```

### 基本情報

| 項目 | 必須 | 内容 | 初期値 |
|---|---|---|---|
| `id` | ○ | 半角英数字・`_`・`-`（64 文字まで）。省略時はファイル名 | ファイル名 |
| `name` | ○ | 画面に表示する名前（40 文字まで・他と重複不可） | － |
| `category` | | 分類（経理・営業など自由） | `その他` |
| `description` | | 説明文 | 空 |
| `order` | | 一覧の表示順（小さいほど上） | `100` |
| `schema_version` | | 書式のバージョン | `1` |

### columns（列のルール）

列は **列名** で指定します。CSV の列の並び順が変わっても動きます。
列名の比較では、全角／半角・前後の空白・大文字／小文字の違いを区別しません。

| 項目 | 内容 |
|---|---|
| `output` | 出力する列を、出力したい順番に書きます。**ここに書いた列が CSV に無い場合は実行できません**（画面で ✗ 表示）。空の場合はすべての列を出力します |
| `remove` | 削除する列。CSV に無くてもエラーになりません |
| `rename` | 列名の変更 `{"元の列名": "新しい列名"}` |
| `date_columns` | 日付変換の対象列。空の場合は日付らしい列を自動判定します |
| `amount_columns` | 金額数値化の対象列。空の場合は自動判定します |
| `amount_keywords` | 金額列の自動判定に使う「列名に含まれる言葉」。空の場合は標準（金額・額・価格・単価・料金・代金・費用） |
| `aliases` | 別名 `{"列名": ["別名1", "別名2"]}`。CSV の列名が「取引先名」でも「顧客名」として扱い、出力時の列名は「顧客名」にそろえます |

> `output` を指定し、`processing.remove_unused_columns` が `true` の場合、`output` に書かれていない列はすべて削除されます。

### processing（加工処理）

| 項目 | 内容 | 初期値 |
|---|---|---|
| `remove_unused_columns` | 不要列削除 | `false` |
| `remove_duplicates` | 重複削除（出力する列の値がすべて同じ行を 1 行に） | `false` |
| `trim_blanks` | 空白削除（前後の空白・全角スペース・空行） | `false` |
| `convert_dates` | 日付統一 | `false` |
| `date_format` | `"YYYY/MM/DD"`・`"YYYY-MM-DD"`・`"YYYYMMDD"` | `"YYYY/MM/DD"` |
| `normalize_amounts` | 金額数値化（`1,000円`→`1000`、`△500`→`-500`、`(500)`→`-500`、全角→半角） | `false` |

処理の順番：読込 → 結合 → 空白削除 → 日付統一 → 金額数値化 → 列の削除・並べ替え・名前変更 → 重複削除 → 集計 → 出力

### output（出力）

| 項目 | 内容 | 初期値 |
|---|---|---|
| `format` | `"excel"`（.xlsx）または `"csv"` | `"csv"` |
| `encoding` | CSV の文字コード：`"utf-8-sig"`（BOM 付き・Excel 推奨）・`"utf-8"`・`"cp932"`（Shift_JIS） | `"utf-8-sig"` |
| `file_name` | 出力ファイル名（拡張子なし）。差し込み文字：`{template}` テンプレート名、`{date}` 20260930、`{datetime}` 20260930_153000、`{input}` 1 つ目の入力ファイル名 | `"{template}_{datetime}"` |

### excel_summary（集計シート・省略可）

`output.format` が `"excel"` のときだけ使えます。Excel の先頭に集計シートを追加します。

| 項目 | 内容 | 初期値 |
|---|---|---|
| `amount_column` | 集計する金額の列（**出力時の列名**。必須） | － |
| `sheet_name` | シート名（31 文字まで） | `"売上集計"` |
| `group_by` | 内訳を出す列（例：担当者）。省略すると内訳なし | なし |
| `chart` | 内訳の棒グラフを付けるか（上位 20 件） | `true` |
| `metric_label` | 指標の呼び方。`"請求"` にすると「請求合計・平均請求…」 | `"売上"` |

集計シートの内容：総件数・合計・平均・最大・最小、内訳（件数・合計・構成比）、棒グラフ。
金額が空欄や数値でない行は、合計・平均の計算から除き、件数を表示します。

## 4. 間違いがあった場合

内容に間違いがあるテンプレートは一覧に表示されず、処理結果欄に理由が表示されます。他のテンプレートは通常どおり使えます。

```
テンプレート「sales.json」を読み込めませんでした：
「sales.json」の内容に問題があります。
  ・processing.remove_duplicate は使えない項目名です。もしかして「remove_duplicates」？
  ・processing.date_format は "YYYY/MM/DD"・"YYYY-MM-DD"・"YYYYMMDD" のいずれかで指定してください。
```

## 5. よくある書き間違い

| 間違い | 正しい書き方 |
|---|---|
| `'顧客名'`（シングルクォート） | `"顧客名"` |
| 最後の項目の後ろにカンマ `"a": 1, }` | `"a": 1 }` |
| `true` を `"true"` や `はい` と書く | `true` / `false`（クォートなし） |
| `"output": "顧客名"` | `"output": ["顧客名"]`（リストにする） |
| パスの `\` を 1 つだけ書く | JSON では `\\` と書く |
