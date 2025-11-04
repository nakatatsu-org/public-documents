# Coding Standard for backend

## プロジェクトのディレクトリ設計

本プロジェクトでは必ず下記のディレクトリ設計に従うこと。

```
cmd/
  server/
    main.go         // サーバ起動
    api_handler.go  // APIハンドラー統合
    wire_gen.go     // DI
internal/
  api/
    gen.go  　      // oapi-codegen で生成
  <module_name>/         // e.g., “ダウンロード”モジュールの場合download/
    handler.go     // HTTPハンドラ
    usecase.go     // ビジネスロジック（サービス層的役割）
    repository.go  // 永続化
    entity.go      // ドメインモデル
    README.md      // 設計を記載
    mock.go        // モック用のコードを入れる
    …
  middleware/      // 認証・ロギングなど共通処理があれば入れる
  utility/         // 汎用ユーティリティ
```


### Logging

* Always use **`slog`** for logging — other logging methods are prohibited.
* use Costructor injection, and use medhot "XXXContext".

```go
import (
    "context"
    "log/slog"
    "github.com/gin-gonic/gin"
)

type (
	// Handler handles ordering-related HTTP requests.
	Handler struct {
		logger  *slog.Logger
	}
)

func (h *Handler) example(c *gin.Context) {
		h.logger.DebugContext(context.Background(), "handling example request")
}
```

- Alway log important user's action. e.g., payment, add point, download goods, etc. 

### Input Validation & Domain Integrity

* Input validation (i.e., request validation) is declared on the TypeSpec side, so no rules are coded on the Golang side, even though Gin uses `ShouldBindJSON`.
* Whether an entity conforms to domain rules is checked within the Entity (or within a Value Object).


### Data Consistency & Transactions

* When an update spans multiple locations, aim for **“all succeed or all fail.”**
* If technical constraints (e.g., cross-service transactions) make that difficult, follow the **Saga pattern**:

  * Roll back with a **compensating transaction** where possible.
  * If rollback is impossible, proceed to completion but ensure the user is not harmed.
  * Save the overall plan first, record each intermediate step, and delegate exceptional edge cases to human operations rather than forcing the system to anticipate everything.


### Handler Layer

* A handler should contain **only the code** that mediates between the caller and the **UseCase**.

  * In other words, place **no other logic** in handlers.


### DynamoDB

#### 重複防止と楽観ロック

- 新規作成時
  - `ConditionExpression: attribute_not_exists(PK)` を指定し、既存レコードの上書きを防止します。

- 更新時
  - 数値型の `Version` 属性を導入し、`ConditionExpression: Version = :current` により楽観的ロックを実現します。
  - 更新成功時は `SET Version = Version + 1` でバージョンを自動インクリメントします。
- 競合時の挙動
  - 条件不一致エラー発生時は `ErrConflict` を返却し、アプリケーション層で再試行またはユーザー通知を行います。

#### UpdateItem の利用

* `Update` 操作は `PutItem` ではなく `UpdateItem` を用いて差分更新とします。
* `SET`／`REMOVE` 句を組み合わせ、変更がある属性のみ書き込み・削除します。
* 例：`SET UpdatedAt = :now, TotalPrice = :price REMOVE DiscountCode`

#### 「もっと見る」アコーディオンによるページネーション

* 複数件取得する関数は最大 `pageSize` 件を返却し、追加データの有無を `LastEvaluatedKey` で判定できるようにします。
* LastEvaluatedKeyが渡された場合は、そのキーから `pageSize` 件を取得します

## エラーハンドリング

- エラーを起こす可能性のある関数は常に `(..., error)` を戻り値に含めていること。`panic` の利用は禁止する。
- エラー時には必ずslogでエラーメッセージとスタックトレースを出力していること。
- ハンドラー層はエラー発生時に500を返していること。ただしロジック上（ユーザーIDがないなど）必要と認められる場合に限り4XXを返すことも許可される。

## ページネーション

- どこまで進んだかを管理するキーは`pageToken` を使っていること

## 認証・認可

- JWT をヘッダー `Authorization: Bearer <token>` で受け取り、ミドルウェアで検証後に `context` にユーザー情報が格納されていること。権限不足時は 403 を返すこと。


## テスト

- UseCase／Service 層のカバレッジは 80% 以上であること。DynamoDB 依存はlocalstackを用いること。
- 必ず[goquality](https://github.com/nakatatsu/goquality) を使用してコードの品質チェックを行うこと。利用例はscripts/run_tool_tests.shを参照せよ
- 各エンドポイントのテストにおいてインジェクション攻撃が成功しないことを確認していること

## 命名規則

- インターフェイスは`Interface`を必ずサフィックスに利用します。

## 禁止事項

- ユニットテストでStripe等の外部サービスAPIをしてはいけない
