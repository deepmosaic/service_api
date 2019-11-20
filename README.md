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
  - [Get Users](#get_user)
  - [Delete User](#delete_user)
- Invitations
  - [Create Invitation](#create_invitation)
- Organizations
  - [Edit Organization](#edit_organization)
- Organization Members
  - [Edit Organization Member](#edit_organization_member)
  - [Delete Organization Member](#delete_organization_member)
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
firebaseでアカウントの作成が完了したときに呼び出すAPI。roleがadminの場合は同時にorganizationも作成する。organization名は、uuidをベースにシステム側で自動的にIDを割り振る。organization_membersにもレコードを作成する

`POST /v1/service/users`

```
curl https://api.deepmosaic.jp/v1/service/users \
    -H "Authorization: Bearer <ACCESS_TOKEN>" \
    -H "Content-Type: application/json" \
    -d '{ 
        "uid": "phEaSZEblSS5mncrzzGU4GCPxTL2",
        "email": "abc@abc.com",
        "role": "admin",
        "company_name": "いろは株式会社",
        "postal_code": "123-4567",
        "address": "東京都港区白銀高輪1-1-1 パークレジデンス1201"
    }' \
    -X POST
```

### Body params

Parameter name   | Type     | Required | Description
---------------- | -------- | -------- | -----------
uid              | string   | Yes      | firebase uid
email            | string   | Yes      | email
role             | string   | Yes      | for organization_members table ユーザ権限
company_name     | string   | Yes      | for organizations table
postal_code      | string   | Yes      | for organizations table
address          | string   | Yes      | for organizations table

### Response
[Users object](#users_object) is returned.

```
Status: 200 OK

{
    "user": {
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
[Users objects](#users_object) are returned.

```
Status: 200 OK

{
    "users": [
        {
            ...
        },
        {
            ...
        },
        ...
    ]
}
```

---

## <a name="get_user"></a>Get User
自分自身のユーザ情報を取得するときにリクエストするエンドポイント

`GET /v1/service/users/:id`

```
curl https://api.deepmosaic.jp/v1/service/users/:id \
    -H "Authorization: Bearer <ACCESS_TOKEN>"
```

### Response
[Users objects](#users_object) is returned.

```
Status: 200 OK

{
    "user": {
        ...
    } 
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
        "is_app_download": 1
    }' \
    -X PUT
```

### Body params

Parameter name   | Type     | Required | Description
---------------- | -------- | -------- | -----------
email            | string   | No       | email
is_app_download  | number   | No       | is_app_download

### Response
[Users object](#users_object) is returned.

```
Status: 200 OK

{
    "user": {
        ...
    }
}
```

---

## <a name="delete_user"></a>Delete User
firebase側に対してはdisabledを1にする

`DELETE /v1/service/users/:id`

```
curl https://api.deepmosaic.jp/v1/service/users/:id \
    -H "Authorization: Bearer <ACCESS_TOKEN>" \
    -X DELETE
```

### Response
An empty [Users object](#users_object) is returned.

```
Status: 200 OK

{
    "user": {}
}
```

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
        "plan_id": 1,
        "quantity": 1        
    }' \
    -X POST
```

### Body params

Parameter name   | Type     | Required | Description
---------------- | -------- | -------- | -----------
plan_id          | number   | Yes      | the ID of the plan that the customer would like to subscribe to
quantity         | number   | Yes      | The quantity of units for the item

### Response
正常に処理が完了すれば 200 OK が返る

```
Status: 200 OK
```

---

## <a name="edit_license"></a>Edit License
ライセンスを変更するときに呼び出す

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

## <a name="create_invitation"></a>Create Invitation
ユーザを招待し、招待メールを送信する。invitationsテーブルにすでに同じemailが存在する場合は、既存のものを削除扱いにして、新しくレコードを作成する

`POST /v1/service/invitations`

```
curl https://api.deepmosaic.jp/v1/service/invitations \
    -H "Authorization: Bearer <ACCESS_TOKEN>" \
    -H "Content-Type: application/json" \
    -d '{ 
        "email": "deep@mosaic.com",
        "role": "member"
        "organization_id": "20d1a7a0-052f-11ea-8660-d95622a8e37b"
    }' \
    -X POST
```

### Body params

Parameter name   | Type     | Required | Description
---------------- | -------- | -------- | -----------
email            | string   | Yes      | email
role             | string   | Yes      | admin or member
organization_id  | string   | Yes      | 招待した人が所属するorganization_idが指定される

### Response

200 OK is returned.

```
Status: 200 OK

```
---

## <a name="edit_organization"></a>Edit Organization
Organization Memberのroleを変更する

`PUT /v1/service/organizations/:id`

```
curl https://api.deepmosaic.jp/v1/service/organizations/:id \
    -H "Authorization: Bearer <ACCESS_TOKEN>" \
    -H "Content-Type: application/json" \
    -d '{ 
        "company_name": "いろは株式会社",
        "postal_code": "123-4567",
        "address": "東京都港区白銀高輪1-1-1 パークレジデンス1201"
    }' \
    -X PUT
```

### Body params

Parameter name   | Type     | Required | Description
---------------- | -------- | -------- | -----------
company_name     | string   | Yes      | for organizations table
postal_code      | string   | Yes      | for organizations table
address          | string   | Yes      | for organizations table

### Response
200 OK is returned.

```
Status: 200 OK

```

---

## <a name="edit_organization_member"></a>Edit Organization Member
Organization Memberのroleを変更する

`PUT /v1/service/organization_members/:id`

```
curl https://api.deepmosaic.jp/v1/service/organization_members/:id \
    -H "Authorization: Bearer <ACCESS_TOKEN>" \
    -H "Content-Type: application/json" \
    -d '{ 
        "role": "admin"
    }' \
    -X PUT
```

### Body params

Parameter name   | Type     | Required | Description
---------------- | -------- | -------- | -----------
role             | string   | No       | role

### Response
200 OK is returned.

```
Status: 200 OK

```

---

## <a name="delete_organization_member"></a>Delete Organization Member
Organization Memberを削除する

`DELETE /v1/service/organization_members/:id`

```
curl https://api.deepmosaic.jp/v1/service/organization_members/:id \
    -H "Authorization: Bearer <ACCESS_TOKEN>" \
    -X DELETE
```

### Response
200 OK is returned.

```
Status: 200 OK

```

---

## <a name="list_payments"></a>List Payments
Payment一覧情報を返す。roleがadminのみ許可

`GET /v1/service/payments`

```
curl https://api.deepmosaic.jp/v1/service/payments \
    -H "Authorization: Bearer <ACCESS_TOKEN>"
```

### Response
[Payments objects](#payaments_object) are returned.

```
Status: 200 OK

{
    "payments": [
        {
            ...
        },
        {
            ...
        },
        ...
    ]
}
```

---


## <a name="create_project"></a>Create Project
firebaseでアカウントの作成が完了したときに呼び出すAPI。roleがadminの場合は同時にorganizationも作成する。organization名は、uuidをベースにシステム側で自動的にIDを割り振る。organization_membersにもレコードを作成する

`POST /v1/service/projects`

```
curl https://api.deepmosaic.jp/v1/service/projects \
    -H "Authorization: Bearer <ACCESS_TOKEN>" \
    -H "Content-Type: application/json" \
    -d '{
        "name": "my_video.mp4",
        "email": "abc@abc.com",
        "role": "admin",
        "company_name": "いろは株式会社",
        "postal_code": "123-4567",
        "address": "東京都港区白銀高輪1-1-1 パークレジデンス1201"
    }' \
    -X POST
```

### Body params

Parameter name   | Type     | Required | Description
---------------- | -------- | -------- | -----------
uid              | string   | Yes      | firebase uid
email            | string   | Yes      | email
role             | string   | Yes      | for organization_members table ユーザ権限
company_name     | string   | Yes      | for organizations table
postal_code      | string   | Yes      | for organizations table
address          | string   | Yes      | for organizations table

### Response
[projects object](#projects_object) is returned.

```
Status: 200 OK

{
    "project": {
        ...
    }
}
```

---

## <a name="list_projects"></a>List projects
自分が作成したプロジェクト全てを返す

`GET /v1/service/projects`

```
curl https://api.deepmosaic.jp/v1/service/projects \
    -H "Authorization: Bearer <ACCESS_TOKEN>"
```

### Response
[projects objects](#projects_object) are returned.

```
Status: 200 OK

{
    "projects": [
        {
            ...
        },
        {
            ...
        },
        ...
    ]
}
```

---

## <a name="get_project"></a>Get project
projectの情報を取得。roleがmemberの場合は自分が作成したprojectのみ取得可能  
adminの場合は所属しているorganization配下で作成されたproject全てを参照することができる

`GET /v1/service/projects/:id`

```
curl https://api.deepmosaic.jp/v1/service/projects/:id \
    -H "Authorization: Bearer <ACCESS_TOKEN>"
```

### Response
[projects objects](#projects_object) is returned.

```
Status: 200 OK

{
    "project": {
        ...
    } 
```

---

## <a name="edit_project"></a>Edit project
ユーザ情報を編集するエンドポイント

`PUT /v1/service/projects/:id`

```
curl https://api.deepmosaic.jp/v1/service/projects/:id \
    -H "Authorization: Bearer <ACCESS_TOKEN>" \
    -H "Content-Type: application/json" \
    -d '{ 
        "email": "efg@efg.com",
        "is_app_download": 1
    }' \
    -X PUT
```

### Body params

Parameter name   | Type     | Required | Description
---------------- | -------- | -------- | -----------
email            | string   | No       | email
is_app_download  | number   | No       | is_app_download

### Response
変更を加えた後の[projects object](#projects_object)が返る

```
Status: 200 OK

{
    "project": {
        ...
    }
}
```

---

## <a name="delete_project"></a>Delete project
projectを削除する。削除しても検出可能時間が元に戻るわけではない

`DELETE /v1/service/projects/:id`

```
curl https://api.deepmosaic.jp/v1/service/projects/:id \
    -H "Authorization: Bearer <ACCESS_TOKEN>" \
    -X DELETE
```

### Response
An empty [projects object](#projects_object) is returned.

```
Status: 200 OK

{
    "project": {}
}
```

---

## <a name="create_export"></a>Create Export
ビデオの書き出しを記録する

`POST /v1/service/exports`

```
curl https://api.deepmosaic.jp/v1/service/projects \
    -H "Authorization: Bearer <ACCESS_TOKEN>" \
    -H "Content-Type: application/json" \
    -d '{
        "is_timerecord": 1
    }' \
    -X POST
```

### Body params

Parameter name   | Type     | Required | Description
---------------- | -------- | -------- | -----------
uid              | string   | Yes      | firebase uid
email            | string   | Yes      | email
role             | string   | Yes      | for organization_members table ユーザ権限
company_name     | string   | Yes      | for organizations table
postal_code      | string   | Yes      | for organizations table
is_timerecord    | number   | Yes      | タイムレコードを

### Response
[projects object](#projects_object) is returned.

```
Status: 200 OK

{
    "project": {
        ...
    }
}
```

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


# JSON Object


## <a name="users_object"></a>Users object
```
{
    "id": "20d1a7a0-052f-11ea-8660-d95622a8e37b",
    "created_at": "2019-12-19 14:00:00",
    "uid": "4QSIV3gybIgbVmeSJ2RPx8AhHHW2",
    "email": "deepmosaic@deepmosaic.com",
    "organization_id": "21391a52-052f-11ea-8660-d95622a8e37b",
    "company_name": "いろは株式会社",
    "postal_code": "142-0003",
    "role": "member",
    "last_signin_at": "2019-12-19 14:00:00",
    "is_app_download": 1,
    "is_signup": 0,
    "video_length": 360
}
```

Parameter name        | JSON Type | MYSQL Type   | Null | Description
--------------------- | --------- | ------------ | ---- | ------------
id                    | string    | BINARY(16)   | No   | Primary key
created_at            | string    | DATETIME     | No   | 作成日時


## <a name="payments_object"></a>Payments object
```
{
    "id": "20d1a7a0-052f-11ea-8660-d95622a8e37b",
    "created_at": "2019-12-19 14:00:00",
    
}
```

Parameter name        | JSON Type | MYSQL Type   | Null | Description
--------------------- | --------- | ------------ | ---- | ------------
id                    | string    | BINARY(16)   | No   | Primary key
created_at            | string    | DATETIME     | No   | 作成日時