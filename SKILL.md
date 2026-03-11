---
name: edinet-api
description: "EDINET API（金融庁 電子開示システム）V2のインターフェース仕様リファレンス。書類一覧API・書類取得APIのエンドポイント、リクエストパラメータ、レスポンスフィールド、ステータスコード、書類種別コード、府令コードなどの仕様情報を提供する。EDINET APIを利用するコードの実装、APIレスポンスの解釈、パラメータの確認、エラーハンドリング、書類種別の特定など、EDINET APIに関する質問やタスクで使用する。トリガー例: 「EDINETの書類一覧を取得するAPIのパラメータは？」「docTypeCode 120は何？」「書類取得APIのレスポンス形式は？」「EDINET APIのエラーコード」"
---

# EDINET API V2 仕様

EDINET API V2は金融庁が提供するREST APIで、有価証券報告書等の開示書類の一覧取得・書類取得を行う。

## API一覧

| API | 用途 | エンドポイント |
|-----|------|---------------|
| 書類一覧API | 日付ごとの提出書類一覧を取得 | `GET /api/v2/documents.json` |
| 書類取得API | 書類管理番号で書類を取得 | `GET /api/v2/documents/{docID}` |

ベースURL: `https://api.edinet-fsa.go.jp`

## 共通仕様

- HTTPメソッド: GET
- 認証: `Subscription-Key` パラメータにAPIキーを指定
- 通信: TLS 1.2以上
- クロスドメイン: 不許可（ブラウザJS不可）

## 書類一覧API 早見表

```
GET /api/v2/documents.json?date={YYYY-MM-DD}&type={1|2}&Subscription-Key={key}
```

| パラメータ | 必須 | 値 |
|-----------|------|-----|
| date | ○ | YYYY-MM-DD（当日以前、10年以内） |
| type | - | 1=メタデータのみ（デフォルト）, 2=書類一覧+メタデータ |
| Subscription-Key | ○ | APIキー |

type=2時のresults配列の主要フィールド: `seqNumber`, `docID`, `edinetCode`, `secCode`, `JCN`, `filerName`, `fundCode`, `ordinanceCode`, `formCode`, `docTypeCode`, `periodStart`, `periodEnd`, `submitDateTime`, `docDescription`, `withdrawalStatus`, `docInfoEditStatus`, `disclosureStatus`, `xbrlFlag`, `pdfFlag`, `attachDocFlag`, `englishDocFlag`, `csvFlag`, `legalStatus`

## 書類取得API 早見表

```
GET /api/v2/documents/{docID}?type={1-5}&Subscription-Key={key}
```

| type値 | 取得内容 | 形式 |
|--------|---------|------|
| 1 | 提出本文書及び監査報告書 | ZIP |
| 2 | PDF | PDF |
| 3 | 代替書面・添付文書 | ZIP |
| 4 | 英文ファイル | ZIP |
| 5 | CSV | ZIP |

成功/エラー判定はContent-Typeで行う（HTTPステータスは常に200）。

## 主要コード早見表

**書類種別コード（よく使うもの）:**
120=有価証券報告書, 130=訂正有報, 140=四半期報告書, 160=半期報告書, 180=臨時報告書, 350=大量保有報告書, 240=公開買付届出書

**府令コード:** 010=企業内容等, 030=特定有価証券, 040=公開買付け（発行者以外）, 060=大量保有

## 詳細仕様

全フィールド定義、全コードリスト、特殊ケース（取下げ・不開示・書類情報修正・閲覧期間満了）の詳細は [references/api-spec.md](references/api-spec.md) を参照。
