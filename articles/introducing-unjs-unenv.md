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

unenv([unjs/unenv](https://github.com/unjs/unenv))は、ブラウザ、Node.js/Deno/Bun、Workers等の特定の実行環境に依存しないユニバーサルなJavaScriptコードに変換するためのユーティリティライブラリです。基本的にはRollupやWebpack等のビルドツールと組み合わせて使い、実行環境に依存しないJavaScriptコードを生成することができます。unenvと同じくUnJSプロジェクトから提供されているH3やNitroは内部的にunenvを使っていて、どのような実行環境でも動作させることが可能になっています。

unenvは主要な機能として次の2つを提供しています:

* **プリセット**: RollupやWebpack等のバンドラーと組み合わせて使う設定をプリセットとして提供していて、Node.js/Deno、Vercel、Cloudflare Workers等の主要な実行環境向けの設定を用意している
* **モック、ポリフィル**: 実行環境毎の差を埋めるためのモックやポリフィル等の実装を用意している

## ユースケース

ここまでの説明だけで、unenvをどのように使うことができるのか想像しにくいと思います。先ほどH3とNitroはunenvを使っていると述べましたが、ここではH3とNitroの事例を基にしてどのような使われ方をしているのか紹介します。

### H3のHTTPリクエストとレスポンス

1つめとして、H3の例を紹介します。
H3はサーバーレス環境、エッジ環境、Node.js/Bun/Deno等の様々なJavaScript実行環境で動作する高速なHTTPサーバーです。Node.jsの`http`モジュールが提供している`IncomingMessage`や`OutgoingMessage`は、Node.js以外のFetch APIに準拠している実行環境が提供している`Request`や`Response`と互換性がありません。Node.js向けに実装したアプリケーションは、そのままではFetch APIに準拠している実行環境で動かすことができません。
H3はこれを解消するためにアダプタという機能を提供しています。アダプタによってNode.jsとそれ以外の実行環境の差を吸収していて、実装したアプリケーションはアダプタを差し替えるだけで別の実行環境で動作させることが可能になっています。

:::message
H3の詳細については、同じシリーズ記事の「[H3 (unjs/h3)について](./introducing-unjs-h3)」という記事で紹介しています
:::

#### モックによる`IncomingMessage`と`OutgoingMessage`の軽量実装

このアダプタ内の処理で、unenvが提供している`IncomingMessage`と`OutgoingMessage`と同じAPIが提供されているが、より軽量な実装になっているモックが使われています。このモックを使うことで`http`モジュールが提供している大きなオブジェクトを初期化しなくてすみ、パフォーマンスへの影響をなくしています。

実際にNode.js向けの実装とFetch APIに準拠している実行環境向けでどのような違いがあるのか紹介します。まずはH3を使ったNode.js向けのHTTPサーバーの基本的な実装は次のようになります:

```javascript
import { createServer } from "node:http";
import { createApp, eventHandler, toNodeListener } from "h3";

const app = createApp();

app.use("/", eventHandler(() => "Hello world!"));

createServer(toNodeListener(app)).listen(8080);
```

`app`インスタンスを作成し、`toNodeListener`関数に渡します。`toNodeListener`はNode.jsで動作させるときに使うアダプタで、`toNodeListener`内で次のような処理が行われます。

```javascript
function toNodeListener(app) {
  return async function (req, res) {
    const event = createEvent(req, res);

    await app.handler(event);
  }
}
```

`toNodeListener`関数内の`req`と`res`は`http`モジュールで初期化された`IncomingMessage`と`OutgoingMessage`のオブジェクトが渡ってきます。このオブジェクトを`createEvent`関数に渡して`event`オブジェクトを作成し、イベントハンドラに渡しています。`IncomingMessage`と`OutgoingMessage`のオブジェクトを`console`等でデバッグしたことがある方ならご存知だと思いますが、これらは凄くデータサイズが大きいオブジェクトになります。

次にFetch APIに準拠している実行環境向けの実装は次のようになります:

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

`app`インスタンスを作成するところまでは同じですが、ここでは`toWebHandler`関数を使っています。この関数はFetch APIに準拠している実行環境で動作させるときに使うアダプタになります。`toWebHandler`関数内で次のような処理が行われます。

```javascript
import { IncomingMessage } from "unenv/runtime/node/http/_request";
import { ServerResponse } from "unenv/runtime/node/http/_response";

async function toWebHandler(app, req) {
  const nodeReq = new IncomingMessage();
  const nodeRes = new ServerResponse(nodeReq);

  const event = createEvent(nodeReq, nodeRes);

  event._method = (request.method || "GET").toUpperCase();
  event._path = req.path;

  await app.handler(event);
}
```

`toWebHandler`関数内の`req`は、Fetch APIの`Request`型のオブジェクトが渡ってきます。`createEvent`関数に渡す引数は、`unenv`で提供されている`IncomingMessage`と`ServerResponse`を初期化したものを渡しています。これらは`http`モジュールの`IncomingMessage`と`OutgoingMessage`と基本的に同じインターフェースですが、不要なプロパティは簡略化されています。

軽量なモックに差し替えることで`app.handler`内の処理はどちらも同じインターフェースによるプロパティを持つオブジェクトを扱うことができ、Fetch APIに準拠している実行環境で`http`モジュールの`IncomingMessage`と`OutgoingMessage`の大きなオブジェクトを作成するための負荷を不要にしています。

### Nitro

NitroはH3と同様、様々な実行環境で動かすことが可能なウェブサーバーを構築するためのライブラリになります。

:::message
Nitroは、この記事では詳細に触れず、別の記事で調査をする予定です。
:::

## プリセット

プリセットは、Node.jsやDeno、Cloudflare WorkersやVercelのEdge Runtime等のそれぞれの実行環境向けに、unenvが提供している設定になります。RollupやWebpack等のビルドツールと組み合わせてビルドすることで、指定した実行環境向けに最適化された成果物が生成されます。

```typescript
interface Environment {
  alias: { [key: string]: string };
  inject: { [key: string]: string | string[] };
  polyfill: string[];
  external: string[];
}

interface Preset extends Partial<Environment> {}
```

プリセットの型定義は`alias`、`inject`、`polyfill`、`external`の4つのプロパティを持っています:

* **alias**: 指定したモジュールをインポートする際のパスを指定します。バンドル時に指定したパスからインポートします。
* **inject**: 
* **polyfill**: モジュールを指定すると、実行環境に不足している機能やAPIを保管するためのコードとしてバンドルされます。
* **external**: 指定したモジュールをバンドル対象から除外して、外部依存として参照するように変換します。

これらの4つのプロパティは、

### プリセット一覧

unenvが提供しているプリセットを紹介します。

#### node

`node`プリセットはNode.js向けの設定で、次の対応がされています:

* Fetch API対応
* Node.jsのビルトインモジュールの依存をバンドル対象から除外

`node`プリセットには、`node-fetch`や`isomorphic-fetch`等のFetch API対応のNPMパッケージをunenvが提供しているモックへのエイリアスを貼っています。Node.jsがコアモジュールとしてFetch API

#### nodeless

`nodeless`プリセットは、

#### deno

`deno`プリセットは、

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
