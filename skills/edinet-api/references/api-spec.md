# EDINET API V2 仕様リファレンス

## 目次
1. [概要](#概要)
2. [書類一覧API](#書類一覧api)
3. [書類取得API](#書類取得api)
4. [ステータスコード](#ステータスコード)
5. [コードリスト](#コードリスト)
6. [特殊ケース](#特殊ケース)

---

## 概要

EDINET APIはEDINETのデータベースからプログラム経由で開示情報を取得するREST APIである。バージョン2（v2）が現行。

- 通信方式: TLS 1.2以上
- クロスドメイン通信: 不許可（ブラウザJS不可）
- HTTPメソッド: GET
- 認証: APIキー（Subscription-Keyパラメータ）

### 取得対象データの範囲

縦覧期間 + 延長期間 = 閲覧期間内の書類が対象。

| 府令 | 書類種別 | 縦覧期間 | 延長期間 | 閲覧期間 |
|------|----------|----------|----------|----------|
| 企業内容等の開示に関する内閣府令 | 有価証券報告書 | 5年 | 5年 | 10年 |
| 同上 | 半期報告書 | 5年 | 5年 | 10年 |
| 同上 | 四半期報告書 | 3年 | 7年 | 10年 |

- 半期報告書: 2024年4月1日以後提出分が延長期間の対象
- 四半期報告書: 2015年4月1日以後提出分が延長期間の対象
- 特定有価証券の有報等は延長期間なし

---

## 書類一覧API

提出された書類の一覧を日付ごとに取得する。

### リクエスト

**エンドポイント:**
```
https://api.edinet-fsa.go.jp/api/v2/documents.json
```

**リクエストURL:**
```
https://api.edinet-fsa.go.jp/api/v2/documents.json?date={YYYY-MM-DD}&type={1|2}&Subscription-Key={APIキー}
```

**パラメータ:**

| パラメータ名 | 項目名 | 必須 | 設定値 | 説明 |
|-------------|--------|------|--------|------|
| date | ファイル日付 | ○ | YYYY-MM-DD | 出力対象の提出書類一覧のファイル日付。当日以前で直近の財務局営業日の24時において10年を経過していない日付（土日祝も指定可） |
| type | 取得情報 | - | 1（デフォルト） | メタデータのみ |
| | | | 2 | 提出書類一覧及びメタデータ |
| Subscription-Key | APIキー | ○ | 文字列 | 認証用APIキー |

**サンプルURL:**
```
# メタデータのみ（type省略でデフォルト1）
https://api.edinet-fsa.go.jp/api/v2/documents.json?date=2023-04-01&Subscription-Key=ZZZ…ZZZ

# 提出書類一覧及びメタデータ
https://api.edinet-fsa.go.jp/api/v2/documents.json?date=2023-04-01&type=2&Subscription-Key=ZZZ…ZZZ
```

### レスポンス（メタデータ: type=1）

Content-Type: `application/json; charset=utf-8`

| # | 項目名 | 項目ID | 型 | 説明 |
|---|--------|--------|----|------|
| 1 | メタデータ | metadata | object | メタデータの識別子 |
| 2 | タイトル | title | string | APIの名称 |
| 3 | パラメータ | parameter | object | リクエストパラメータ |
| 4 | ファイル日付 | date | string | YYYY-MM-DD |
| 5 | 取得情報 | type | string | 指定した取得情報 |
| 6 | 結果セット | resultset | object | 結果セットの識別子 |
| 7 | 件数 | count | number | 提出書類一覧の件数 |
| 8 | 書類一覧更新日時 | processDateTime | string | YYYY-MM-DD hh:mm。内容に変更がなくても更新される |
| 9 | ステータス | status | string | "200"等 |
| 10 | メッセージ | message | string | "OK"等 |

```json
{
  "metadata": {
    "title": "提出された書類を把握するためのAPI",
    "parameter": { "date": "2023-04-03", "type": "1" },
    "resultset": { "count": 1 },
    "processDateTime": "2023-04-03 13:01",
    "status": "200",
    "message": "OK"
  }
}
```

### レスポンス（提出書類一覧: type=2）

Content-Type: `application/json; charset=utf-8`

メタデータ部分はtype=1と同様。追加で `results` 配列が含まれる。

**resultsの各要素:**

| # | 項目名 | 項目ID | 型 | 説明 |
|---|--------|--------|----|------|
| 12 | 連番 | seqNumber | number | ファイル日付ごとの連番。一度付与されたら変わらない |
| 13 | 書類管理番号 | docID | string(8) | 書類ごとの一意の番号 |
| 14 | 提出者EDINETコード | edinetCode | string(6) | 提出者のEDINETコード |
| 15 | 提出者証券コード | secCode | string(5) | 提出者の証券コード |
| 16 | 提出者法人番号 | JCN | string(13) | 提出者の法人番号 |
| 17 | 提出者名 | filerName | string | 提出者の名前 |
| 18 | ファンドコード | fundCode | string(6) | ファンドコード |
| 19 | 府令コード | ordinanceCode | string(3) | 府令コード |
| 20 | 様式コード | formCode | string(6) | 様式コード |
| 21 | 書類種別コード | docTypeCode | string(3) | 書類種別コード |
| 22 | 期間（自） | periodStart | string | YYYY-MM-DD。有報・半期報告書は事業年度、四半期報告書は四半期会計期間 |
| 23 | 期間（至） | periodEnd | string | YYYY-MM-DD |
| 24 | 提出日時 | submitDateTime | string | YYYY-MM-DD hh:mm |
| 25 | 提出書類概要 | docDescription | string | 書類の概要文字列 |
| 26 | 発行会社EDINETコード | issuerEdinetCode | string(6) | 大量保有の発行会社 |
| 27 | 対象EDINETコード | subjectEdinetCode | string(6) | 公開買付けの対象 |
| 28 | 子会社EDINETコード | subsidiaryEdinetCode | string | カンマ区切り（最大10個） |
| 29 | 臨報提出事由 | currentReportReason | string | カンマ区切り |
| 30 | 親書類管理番号 | parentDocID | string(8) | 親書類の書類管理番号 |
| 31 | 操作日時 | opeDateTime | string | YYYY-MM-DD hh:mm。書類情報修正・不開示・磁気/紙面提出の日時 |
| 32 | 取下区分 | withdrawalStatus | string | "0":通常, "1":取下書, "2":取り下げられた書類 |
| 33 | 書類情報修正区分 | docInfoEditStatus | string | "0":通常, "1":修正情報, "2":修正された書類 |
| 34 | 開示不開示区分 | disclosureStatus | string | "0":通常, "1":不開示開始情報, "2":不開示中, "3":不開示解除情報 |
| 35 | XBRL有無フラグ | xbrlFlag | string | "0" or "1" |
| 36 | PDF有無フラグ | pdfFlag | string | "0" or "1" |
| 37 | 代替書面・添付文書有無フラグ | attachDocFlag | string | "0" or "1" |
| 38 | 英文ファイル有無フラグ | englishDocFlag | string | "0" or "1" |
| 39 | CSV有無フラグ | csvFlag | string | "0" or "1" |
| 40 | 縦覧区分 | legalStatus | string | "1":縦覧中, "2":延長期間中, "0":閲覧期間満了 |

```json
{
  "metadata": {
    "title": "提出された書類を把握するためのAPI",
    "parameter": { "date": "2023-04-03", "type": "2" },
    "resultset": { "count": 2 },
    "processDateTime": "2023-04-03 13:01",
    "status": "200",
    "message": "OK"
  },
  "results": [
    {
      "seqNumber": 1,
      "docID": "S1000001",
      "edinetCode": "E10001",
      "secCode": "10000",
      "JCN": "6000012010023",
      "filerName": "エディネット株式会社",
      "fundCode": "G00001",
      "ordinanceCode": "030",
      "formCode": "04A000",
      "docTypeCode": "030",
      "periodStart": null,
      "periodEnd": null,
      "submitDateTime": "2023-04-03 12:34",
      "docDescription": "有価証券届出書（内国投資信託受益証券）",
      "issuerEdinetCode": null,
      "subjectEdinetCode": null,
      "subsidiaryEdinetCode": null,
      "currentReportReason": null,
      "parentDocID": null,
      "opeDateTime": null,
      "withdrawalStatus": "0",
      "docInfoEditStatus": "0",
      "disclosureStatus": "0",
      "xbrlFlag": "1",
      "pdfFlag": "1",
      "attachDocFlag": "1",
      "englishDocFlag": "0",
      "csvFlag": "1",
      "legalStatus": "1"
    }
  ]
}
```

### 連番の運用

- 同じファイル日付内で一度付与された連番は変わらない
- 当日の一覧を複数回取得する場合、前回の最後の連番より大きい連番のみ処理すれば漏れなく取得可能
- 基本的に「書類の提出日時（操作日時がある場合は操作日時）」の順に追加される

### 更新タイミング

- **当日分**: 提出書類は提出直後に反映。書類情報修正・不開示の開始/解除は操作直後に反映
- **過去分**: 取下げ・書類情報修正・不開示による更新は日次更新処理（日本時間24時過ぎ）で反映

---

## 書類取得API

書類管理番号を指定して書類を取得する。

### リクエスト

**エンドポイント:**
```
https://api.edinet-fsa.go.jp/api/v2/documents/{書類管理番号}
```

**リクエストURL:**
```
https://api.edinet-fsa.go.jp/api/v2/documents/{書類管理番号}?type={1-5}&Subscription-Key={APIキー}
```

**パラメータ:**

| パラメータ名 | 項目名 | 必須 | 設定値 | 説明 |
|-------------|--------|------|--------|------|
| type | 必要書類 | ○ | 1 | 提出本文書及び監査報告書（ZIP） |
| | | | 2 | PDF |
| | | | 3 | 代替書面・添付文書（ZIP） |
| | | | 4 | 英文ファイル（ZIP） |
| | | | 5 | CSV（ZIP） |
| Subscription-Key | APIキー | ○ | 文字列 | 認証用APIキー |

**サンプルURL:**
```
https://api.edinet-fsa.go.jp/api/v2/documents/S1234567?type=1&Subscription-Key=ZZZ…ZZZ
```

### レスポンス

**レスポンスヘッダ Content-Type:**

| Content-Type | 状況 |
|-------------|------|
| application/octet-stream | ZIP形式の取得成功 |
| application/pdf | PDF形式の取得成功 |
| application/json; charset=utf-8 | 取得失敗（エラー） |

**必要書類コードとファイル形式:**

| コード | 名称 | ファイル形式 |
|--------|------|-------------|
| 1 | 提出本文書及び監査報告書 | ZIP |
| 2 | PDF | PDF |
| 3 | 代替書面・添付文書 | ZIP |
| 4 | 英文ファイル | ZIP |
| 5 | CSV | ZIP |

### ZIPファイル構成

**type=1（提出本文書及び監査報告書）:**
```
ZIP/
├── PublicDoc/          # 提出本文書
├── AuditDoc/           # 監査報告書
└── XBRL/
    ├── PublicDoc/      # 提出本文書XBRL
    ├── AuditDoc/       # 監査報告書XBRL
    └── EnglishDoc/     # 英文ファイル
```

**type=3（代替書面・添付文書）:**
```
ZIP/
└── AttachDoc/          # 代替書面・添付文書
```

**type=4（英文ファイル）:**
```
ZIP/
└── EnglishDoc/         # 英文ファイル
```

**type=5（CSV）:**
```
ZIP/
└── XBRL_TO_CSV/        # CSV変換済みファイル
```

### 成功/エラー判定

レスポンスのHTTPステータスは成功/エラーとも"200"になるため、**Content-Typeで判定する**。
- `application/octet-stream` or `application/pdf` → 成功
- `application/json` → エラー（JSON本文にステータスとメッセージ）

---

## ステータスコード

| ステータス | メッセージ | 説明 | 対処 |
|-----------|-----------|------|------|
| 200 | OK | 成功 | - |
| 400 | Bad Request | パラメータ誤り | リクエスト内容（エンドポイント、パラメータ形式）を見直す |
| 401 | Access denied due to invalid subscription key... | APIキー無効 | APIキー発行画面で正しいキーを確認 |
| 404 | Not Found | リソース不在 | パラメータの指定方法と設定値を見直す |
| 500 | Internal Server Error | サーバエラー | EDINETトップページでメンテナンス情報を確認 |

**エラーレスポンス例（400/404/500）:**
```json
{
  "metadata": {
    "title": "提出された書類を把握するためのAPI",
    "status": "404",
    "message": "Not Found"
  }
}
```

**エラーレスポンス例（401）:**
```json
{
  "StatusCode": 401,
  "message": "Access denied due to invalid subscription key. Make sure to provide a valid key for an active subscription."
}
```

---

## コードリスト

### 府令コード

| コード | 名称 |
|--------|------|
| 010 | 企業内容等の開示に関する内閣府令 |
| 015 | 財務計算に関する書類その他の情報の適正性を確保するための体制に関する内閣府令 |
| 020 | 外国債等の発行者の開示に関する内閣府令 |
| 030 | 特定有価証券の内容等の開示に関する内閣府令 |
| 040 | 発行者以外の者による株券等の公開買付けの開示に関する内閣府令 |
| 050 | 発行者による上場株券等の公開買付けの開示に関する内閣府令 |
| 060 | 株券等の大量保有の状況の開示に関する内閣府令 |

### 書類種別コード

| コード | 名称 |
|--------|------|
| 010 | 有価証券通知書 |
| 020 | 変更通知書（有価証券通知書） |
| 030 | 有価証券届出書 |
| 040 | 訂正有価証券届出書 |
| 050 | 届出の取下げ願い |
| 060 | 発行登録通知書 |
| 070 | 変更通知書（発行登録通知書） |
| 080 | 発行登録書 |
| 090 | 訂正発行登録書 |
| 100 | 発行登録追補書類 |
| 110 | 発行登録取下届出書 |
| 120 | 有価証券報告書 |
| 130 | 訂正有価証券報告書 |
| 135 | 確認書 |
| 136 | 訂正確認書 |
| 140 | 四半期報告書 |
| 150 | 訂正四半期報告書 |
| 160 | 半期報告書 |
| 170 | 訂正半期報告書 |
| 180 | 臨時報告書 |
| 190 | 訂正臨時報告書 |
| 200 | 親会社等状況報告書 |
| 210 | 訂正親会社等状況報告書 |
| 220 | 自己株券買付状況報告書 |
| 230 | 訂正自己株券買付状況報告書 |
| 235 | 内部統制報告書 |
| 236 | 訂正内部統制報告書 |
| 240 | 公開買付届出書 |
| 250 | 訂正公開買付届出書 |
| 260 | 公開買付撤回届出書 |
| 270 | 公開買付報告書 |
| 280 | 訂正公開買付報告書 |
| 290 | 意見表明報告書 |
| 300 | 訂正意見表明報告書 |
| 310 | 対質問回答報告書 |
| 320 | 訂正対質問回答報告書 |
| 330 | 別途買付け禁止の特例を受けるための申出書 |
| 340 | 訂正別途買付け禁止の特例を受けるための申出書 |
| 350 | 大量保有報告書 |
| 360 | 訂正大量保有報告書 |
| 370 | 基準日の届出書 |
| 380 | 変更の届出書 |

### 様式コード

別紙1様式コードリストを参照。EDINET閲覧サイトで取得可能。

### EDINETコードリスト・ファンドコードリスト

EDINET閲覧サイトのトップページ → 「EDINETタクソノミ及びコードリスト ダウンロード」→「EDINETコードリスト」から取得可能。

---

## 特殊ケース

### 書類の取下げ

- `withdrawalStatus`: "0"=通常, "1"=取下書, "2"=取り下げられた書類
- 取下げられた書類は、取下書提出日の日本時間24時過ぎに日次更新で連番・docID・parentDocID・withdrawalStatus以外がnullに更新
- 取下書および取下げられた書類は書類取得APIで取得不可
- 親書類が取下げられた場合、子書類（訂正書等）も連動して取り下げられる

### 財務局職員による書類情報修正

- `docInfoEditStatus`: "0"=通常, "1"=修正情報, "2"=修正された書類
- 修正対象項目: fundCode, ordinanceCode, formCode, docTypeCode, periodStart, periodEnd, submitDateTime, docDescription, issuerEdinetCode, subjectEdinetCode, subsidiaryEdinetCode, parentDocID
- 修正された書類自体は変わらない（書類取得APIの結果は修正前後で同一）
- 修正情報は操作日をファイル日付とする一覧に出力される
- 修正された書類のdocInfoEditStatusは翌日の日次更新で"2"に更新

### 財務局職員による書類の不開示

- `disclosureStatus`: "0"=通常, "1"=不開示開始情報, "2"=不開示中, "3"=不開示解除情報
- 不開示書類を書類取得APIで取得すると、不開示を示すPDFが返される
- 全部不開示かつXBRLダウンロード不可の場合、XBRL/CSVは取得不可
- 不開示解除後はdisclosureStatusが"0"に戻る

### 閲覧期間の満了

- `legalStatus`: "1"=縦覧中, "2"=延長期間中, "0"=閲覧期間満了
- 閲覧期間が満了した書類は書類取得APIで取得不可
- 満了日の翌営業日以降、legalStatusが"0"に更新される

### 提出者情報の変更

- EDINETコードに紐づく属性情報（secCode, JCN, filerName等）が変更されても、提出書類一覧上は変更されない
- EDINETコード自体の変更は、新コード発行 + 旧コード廃止で行われる

### 親書類管理番号の関係

- 訂正報告書 → 訂正前の書類
- 変更報告書 → 基となる大量保有報告書/変更報告書
- 発行登録追補書類 → 発行登録書
- 確認書 → 確認対象の有報等
- 公開買付報告書/撤回届出書/対質問回答報告書/意見表明報告書 → 公開買付届出書
- 取下書 → 取下げ対象の書類
