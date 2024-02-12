---
title: "プラットフォームに依存しないJavaScriptコードに変換するunenvについて"
emoji: "🕊️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["nuxtjs", "nuxt3", "unjs"]
published: true
publication_name: "comm_vue_nuxt"
---

## この記事について

この記事は、UnJSの主要なライブラリを調査していくシリーズ記事の1つになります。
シリーズ記事の概要や今後公開される予定の記事の確認は[こちらの記事](./introducing-unjs)を参照してください。今回はunenvというライブラリについて紹介します。

## unenvの特徴

unenv([unjs/unenv](https://github.com/unjs/unenv))は、ブラウザ、Node.js、Workers等の特定の実行環境に依存しないユニバーサルなJavaScriptコードに変換するためのユーティリティライブラリです。RollupやWebpack等のバンドルツールと組み合わせてビルドした成果物

unenvの主要な特徴として、次のものがあります:

* ブラウザ、Node.js、Workers等の様々なJavaScript実行環境で動作させることが可能な機能を提供
* 主要なJavaScript実行環境用にプリセットとしてRollupやWebpack等のバンドラの設定を用意しているため、開発環境に導入することが容易
* JavaScript実行環境の依存をなくすために、モックやポリフィルを提供

## ユースケース

具体的にどのようなことができるのか
unenvは、UnJSのいくつかのライブラリで実際に使われているため紹介します。

### H3のHTTPリクエストとレスポンス

H3はサーバーレス環境、エッジ環境、Node.js/Bun/Deno等の様々なJavaScript実行環境で動作する高速なHTTPサーバーです。Node.jsの`http`モジュールが提供している`IncomingMessage`や`OutgoingMessage`と、Node.js以外のFetch APIに準拠している実行環境が提供している`Request`と`Response`は互換性がありません。Node.js向けに実装した
H3はこれを解消するためにアダプタという機能を提供していて、アダプタによってNode.jsとそれ以外の実行環境の差を吸収しています。
具体的にはH3サーバーに登録したイベントハンドラ内で処理されるリクエストとレスポンスは

#### Node.jsの処理

H3を使ったHTTPサーバーの基本的な実装は次のようになります。

```javascript
import { createServer } from "node:http";
import { createApp, eventHandler, toNodeListener } from "h3";

const app = createApp();

app.use("/", eventHandler(() => "Hello world!"));

createServer(toNodeListener(app)).listen(8080);
```

`app`インスタンスを作成し、`toNodeListener`に渡します。この時に`toNodeListener`内で次のような処理が行われます。

```javascript
function toNodeListener(app) {
  return async function (req, res) {
    const event = createEvent(req, res);

    await app.handler(event);
  }
}
```

`toNodeListener`内の`req`と`res`は`http`モジュールで初期化された`IncomingMessage`と`OutgoingMessage`のオブジェクトになります。このオブジェクトを基に`event`オブジェクトを作成し、イベントハンドラに渡されます。
`IncomingMessage`と`OutgoingMessage`のオブジェクトを`console`等でデバッグしたことがある方ならご存知だと思いますが、これらは凄くデータサイズが大きいオブジェクトになります。

#### Fetch APIに準拠している実行環境の処理

```javascript
import { createApp, eventHandler, toWebHandler } from "h3";

const app = createApp();

app.use("/", eventHandler(() => "Hello world!"));

const handler = toWebHandler(app);

export default {
  fetch: (req) => {
    return handler(req);
  },
};
```

```javascript
import { IncomingMessage } from "unenv/runtime/node/http/_request";
import { ServerResponse } from "unenv/runtime/node/http/_response";

async function toWebHandler(app, req) {
  const nodeReq = new IncomingMessage();
  const nodeRes = new ServerResponse(nodeReq);

  const event = createEvent(req, res);

  await app.handler(event);
}
```

Fetch APIに準拠している実行環境

:::message
H3の詳細については、同じシリーズ記事の「[H3 (unjs/h3)について](./introducing-unjs-h3)」という記事で紹介しています
:::

### Nitro

## プリセット

プリセットは特定の実行環境向けにunenvで用意しているコンフィグになります。

```typescript
interface Environment {
  alias: { [key: string]: string };
  inject: { [key: string]: string | string[] };
  polyfill: string[];
  external: string[];
}

interface Preset extends Partial<Environment> {}
```

* alias
* inject
* polyfill
* external

### プリセット一覧

#### node

`node`プリセットは、Node.js向けの設定で、

#### nodeless

#### deno

#### cloudflare

`cloudflare`プリセットは、[Cloudflare Workers](https://developers.cloudflare.com/workers/)向けの設定です。

#### vercel

`vercel`プリセットは、Vercelの[Edge Runtime](https://vercel.com/docs/functions/runtimes/edge-runtime)向けの設定です。`Edge Runtime`でJavaScriptを実行すると`async_hooks`、`events`、`buffer`等のコアモジュールは`node:`プリフィックスをつけてもつけなくてもインポートすることができます[^1]。

[^1]: [Compatible Node.js modules - Edge Runtime](https://vercel.com/docs/functions/runtimes/edge-runtime#compatible-node.js-modules)

### プリセットの作り方

## モック/ポリフィル

### node:http

### node:path

### node-fetch

### globalThis

## 使い方

```javascript
import { env } from "unenv";

const { alias, inject, polyfill, external } = env({}, {}, {});
```

## 調査しながら思ったこと

### 対象となっている実行環境以外でも使うことができるのか

Google App EngineとかAWS Lambdaとか
unenvがサポートする ≒ Nuxtを動かすことが可能な実行環境が増えるということを意味する

## 参考

* [Introducing unjs - VueConf US 2023](https://www.youtube.com/watch?v=jc-42ZtaD_k)
