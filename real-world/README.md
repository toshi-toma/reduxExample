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

# HTML(public/index.html)
```html
<div id="root"></div>
```

# index.js
```js
const store = configureStore()

render(
    <Router>
        <Root store={store} />
    </Router>,
    document.getElementById('root')
)
```
- configureStoreでstoreの作成
- Root Container Componentをrenderする。

# store(store/configureStore.js, configureStore.js, configureStore.prod.js)
```js
if (process.env.NODE_ENV === 'production') {
    module.exports = require('./configureStore.prod')
} else {
    module.exports = require('./configureStore.dev')
}
```
- NODE_ENVに応じて、moduleを切り替えて、storeを作成する

```js
const configureStore = preloadedState => createStore(
    rootReducer,
    preloadedState,
    applyMiddleware(thunk, api)
)
export default configureStore
```

# Action(actions/index.js)
Actionは、アプリケーションからStoreにデータを送る情報のペイロード
(Storeにおける唯一の情報源)
Actionは、typeプロパティに何かしらのアクション名と状態の値を持つオブジェクトの事
ActionCreatorは、何かしらの引数をとって、Actionを返すもの

### loadUser
```js
{
    [CALL_API]: {
        types: [ USER_REQUEST, USER_SUCCESS, USER_FAILURE ],
        endpoint: `users/${login}`,
        schema: Schemas.USER
    }
}
```
- ユーザー情報取得

### loadRepo
```js
{
    [CALL_API]: {
        types: [ REPO_REQUEST, REPO_SUCCESS, REPO_FAILURE ],
        endpoint: `repos/${fullName}`,
        schema: Schemas.REPO
    }
}
```
- リポジトリ取得

### loadStarred
```js
{
    login,
    [CALL_API]: {
        types: [ STARRED_REQUEST, STARRED_SUCCESS, STARRED_FAILURE ],
        endpoint: nextPageUrl,
        schema: Schemas.REPO_ARRAY
    }
}
```
- ユーザーがスターしたリポジトリ取得

### loadStargazers
```js
{
    fullName,
    [CALL_API]: {
        types: [ STARGAZERS_REQUEST, STARGAZERS_SUCCESS, STARGAZERS_FAILURE],
        endpoint: nextPageUrl,
        schema: Schemas.USER_ARRAY
    }
}
```
- リポジトリをスターしたユーザー取得

### resetErrorMessage
```js
{
    type: RESET_ERROR_MESSAGE
}
```
- エラーメッセージをリセット

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
キャッシュを用いてstate更新?(ここわからん)
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
- starredByUser:ユーザがスターしたリポジトリのpaginate
- stargazersByRepo:リポジトリをスターしたユーザのpaginate
 
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

dispatch(fetchUser(login))
```
actions/index.js  
actionで定義されている情報

# containers(containers/)
### Root(Root.js, Root.dev.js, Root.prod.js)
```js
<div>
    <Route path="/" component={App} />
    <Route path="/:login/:name"
           component={RepoPage} />
    <Route path="/:login"
           component={UserPage} />
    <DevTools />
</div>
```

```js
import { Route } from 'react-router-dom'
```
Reactでルーティングをするためのライブラリ。
pathに対応して、componentが切り替わる。

### App(App.js)
```js
<Explore value={inputValue}
                         onChange={this.handleChange} />
<hr />
{this.renderErrorMessage()}
```
- Explore(usernameのinputフォームなど) componentのrender
- errorMessageのrender

### UserPage(UserPage.js)
```js
componentWillMount() {
        loadData(this.props)
}
...
renderRepo([ repo, owner ]) {
    return (
        <Repo
            repo={repo}
            owner={owner}
            key={repo.fullName} />
    )
}
...
<User user={user} />
    <hr />
    <List renderItem={this.renderRepo}
          items={zip(starredRepos, starredRepoOwners)}
          onLoadMoreClick={this.handleLoadMoreClick}
          loadingLabel={`Loading ${login}'s starred...`}
          {...starredPagination} />
```
- User名からスターしたリポジトリ取得
- Repo(１つのリポジトリ) componentのrender
- List(リポジトリのリスト) componentのrender

### RepoPage(RepoPage.js)
```js
componentWillMount() {
    loadData(this.props)
}
...
renderUser(user) {
    return <User user={user} key={user.login} />
}
...
<Repo repo={repo}
          owner={owner} />
    <hr />
    <List renderItem={this.renderUser}
          items={stargazers}
          onLoadMoreClick={this.handleLoadMoreClick}
          loadingLabel={`Loading stargazers of ${name}...`}
          {...stargazersPagination} />
```
- リポジトリからスターしたユーザー取得
- User(１ユーザー) componentのrender
- Repo(１つのリポジトリ) componentのrender
- List(ユーザーのリスト) componentのrender

# components(components/)
### Explore(Explore.js)
Explore(usernameのinputフォームなど) component
### List(List.js)
List(リスト) component
### Repo(Repo.js)
Repo(１つのリポジトリ) component
### User(User.js)
User(１ユーザー) component
