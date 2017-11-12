# real-world
# 構成
- public/
    - index.html
- src/
    - actions/
        - index.js
    - components/
        - Explore.js
        - List.js
        - Repo.js
        - User.js
    - containers/
        - App.js
        - DevTools.js
        - RepoPage.js
        - Root.dev.js
        - Root.js
        - Root.prod.js
        - UserPage.js
    - middleware/
        - api.js
    - reducers/
        - index.js
        - paginate.js
    - store/
        - configureStore.dev.js
        - configureStore.js
        - configureStore.prod.js
    - index.js
- package.json

# Action(actions/index.js)
Actionは、アプリケーションからStoreにデータを送る情報のペイロード
(Storeにおける唯一の情報源)
Actionは、typeプロパティに何かしらのアクション名と状態の値を持つオブジェクトの事
ActionCreatorは、何かしらの引数をとって、Actionを返すもの

### loadUser
ユーザー情報取得

### loadRepo
リポジトリ取得

### loadStarred
ユーザーがスターしたリポジトリ取得

### loadStargazers
スターしたユーザー取得

### resetErrorMessage
エラーメッセージをリセット

# reducers(reducers/index.js)
reducerは、Actionに応じてアプリケーションの状態（state）を
どのように変化させるか指定する役割を持った関数  
※reducerは同じinput(state, action)に応じて一意のoutputを返すようにする

```js
...
const rootReducer = combineReducers({
    entities,
    pagination,
    errorMessage,
})
```
### entities
キャッシュを用いてstate更新
```js
import merge from 'lodash/merge'
...
return merge({}, state, action.response.entities)
```
再帰的にObjectを合体する関数。
配列に対しては、中身がObjectなら同インデックスのObject同士を合体する。  
[lodash.merge](https://qiita.com/minodisk/items/981c074f12d4d1d7b0d5])

### errorMessage
失敗したフェッチについて通知するエラーメッセージを更新

### pagination(reducers/paginate.js) 
index.js
```js
const pagination = combineReducers({
    starredByUser: paginate({
        ...
    }),
    stargazersByRepo: paginate({
        ...
    })
})
```

starredByUser:ユーザがスターしたリポジトリのpaginate
stargazersByRepo:リポジトリをスターしたユーザのpaginate
 
paginate.js
ページネーションのstateの変化を管理
```js
const paginate = ({ types, mapActionToKey }) => {
  ... 
}
state = {
    isFetching: false,
    nextPageUrl: undefined,
    pageCount: 0,
    ids: []
}
```

# middleware(middleware/api.js)
```js
import { normalize, schema } from 'normalizr'
import { camelizeKeys } from 'humps'
```
[normalizr](https://laboradian.com/how-to-use-normalizr/)  
JSONをスキーマに従って正規化する
(すべてのAPIレスポンスは、どのようにネストされていても同じ形状になる)

```js
export default store => next => action => {
    const callAPI = action[CALL_API]
    let { endpoint } = callAPI
    const { schema, types } = callAPI
    ...
    return callApi(endpoint, schema).then(
            response => next(actionWith({
                response,
                type: successType
            })),
            error => next(actionWith({
                type: failureType,
                error: error.message || 'Something bad happened'
            }))
        )
}
```

```js
const API_ROOT = 'https://api.github.com/'
const callApi = (endpoint, schema) => {
    ...
    return fetch(fullUrl)
        .then(response =>
            ...
        )
}
```
fetch apiを使ってgithubのapiをたたく

```js
const fetchUser = login => ({
    [CALL_API]: {
        types: [ USER_REQUEST, USER_SUCCESS, USER_FAILURE ],
        endpoint: `users/${login}`,
        schema: Schemas.USER
    }
})
```
actions/index.js  
actionで定義されている情報
