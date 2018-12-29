# JPStock

## 機能一覧
### 1.株価収集
+ サイトからAPIのスクレイピングを行う
+ 取得した情報をDBに格納する
### 2.株価API公開
+ 株価を公開するためのAPIを作成する
+ CSVでも取得できるようにする
### 3.API広告収入
+ 期限付きのAPIキーを設ける(2週間など)
+ APIキーの発行更新画面に広告を載せる

## タスク一覧(Trello)
https://trello.com/b/VsMQZSIM/jpstock
### 利用ルール
- ボード: 未分類/\{担当者\}/レビュー/完了
  - 未分類: 新規に発行したカードを配置する
  - 担当者: 未分類のカードから自ら積極的にアサインする
  - レビュー: 省略可能。レビューしてほしいユーザーをアサインし、カードを配置する
  - 完了: 完了したカードを配置する
- カード: 
  - カード名: \[カテゴリ名\]タスク名
    - カテゴリ: 設計/実装/テスト/検討/?
- ラベル: 主要機能のいずれかを選択する

## 開発環境・言語・フレームワーク
+ 収集プログラム: Golang
+ API: Golang
+ APIキー生成・更新画面: JavaScript(VueJS)
+ APIキー生成・更新: Golang
+　Lambda, CloudWatch, APIGateway, DynamoDB

## 機能詳細
### 1.株価収集
#### DB構造
+ stockNumbers: 収集する銘柄一覧
  + id: ID / Integer / index / auto increment
  + stockCode: 銘柄コード / Integer
  + stockName: 銘柄名 / String
+ stocks: 株価一覧
  + stockNumber: 銘柄コード / Integer / index
  + date: 日付 / String
  + openingPrice: 始値 / Integer
  + highPrice: 高値 / Integer
  + lowPrice: 安値 / Integer
  + closingPrice: 終値 / Integer
  + turnover: 出来高 / Integer
+ apiKeys: APIキー一覧
  + id: ID / Integer / index / auto increment
  + apiKey: API Key / String
  + expiration: DateTime有効期限 / Date
  + createdAt: 作成日時 / Date
  + updatedAt: 更新日時 / Date
  
#### 銘柄収集プログラム
1. 1日1回、X情報源を元に銘柄一覧情報を取得する
2. 銘柄コードを元に、stockNumbersテーブルから銘柄情報を取得する
3. 存在する場合、銘柄名に違いがある場合は、更新処理を行う
4. 存在しない場合、銘柄コード及び銘柄名を新規で格納する

#### 株価収集プログラム
1. 1日1回、stockNumbersの全てを繰り返し取得する
  1. 取得した銘柄コードを元に、X情報源から株価情報を取得する
  2. stocksテーブルに株価情報を格納する

### 2.株価API公開
#### 株価取得API
+ URI: /api/getStock
+ Method:GET
##### Request
+ stockCode: 銘柄コード / Integer
+ startDate: 開始日時 / String
+ endDate: 終了日時 / String

###### Example
```
{
  "stockCode": 1000,
  "startDate": "2018-01-01",
  "endDate": "2018-12-31"
}
```
##### Response
+ stockCode: 銘柄コード / Integer
+ stockName: 銘柄名 / String
+ startDate: 開始日時 / String
+ endDate: 終了日時 / String
+ stocks: 株価一覧 / Array
  + date: 日付 / String
  + openingPrice: 始値 / Integer
  + highPrice: 高値 / Integer
  + lowPrice: 安値 / Integer
  + closingPrice: 終値 / Integer
  + turnover: 出来高 / Integer

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

#### APIキー発行画面
1. reCapcha v3通過後、生成ボタンを有効化する
2. 生成ボタン押下後、APIキーを生成し、APIキーと期限日時を表示する

#### APIキー更新画面
1. APIキーの入力及び、reCapcha v3通過後、更新ボタンを有効化する
2. 更新ボタン押下後、APIキーを更新し、期限日時を表示する

#### APIキー発行処理
1. reCapcha v3認証処理を行う
2. 認証に成功した場合、
  1. 現在日時に2週間加算した日時をexpirationとしてapiKeysテーブルに格納する
  2. expiration及びapiKeyを返す
3. 認証に失敗した場合、エラーを返す

#### APIキー更新処理
1. reCapcha v3認証処理を行う
2. 認証に成功した場合、
  1. 引数のAPIキーを元に、apiKeysテーブルから取得する
    1. 取得に成功した場合、現在日時に2週間加算した日時をexpirationとしてapiKeysテーブルに格納する
    2. expiration及びapiKeyを返す
  2. 取得に失敗した場合、エラーを返す
3. 認証に失敗した場合、エラーを返す

### 3.API広告収入
#### ASP
+ Amazonアソシエイト
+ 株やパソコンに関するもの
