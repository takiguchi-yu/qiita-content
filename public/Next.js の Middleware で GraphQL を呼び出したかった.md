---
title: Next.js の Middleware で GraphQL を呼び出したかった
tags:
  - Next.js
private: false
updated_at: '2025-05-22T22:56:06+09:00'
id: bde312fed7208e23e6ee
organization_url_name: null
slide: false
ignorePublish: false
---

# 前提

- Next.js 15.3
- GraphQL クライアントは `urql` を使用

# やりたかったこと

- App router でルーティングされた動的パスルートに入る前に GraphQL を叩いて、レスポンスを取得したかった。
- 取得したレスポンスを元に、結果が OK であればそのままルーティング、NG であれば 308 リダイレクトしたかった。

# 何があったのか？

- Next.js の [Middleware](https://nextjs.org/docs/app/building-your-application/routing/middleware) から GraphQL は叩けなかった
- 厳密にはローカルでは叩けるが、 `next build` で失敗してしまい、デプロイできなかった

```sh
Failed to compile.

./src/lib/urql/graphql-client.ts
Dynamic Code Evaluation (e. g. 'eval', 'new Function', 'WebAssembly.compile') not allowed in Edge Runtime
Used by persistedExchange
Learn More: https://nextjs.org/docs/messages/edge-dynamic-code-evaluation

The error was caused by importing '@urql/exchange-persisted/dist/urql-exchange-persisted.mjs' in './src/lib/urql/graphql-client.ts'.

Import trace for requested module:
  ./src/lib/urql/graphql-client.ts
  ./src/lib/urql/index.ts
  ./src/features/lib/redirect-info.ts
  ./src/middleware.ts


> Build failed because of webpack errors
 ELIFECYCLE  Command failed with exit code 1.
```

# 何このエラー？

- Next.js は実は 2 つのランタイムを持っていて、1 つは `Node.js`、もう 1 つは `Edge` というもの
- Edge ランタイムでは、軽量で高速である一方、Node.js の機能は一部しか使えない
- Middleware のようなすべてのリクエストに対して実行されるものは、Edge ランタイムの方がパフォーマンスが良いとされている
- **GraphQL クライアントなど、特定のライブラリは、Node.js の機能を使用しているため、Edge ランタイムでは動作しない**

### Edge ランタイムってなに？

https://edge-runtime.vercel.app/

Edge ランタイム自体は Edge コンピューティングでの使用を想定しているようです。  
Edge コンピューティングといえば

- Cloudflare Workers
- Vercel Edge Functions
- AWS Lambda@Edge

ですね。

（ということはつまり、Vercel にデプロイしないと旨みがない・・・？）

# 試したこと

Middleware で GraphQL を叩けない以上、別のアプローチを試してみました。

## 1. page.tsx から GraphQL を叩く

これはダメでした。  
GraphQL を実行して結果が NG だった場合、リダイレクトする必要があったのですが、リダイレクト前のページが先に一瞬レンダリングされてしまい、ユーザーに不自然な体験を与えてしまいました。

- 例えば、 `/users/1` にアクセスしたときに、 `/users/1` のページが一瞬表示されてしまう
- その後、リダイレクトされて `/search` に遷移する

という挙動です。

さらに本来は 308 リダイレクトをしたかったのですが、実際の挙動は `/users/1` に 200 ステータスが返ってきて、 `/search` に 200 ステータスが返ってくるというものでした。
（リダイレクトではなく RSC の fetch が走っていました。）

## 2. layout.tsx から GraphQL を叩く

page.tsx と同じ挙動でした。

# 解決策

現時点では、手詰まりです。

# 朗報

Middleware のランタイムを切り替えることが将来できるようになるようです。

https://nextjs.org/docs/app/building-your-application/routing/middleware

現在は canary バージョンであれば次のように `next.config.ts` に記述することで、Node.js ランタイムを使用することができるようです。

```sh package.json
{
  "dependencies": {
    "next": "canary"
  }
}
```

```js next.config.ts
import type { NextConfig } from 'next'

const nextConfig: NextConfig = {
  experimental: {
    nodeMiddleware: true,
  },
}

export default nextConfig
```

```ts middleware.ts
export const config = {
  runtime: 'nodejs',
};
```

次回の安定版まで待ちます。
