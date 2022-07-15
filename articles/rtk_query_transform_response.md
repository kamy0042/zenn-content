---
title: "RTK Query で取得したデータをキャッシュ前に操作したい"
emoji: "🔥"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ['react','redux','frontend']
published: true
---

# 結論
`transformResponse` を使う
https://redux-toolkit.js.org/rtk-query/usage/customizing-queries#customizing-query-responses-with-transformresponse

# transformResponse とは
`useQuery` や `useMutation` のレスポンスを操作するための機能。
レスポンスが返ってきたタイミングで呼び出され、操作後の返り値をキャッシュデータとして用いることができる。

デフォルトだと取得した値をそのまま返している。（各引数の詳細については[こちら](https://redux-toolkit.js.org/rtk-query/usage/customizing-queries#customizing-query-responses-with-transformresponse)を参照）
```js
function defaultTransformResponse(
  baseQueryReturnValue: unknown,
  meta: unknown,
  arg: unknown
) {
  return baseQueryReturnValue
}
```
利用する際は、 `endpoints` の各処理に渡すオブジェクト内で下記のように書けば　OK
```js
export const someApi = createApi({
  reducerPath: 'someApi',
  baseQuery: fetchBaseQuery({ baseUrl: 'https://someapi.co/api/v2/' }),
  endpoints: (builder) => ({
    getItemById: builder.query<someResponse, string>({
      query: (id) => `item/${id}`,
      transformResponse:(response:someResponse) => {
        // 処理は適当
        return response.toUpperCase();
      }
    }),
  }),
})
```

# 利用シーン
### データの正規化
Redux のドキュメントではデータを正規化して store に格納することを推奨している。また、Redux Toolkit では正規化のための `createEntityAdaptor` という API も準備されている。
しかし、RTK Query を使ってキャッシュデータを管理するのであれば [正規化する必要性が低くなる](https://redux-toolkit.js.org/rtk-query/usage/cache-behavior#no-normalized-or-de-duplicated-cache)。
とはいえ、「RTK Query を利用しているけれどもデータを正規化しておきたい」という需要もあるかもしれない。その場合は `transformResponse` 内で `createEntityAdaptor` を用いることができる。

### 深くネストしたデータを取り出す
[公式](https://redux-toolkit.js.org/rtk-query/usage/customizing-queries#customizing-query-responses-with-transformresponse)より。
深くネストした状態から必要なデータのみ取り出すことができる
```js
transformResponse: (response, meta, arg) =>
  response.some.deeply.nested.collection
```

### meta や arg を元にレスポンスを変換する
こちらも[公式](https://redux-toolkit.js.org/rtk-query/usage/customizing-queries#customizing-query-responses-with-transformresponse)より。
詳細は上記リンクを参照。
```js
transformResponse: (response: { sideA: Tracks; sideB: Tracks }, meta, arg) => {
  if (meta?.coinFlip === 'heads') {
    return response.sideA
  }
  return response.sideB
}
```
```js
transformResponse: (response: Posts, meta, arg) => {
  return {
    originalArg: arg,
    data: response,
  }
}
```
### 取得したオブジェクトのキーをキャメルケースに変換する
読んで字のごとく。

# 個人的に嬉しいポイント
Redux にも通じることだが、「処理を書く場所が決められている」ことにより迷いが無くなる点は嬉しい。
特にチームで開発を進める場合、「個々人が適当と思う場所に処理を書く」という事態を防ぎやすくなるのはありがたい。（もちろん完全に防げるわけではないが）



