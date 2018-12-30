# JPStock

## 機能一覧

### 1.株価収集

- 銘柄一覧と株価の最新情報の取得、DB へ格納する

### 2.株価 API 公開

- API キーの生成、更新を行う
- API により株価の提供を行う

### 3.API 広告収入

- 期限付き(1 週間)の API キーの生成、更新を行う
- API キーの生成更新画面に広告を載せる

## タスク管理(Trello)

https://trello.com/b/VsMQZSIM/jpstock

### 利用ルール

- ボード: 未分類/\{担当者\}/レビュー　{担当者\}/完了　{担当者\}
  - 未分類: 新規に発行したカードを配置する
  - {担当者\}: 未分類から積極的に自らアサインする
  - レビュー　{担当者\}: 省略可能。レビューして欲しいユーザーのカードを配置する
  - 完了　{担当者\}: 処理を行った担当者のボードにカードを移動する
- カード:
  - カード名: \[カテゴリ名\]タスク名
    - カテゴリ: 設計/実装/テスト/検討/?
- ラベル: 主要機能のいずれかを選択する

## 開発環境・言語・フレームワーク

- 収集プログラム: Golang
- API: Golang
- API キー生成・更新画面: JavaScript(VueJS)
- API キー生成・更新: Golang
- 未分類: Lambda, CloudWatch, APIGateway, DynamoDB

## 機能詳細

### 1.株価収集

#### DB 構造

- stockNumbers: 収集する銘柄一覧
  - id: ID / Integer / index / auto increment
  - stockCode: 銘柄コード / Integer
  - stockName: 銘柄名 / String
- stocks: 株価一覧
  - stockNumber: 銘柄コード / Integer / index
  - date: 日付 / String
  - openingPrice: 始値 / Integer
  - highPrice: 高値 / Integer
  - lowPrice: 安値 / Integer
  - closingPrice: 終値 / Integer
  - turnover: 出来高 / Integer
- apiKeys: API キー一覧

  - id: ID / Integer / index / auto increment
  - apiKey: API Key / String
  - expiration: DateTime 有効期限 / Date
  - createdAt: 作成日時 / Date
  - updatedAt: 更新日時 / Date

#### 銘柄収集プログラム

銘柄一覧情報を取得し、新規や変更が存在すれば更新する

1. 1 日 1 回、X 情報源を元に銘柄一覧情報を取得する
2. 銘柄一覧情報を繰り返す
   1. 取得した銘柄コードを元に、stockNumbers テーブルから銘柄情報を取得する
   2. 存在する場合、銘柄名に違いがある場合は、更新処理を行う
   3. 存在しない場合、銘柄コード及び銘柄名を新規で格納する
3. 全ての処理終了後、株価収集プログラムを実行する

#### 株価収集プログラム

銘柄一覧を元に、最新の株価情報の取得する

1. stockNumbers テーブルの全てを繰り返し取得する
   1. 取得した銘柄コードを元に、X 情報源から本日の株価情報を取得する
   2. stocks テーブルに株価情報を格納する

### 2.株価 API 公開

#### 株価取得 API

- URI: /api/getStock
- Method:GET

##### Request

- apiKey: API キー / String
- stockCode: 銘柄コード / Integer
- startDate: 開始日時 / String
- endDate: 終了日時 / String

###### Example

```
{
  "apiKey": "01123456789abcdefghijKLMNOPQRSTU",
  "stockCode": 1000,
  "startDate": "2018-01-01",
  "endDate": "2018-12-31"
}
```

##### Response

- stockCode: 銘柄コード / Integer
- stockName: 銘柄名 / String
- startDate: 開始日時 / String
- endDate: 終了日時 / String
- stocks: 株価一覧 / Array
  - date: 日付 / String
  - openingPrice: 始値 / Integer
  - highPrice: 高値 / Integer
  - lowPrice: 安値 / Integer
  - closingPrice: 終値 / Integer
  - turnover: 出来高 / Integer

###### Example

```
{
  "stockCode": 1000,
  "stockName": "銘柄X",
  "startDate": "2018-01-01",
  "endDate": "2018-01-02",
  "stocks": [
    "date": "2018-01-01",
    "openingPrice": 10000,
    "highPrice": 15000,
    "lowPrice": 5000,
    "closingPrice": 10000,
    "turnover": 1000000
  ]
}
```

### 3.API 広告収入

#### API キー発行画面

reCapcha による認証、API キーの生成と期限日時の表示を行う

##### フロントエンド

1. reCapcha v3 通過後、生成ボタンを有効化する
2. 生成ボタン押下後、API キーを生成し、API キーと期限日時を表示する

##### バックエンド

1. reCapcha v3 認証処理を行う
2. 認証に成功した場合、
   1. 現在日時に 2 週間加算した日時を expiration として apiKeys テーブルに格納する
   2. expiration 及び apiKey を返す
3. 認証に失敗した場合、エラーを返す

#### API キー更新画面

reCapcha による認証、期限日時の更新と表示を行う

##### フロントエンド

1. API キーの入力及び、reCapcha v3 通過後、更新ボタンを有効化する
2. 更新ボタン押下後、API キーを更新し、期限日時を表示する

##### バックエンド

1. reCapcha v3 認証処理を行う
2. 認証に成功した場合、
   1. 引数の API キーを元に、apiKeys テーブルから取得する
      1. 取得に成功した場合、現在日時に 2 週間加算した日時を expiration として apiKeys テーブルに格納する
      2. expiration 及び apiKey を返す
   2. 取得に失敗した場合、エラーを返す
3. 認証に失敗した場合、エラーを返す

#### ASP

- Amazon アソシエイト
- 種別: 株やコンピュータに関するものを提供する
