# Service API

Deepmosaicのバックエンドを担当するコンポーネント

## 目次
- [はじめに](https://github.com/LancersDevTeam/spam_api/wiki)


## API List
- Connection
  - [Ping](#ping)
- Users
  - [Create User](#create_user)
  - [Edit User](#edit_user)
  - [List Users](#list_users)
  - [Delete User](#delete_user)
- Invitations
  - [Create Invitation](#create_invitation)
  - [Edit Invitation](#edit_invitation)
- Licenses
  - [Create License](#create_license)
  - [Edit License](#edit_license)
- Payments
  - [List Payments](#list_payments)
- Projects
  - [Create Project](#create_project)
  - [Get Project](#get_project)
  - [List Projects](#list_projects)
  - [Edit Project](#edit_project)
  - [Delete Project](#delete_project)
- Exports
  - [Create Export](#create_export)
- Other
  - [HTTP Status Code](#http_status_code)
  - [Error](#error)
- Table Schema
  - [message_spams](#message_spams)
  - [spam_publics](#spam_publics)
- Object
  - [Message spams object](#message_spams_object)
  - [Message spams object for admin](#message_spams_object_admin)

---
## <a name="ping"></a>Ping
サーバのヘルスチェックを行う

`GET /v1/service/connection/ping`

```
curl https://api.deepmosaic.jp/v1/service/connection/ping
```

### Response
リクエストが成功すると、**JSON** 形式でレスポンスデータが返る。

```
Status: 200 OK

{
    "ping": "pong"
}
```

Field name    | Type          | Description
------------- | ------------- | ---------------
ping          | string        | `pong` is always returned.

---

## <a name="create_user"></a>Create User
firebaseでアカウントの作成が完了したときに呼び出すAPI。管理者ユーザを作成する。roleがadminの場合は同時にorganizationも作成する。organization名は、uuidをベースにシステム側で自動的にIDを割り振る

`POST /v1/service/users`

```
curl https://api.deepmosaic.jp/v1/service/users \
    -H "Authorization: Bearer <ACCESS_TOKEN>" \
    -H "Content-Type: application/json" \
    -d '{ 
        "role": "admin",
        "email": "abc@abc.com",
        "phone_number": "03-1234-5678",
        "company_name": "いろは株式会社"
    }' \
    -X POST
```

### Body params

Parameter name   | Type     | Required | Description
---------------- | -------- | -------- | -----------
role             | string   | Yes      | message id
email            | string   | Yes      | message id
phone_number     | number   | Yes      | message id
company_name     | string   | Yes      | message id

### Response
正常に処理が完了すれば 200 OK が返る

```
Status: 200 OK
```

---

## <a name="edit_user"></a>Edit User
ユーザ情報を編集するエンドポイント

`PUT /v1/service/users/:id`

```
curl https://api.deepmosaic.jp/v1/service/users/:id \
    -H "Authorization: Bearer <ACCESS_TOKEN>" \
    -H "Content-Type: application/json" \
    -d '{ 
        "email": "efg@efg.com",
        "phone_number": "03-8765-4321"
        "display_name": "ディープモザイク",
        "photo_url": "",
        "is_download": 1
    }' \
    -X PUT
```

### Body params

Parameter name   | Type     | Required | Description
---------------- | -------- | -------- | -----------
email            | string   | No       | email
phone_number     | string   | No       | phone_number
display_name     | string   | No       | display_name
photo_url        | string   | No       | photo_url
is_download      | number   | No       | is_download

### Response
Users object are returned.

```
Status: 200 OK

{
    "id": "886313e1-3b8a-5372-9b90-0c9aee199e5d",
    "self_link": "https://api2.lancers.jp/v1/spam/messages?page=3&feedback_from_admin=0",
    "users": {
        ...
    }
}
```

---

## <a name="list_users"></a>List Users
ユーザのroleがadminの場合は、organizationに属するユーザ全ての情報を返す

`GET /v1/service/users`

```
curl https://api.deepmosaic.jp/v1/service/users \
    -H "Authorization: Bearer <ACCESS_TOKEN>"
```

### Response
[Users for management object](#users_for_management_object) are returned.

```
Status: 200 OK

{
    "users": [
        {
            ...
        }
    ]
}
```

Field name    | Type          | Description
------------- | ------------- | ---------------
user_id       | string        |
email         | string        | The unique name for this response.
role          | string        | A URL to re-request this resource.
created_at    | string        | message_spamsテーブルにおける`feedback_from_admin=0`の全レコード数
signin_at     | number        |
video_length  | number        |
status        | string        | [Users for management object](#users_for_management_object)

---



## <a name="create_license"></a>Create License
stripeのチェックアウト処理が完了したときにリダイレクトするエンドポイント。success_urlに指定するエンドポイント
webhookとしてpostされる

`POST /v1/service/licenses`

```
curl https://api.deepmosaic.jp/v1/service/licenses \
    -H "Authorization: Bearer <ACCESS_TOKEN>" \
    -H "Content-Type: application/json" \
    -d '{ 
        "length": 720,
        "price": 14800
    }' \
    -X POST
```

### Body params

Parameter name   | Type     | Required | Description
---------------- | -------- | -------- | -----------
length           | number   | Yes      | message id
price            | number   | Yes      | message id

### Response
正常に処理が完了すれば 200 OK が返る

```
Status: 200 OK
```

---

## <a name="edit_license"></a>Edit License
stripeのチェックアウト処理が完了したときに呼び出すエンドポイント

`POST /v1/service/licenses/:id`

```
curl https://api.deepmosaic.jp/v1/service/licenses/:id \
    -H "Authorization: Bearer <ACCESS_TOKEN>" \
    -H "Content-Type: application/json" \
    -d '{ 
        "cancel_applied_at": "2020-04-01 14:12:34"
    }' \
    -X PUT
```

### Body params

Parameter name   | Type     | Required | Description
---------------- | -------- | -------- | -----------
message_id       | number   | Yes      | message id

### Response
正常に処理が完了すれば 200 OK が返る

```
Status: 200 OK
```

---

## <a name="list_messages"></a>List Messages
idの降順にmessage_spamsレコードを取得する。1ページあたり50個の要素を取得する。

`GET /v1/spam/messages`

```
# 旧メッセージページ・スマホページを表示させるときにmessage_spamsオブジェクトを取得する例
curl https://api.deepmosaic.jp/v1/service/messages?board_id=1234567

# 管理画面で表示させるmessage_spamsオブジェクトを取得する例
curl https://api.deepmosaic.jp/v1/service/messages?page=3&feedback_from_admin=0
```

### Parameters

Parameter name | Type          | Required | Description
-------------- | ------------- | -------- | -----------
board_id       | string        | No       | 指定したboard_idのmessage_spamsオブジェクトのみを取得。
page           | number        | No       | page番号を指定する。pageを指定しない場合は1ページ目が返る。
feedback_from_admin | number   | No       | 指定したfeedback_from_adminのmessage_spamsオブジェクトのみを取得する。管理画面に表示させるmessage_spamsオブジェクトを取得する場合は `0` を指定する。

### Response
Message spams object are returned.

```
Status: 200 OK

{
    "id": "886313e1-3b8a-5372-9b90-0c9aee199e5d",
    "self_link": "https://api.deepmosaic.jp/v1/service/messages?page=3&feedback_from_admin=0",
    "count": 89,
    "previous": "https://api.deepmosaic.jp/v1/service/messages?page=2&feedback_from_admin=0",
    "next": "https://api.deepmosaic.jp/v1/service/messages?page=1&feedback_from_admin=0",
    "message_spams": [
        {
            ...
        }
    ]
}
```


Field name    | Type          | Description
------------- | ------------- | ---------------
id            | string        | The unique name for this response.
self_link     | string        | A URL to re-request this resource.
count         | number        | message_spamsテーブルにおける`feedback_from_admin=0`の全レコード数
previous      | string or null | 1つ前のページのリンクがセットされる。1ページ目の場合はnullがセットされる。
next          | string or null | さらにデータがある場合は次のページのリンクがセットされる。現在のページ以上のデータが存在しない場合はnullがセットされる。
message_spams  | array         | [Message spams object](#message_spams_object)

---
## <a name="edit_message"></a>Edit Message
特定のmessage_spamオブジェクトを編集する。spam_apiによって予測された違反判定の結果を管理画面から編集する際に当エンドポイントへPUTリクエストする。

`PUT /v1/spam/messages/:message_spam_id`

```
curl https://api.deepmosaic.jp/v1/service/messages/123456789 \
    -H "Content-Type: application/json" \
    -d '{ "feedback_from_admin": 2 }' \
    -X PUT
```

### Parameters

Parameter name | Type          | Required | Description
-------------- | ------------- | -------- | -----------
message_spam_id | number       | Yes      | message spam id

### Body params

Parameter name   | Type     | Required | Description
---------------- | -------- | -------- | -----------
feedback_from_admin | number | Yes     | 「公開」であれば `2`,「非公開」であれば `1` をセットする。

### Response
Message spams objects for admin are returned.

```
Status: 200 OK

{
    "id": "886313e1-3b8a-5372-9b90-0c9aee199e5d",
    "self_link": "https://api.deepmosaic.jp/v1/service/messages/123456789",
    "message_spams": {
        ...
    }
}
```


Field name    | Type          | Description
------------- | ------------- | ---------------
id            | string        | The unique name for this response.
self_link     | string        | A URL to re-request this resource.
message_spams  | object       | [Message spams object for admin](#message_spams_object_admin) 便宜上、Field nameはmessage_spamsで統一している。

---

## <a name="add_public"></a>Add Public
MLM Checkerとして公開しているサービスで、テキストデータを受け取るためのエンドポイント

`POST /v1/spam/publics`

```
curl https://api.deepmosaic.jp/v1/service/publics \
    -H "Content-Type: application/json" \
    -d '{ "description": 'xxxxxxxx' }' \
    -X POST
```

### Body params

Parameter name   | Type     | Required | Description
---------------- | -------- | -------- | -----------
description      | string   | Yes      | テキストデータ

### Response
正常に処理が完了すれば 200 OK とともにpublic objectが返る。

Field name    | Type          | Description
------------- | ------------- | ---------------
id            | string        | The unique name for this response.
self_link     | string        | A URL to re-request this resource.
public        | object        | [public object](#public_object)

```
Status: 200 OK

{
    "id": "886313e1-3b8a-5372-9b90-0c9aee199e5d",
    "self_link": "https://api.deepmosaic.jp/v1/service/publics/12345",
    "public": {
        ...
    }
}

```

---

## <a name="edit_public"></a>Edit Public
ユーザからのフィードバックを受け付けるエンドポイント

`PUT /v1/spam/publics/:public_id`

```
curl https://api.deepmosaic.jp/v1/service/publics/12345 \
    -H "Content-Type: application/json" \
    -d '{ "feedback_from_user": 2 }' \
    -X PUT
```

### Parameters

Parameter name | Type          | Required | Description
-------------- | ------------- | -------- | -----------
public_id      | number        | Yes      | public id

### Body params

Parameter name   | Type     | Required | Description
---------------- | -------- | -------- | -----------
feedback_from_admin | number | Yes     | spamだと判断した場合: 1, spamではないと判断した場合: 2

### Response
public object edited is returned.

```
Status: 200 OK

{
    "id": "886313e1-3b8a-5372-9b90-0c9aee199e5d",
    "self_link": "https://api.deepmosaic.jp/v1/service/publics/12345",
    "public": {
        ...
    }
}
```


Field name    | Type          | Description
------------- | ------------- | ---------------
id            | string        | The unique name for this response.
self_link     | string        | A URL to re-request this resource.
public        | object        | [public object](#public_object)

---

# <a name="http_status_code"></a>HTTP Status Code

Status | Phrase                | Message
------ | --------------------- | -----------
200    | OK                    | リクエストは正常に処理された
201    | Created               | リソースが正常に作成された
204    | No Content            | リクエストは正常に処理されたが、返す値がない
400	   | Bad Request           | パラメータ値が正しくない
401    | Unauthorized          | 認証されていない
403    | Forbidden             | リソースへのアクセスが禁止されている
404    | Not Found             | リソースが存在しない
405    | Method Not Allowed    | 期待されていないHTTPメソッドが使用された
408    | Request Timeout       | 10秒以上経過しても処理を返せなかった
409    | Conflict              | 既存のリソースと競合するので処理を完了できない
500    | Internal Server Error | サーバー内部エラー

---
## <a name="error"></a>Error

### Error response

```
{
    "status": number,
    "code": string,
    "message": string,
    "errors": [
        {
            "field": string,
            "code": string
        }
    ]
}
```

Field name            | Type     | Description
--------------------- | -------- | ---------------
status                | number   | An HTTP status code value
code                  | string   | [A status code](#status_code)
message               | string   | エラーの内容
errors                | array    | [Error detail](#error_detail) validation errorの場合はerrorsフィールドが付与される


### <a name="status_code"></a>Status Code
status                | code                  |  Description
--------------------- | --------------------- | -----------
400                   | invalid_format        | application/jsonでアクセスしていない
400                   | invalid_parameter     | パラメータ値が正しくない
401                   | access_token_expired  | アクセストークンの有効期限が切れている
401                   | access_token_invalid  | アクセストークンが不正である
403                   | forbidden             | リソースにアクセスすることが禁止されている
404                   | not_found             | 存在しないエンドポイントにアクセスした
409                   | conflict              | 既存のリソースと競合するので処理を完了できない
500                   | internal_server_error | サーバが停止している

### <a name="error_detail"></a>errors
Parameter name        | Type     | Description
--------------------- | -------- | -----------
field                 | string   | フィールド名
code                  | string   | [Validation error code](#validation_error_code)

### <a name="validation_error_code"></a>Validation error code
Validation error code | Description
--------------------- | -----------
missing               | This means a resource does not exist.
missing_field         | This means a required field on a resource has not been set.
invalid               | This means the formatting of a field is invalid. The documentation for that resource should be able to give you more specific information.
already_exists        | This means another resource has the same value as this field. This can happen in resources that must have some unique key (such as Label names).

# Spam Table Schema

## <a name="message_spams"></a>message_spams

Parameter name        | JSON Type | MYSQL Type   | Null | Description
--------------------- | --------- | ------------ | ---- | ------------
id                    | number    | INT(11)      | No   | Primary key
created               | string    | DATETIME     | No   | 作成日時
modified              | string    | DATETIME     | No   | 最終更新日時
board_id              | number    | INT(11)      | No   | board_id
message_id            | number    | INT(11)      | No   | message_id
score                 | string    | FLOAT        | Yes  | spamである可能性。数値が大きいほどspamである可能性が高い
predict               | number    | TINYINT(2)   | No   | 判定結果 1: spam, 2: not spam
feedback_from_admin   | number    | TINYINT(2)   | No   | 値が入っていない状態を0とする。管理者がspamだと判断した場合: 1, 管理者がspamではないと判断した場合: 2
feedback_from_user    | number    | TINYINT(2)   | No   | 値が入っていない状態を0とする。ユーザがspamだと判断した場合: 1
biz_filter            | object    | TEXT         | Yes  | ビジネスフィルター。JSON形式でデータを保存する。


## <a name="spam_publics"></a>spam_publics

Parameter name        | JSON Type | MYSQL Type   | Null | Description
--------------------- | --------- | ------------ | ---- | ------------
id                    | number    | INT(11)      | No   | Primary key
created               | string    | DATETIME     | No   | 作成日時
modified              | string    | DATETIME     | No   | 最終更新日時
description           | string    | TEXT         | No   | 送信された文章データ
score                 | string    | FLOAT        | No   | spamである可能性。数値が大きいほどspamである可能性が高い
feedback_from_admin   | number    | TINYINT(2)   | No   | 値が入っていない状態を0とする。管理者がspamだと判断した場合: 1, 管理者がspamではないと判断した場合: 2
feedback_from_user    | number    | TINYINT(2)   | No   | 値が入っていない状態を0とする。ユーザがspamだと判断した場合: 1

### Index

```
create index feedback_from_admin_index on spam_publics(feedback_from_admin);
create index feedback_from_user_index on spam_publics(feedback_from_user);
```

## <a name="message_spams_object"></a>Message spams object
```
{
    "4321": {
        "id": 5647,
        "created": "2017-12-19 14:00:00",
        "board_id": 8765,
        "message_id": 4321,
        "description": "突然のメッセージ失礼します。ネット関連の仕事をしていて...安田",
        "send_user_id": 1234,
        "send_user_nickname": 'yasuda',
        "spam": 2
    }
}
```

message_idを各オブジェクトのKeyとする。Keyの型はstring型である。JSONはKeyにstring型しか使えない。

Parameter name        | JSON Type | Description
--------------------- | --------- | -------------------------------------
id                    | number    | message spam id
created               | string    | message_spamsレコードが作成された日時
board_id              | number    | messageが属するboard id
message_id            | number    | message id
description           | string    | messageの本文
send_user_id          | number    | messageを送ったユーザのid
send_user_nickname    | number    | messageを送ったユーザのnickname
spam                  | number    | messageのspamに関する状態。`spam: 0` spamではない。`spam: 1` spamだが管理者によるチェックは未実施。`spam: 2` 管理者によるチェックによってspamであると判断されたメッセージ。

## <a name="message_spams_object_admin"></a>Message spams object for admin
管理画面からmessage_spamsオブジェクトを編集した時のレスポンスデータ用
```
{
    "4321": {
        "id": 5647,
        "created": "2017-12-19 14:00:00",
        "board_id": 8765,
        "board_title": "rere_5ars yasuda",
        "message_id": 4321,
        "description": "突然のメッセージ失礼します。ネット関連の仕事をしていて...安田",
        "send_user_id": 1234,
        "send_user_nickname": "yasuda",
        "send_user_status": "blacked",
        "receive_user_id": [1234, 5678, 9123]
        "predict": 1,
        "feedback_from_admin": 0,
        "feedback_from_user": 0
    }
}
```

message_idをオブジェクトのKeyとする。Keyの型はstring型である。JSONはKeyにstring型しか使えない。

Parameter name        | JSON Type | Description
--------------------- | --------- | -------------------------------------
id                    | number    | message spam id
created               | string    | message_spamsレコードが作成された日時
board_id              | number    | messageが属するboard id
board_title           | string    | boardのタイトル
message_id            | number    | message id
description           | string    | messageの本文
send_user_id          | number    | messageを送ったユーザのid
send_user_nickname    | number    | messageを送ったユーザのnickname
send_user_status      | number    | messageを送ったユーザのstatus
receive_user_id       | array     | message受信者のid
predict               | number    | 判定結果 1: spam, 2: not spam
feedback_from_admin   | number    | 値が入っていない状態を0とする。管理者がspamだと判断した場合: 1, 管理者がspamではないと判断した場合: 2
feedback_from_user    | number    | 値が入っていない状態を0とする。ユーザがspamだと判断した場合: 1


## <a name="public_object"></a>Public object
```
{
    "id": 5647,
    "created": "2018-01-25 14:00:00",
    "modified": "2018-01-25 14:00:00",
    "score": 2.346763,
    "description": "突然のメッセージ失礼します。ネット関連の仕事をしていて...安田",
    "feedback_from_admin": 1234,
    "feedback_from_user": 'yasuda'    
}
```

message_idを各オブジェクトのKeyとする。Keyの型はstring型である。JSONはKeyにstring型しか使えない。

Parameter name        | JSON Type | Description
--------------------- | --------- | -------------------------------------
id                    | number    | public object id
created               | string    | 作成日時
modified              | string    | 最終更新日時
score                 | string    | spamである可能性。数値が大きいほどspamである可能性が高い
description           | string    | 送信されたテキスト
feedback_from_admin   | number    | 値が入っていない状態を0とする。管理者がspamだと判断した場合: 1, 管理者がspamではないと判断した場合: 2
feedback_from_user    | number    | 値が入っていない状態を0とする。ユーザがspamだと判断した場合: 1


## Service Level Agreement
1. サーバーの月間稼働率は95.0%以上である。
2. 事前に告知した状態で、または、緊急でメンテナスを実施する可能性がある。
