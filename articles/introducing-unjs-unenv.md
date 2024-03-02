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

### 前提条件

この記事の説明で使うライブラリのバージョンは次のものを使っています:

* `unenv`: v1.9.0
* `H3`: v1.10.2
* `Nitro`: v2.8.1

## unenvの特徴

unenv([unjs/unenv](https://github.com/unjs/unenv))は、ブラウザ、Node.js/Deno/Bun、Workers等の特定の実行環境に依存しないユニバーサルなJavaScriptコードに変換するためのユーティリティライブラリです。基本的にはRollupやWebpack等のビルドツールと組み合わせて使い、実行環境に依存しないJavaScriptコードを生成することができます。unenvと同じくUnJSプロジェクトから提供されているH3やNitroは、どのような実行環境でも動作させることができることを謳っていますが、これらのライブラリが内部的にunenvを使っていることが関係しています。

unenvは主要な機能として、**プリセット**と**モック・ポリフィル**の2つを提供しています:

* **プリセット**: RollupやWebpack等のビルドツール向けの事前定義された設定をプリセットとして提供していて、Node.js/Deno、Vercel、Cloudflare Workers等の主要な実行環境向けの設定を用意しています
* **モック、ポリフィル**: unenvがサポートしている実行環境間の差を埋めるためのモックやポリフィル等の実装を用意しています

## ユースケース

ここまでの説明だけで、unenvをどのように使うことができるのか想像しにくいと思います。先ほどH3とNitroはunenvを使っていると述べましたが、ここではH3とNitroの事例を基にしてどのような使われ方をしているのかを紹介します。

### H3のHTTPリクエストとレスポンス

H3はサーバーレス環境、エッジ環境、Node.js/Bun/Deno等の様々なJavaScript実行環境で動作する高速なHTTPサーバーです。Node.jsの`http`モジュールが提供している`IncomingMessage`や`OutgoingMessage`は、Fetch APIに準拠している実行環境(以後、Fetch API準拠の実行環境)で使える`Request`や`Response`と互換性がありません。Node.js向けに実装したアプリケーションは、そのままではFetch API準拠の実行環境で動かすことができません。
これを解決するために、H3はアダプタという機能を提供しています。アダプタによってNode.jsとFetch API準拠の実行環境の差を吸収しているため、実装したアプリケーションを別の実行環境で容易に動作させることができます。

:::message
H3の詳細については、同じシリーズ記事の「[H3 (unjs/h3)について](./introducing-unjs-h3)」という記事で紹介しています
:::

#### モックによる`IncomingMessage`と`OutgoingMessage`の軽量実装

unenvのアダプタは、`http`モジュールの`IncomingMessage`や`OutgoingMessage`にインターフェースに合わせることで実装の差を吸収しています。

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

`toNodeListener`関数内の`req`と`res`は`http`モジュールで初期化された`IncomingMessage`と`OutgoingMessage`のオブジェクトが渡ってきます。このオブジェクトを`createEvent`関数に渡して`event`オブジェクトを作成し、イベントハンドラに渡しています。

次にFetch API準拠の実行環境向けの実装は、基本的に次のようになります:

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

`app`インスタンスを作成するところまでは同じですが、ここでは`toWebHandler`関数を使っています。この関数はFetch API準拠の実行環境で動作させるときに使うアダプタになります。`toWebHandler`関数内で次のような処理が行われます。

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

`toWebHandler`関数に渡ってくる`req`は、Fetch APIの`Request`オブジェクトが渡ってきます。`createEvent`関数に渡す引数は、`unenv`から提供されている`IncomingMessage`と`ServerResponse`を初期化したものを渡しています。これらは`http`モジュールの`IncomingMessage`と`OutgoingMessage`と基本的に同じインターフェースですが、不要なプロパティが簡略化されたデータサイズが小さいオブジェクトになります。
`http`モジュールの`IncomingMessage`と`OutgoingMessage`のオブジェクトは、`console`等でデバッグしたことがあるならご存知だと思いますが、これらは凄くデータサイズが大きいオブジェクトで、これらのオブジェクトを初期化するのはコストがかかるため、HTTPサーバーがリクエストを受付ける度に初期化するのはパフォーマンスに影響をもたらします。

軽量なモックに差し替えることで`app.handler`内の処理はどちらも同じプロパティを持つオブジェクトを扱うことをできるようにしつつ、Fetch API準拠の実行環境で`http`モジュールの`IncomingMessage`と`OutgoingMessage`のデータサイズが大きなオブジェクトを作成するためのコストを回避しています。

### Nitro

NitroはH3と同様、様々な実行環境で動かすことが可能なウェブサーバーを構築するためのライブラリになります。
Nitroで作成するウェブサーバーをNode.jsで動かす必要がない場合は、`nitro.config.ts`の設定ファイルに次のように指定します:

```typescript:nitro.config.ts
export default defineNitroConfig({
  node: false,
});
```

このように指定することで、ビルドする際にNode.jsに依存する箇所を、unenvで提供されているモックに可能な限り差し替えます。先程のH3の事例と同様、Node.jsの実装を極力モックに差し替え、よりモダンで標準に沿った実装の方に揃えています[^1]。

[^1]: Nitroの実装を見るとNode.jsのプリセット、Nitroのビルトインプリセット、ユーザーが設定ファイルで指定するプリセットの順で優先度が高くなるように実装されていることが分かります。 ([ソースコード](https://github.com/unjs/nitro/blob/b53e00191d0752c7ae7463a907f0858c8bb8182e/src/rollup/config.ts#L59))

:::message
Nitroは、この記事では詳細に触れず、別の記事で調査をする予定です。
:::

## プリセット

プリセットは、Node.jsやDeno、Cloudflare WorkersやVercelのEdge Runtime等の様々な実行環境向けに個別で定義した設定になります。RollupやWebpack等のビルドツールと組み合わせてビルドすることで、指定した実行環境向けに最適化された成果物が生成されます。
プリセットは`alias`、`inject`、`polyfill`、`external`の4つのプロパティを持っています:

* **alias**: 指定したモジュールをインポートする際のパスを指定します。バンドル時に指定したパスからインポートします。
* **inject**: 
* **polyfill**: モジュールを指定すると、実行環境に不足している機能やAPIを保管するためのコードとしてバンドルされます。
* **external**: 指定したモジュールをバンドル対象から除外して、外部依存として参照するように変換します。

### プリセット一覧

unenvが提供しているプリセットを紹介します。

#### node

`node`プリセットはNode.js向けの設定で、**alias**、**polyfill**、**external**の設定がされています:

* **alias**: Fetch API対応。
* **external**: Node.jsのビルトインモジュールをバンドル対象から除外しています。ビルトインモジュールと同名でブラウザ環境で動作させるためのポリフィルのライブラリがあり、それらはNode.jsで動作させるには必要なく、成果物に含まれるのを防ぐために指定されているのだと思います。(例: Node.jsのビルトインモジュールの`assert`は、ブラウザ向けのポリフィルの[assert](https://www.npmjs.com/package/assert)がNPMパッケージとして公開されています)

`node`プリセットは、`node-fetch`や`isomorphic-fetch`等のFetch API対応のNPMパッケージをunenvが提供しているモックに差し替えています。

#### nodeless

`nodeless`プリセットは、Node.js向けに作られたライブラリを他のJavaScript実行環境で動かすための設定になります。

:::message
`nodeless`プリセットが具体的にどのようなユースケースで使われるのか想像できなかったので、別途調査したいと思います。
Node.jsで一般的に使われるモジュールや全ビルトインモジュールをunenvのモックへのエイリアスとして差し替えているため、Node.jsへの依存を極力なくすことが目的のプリセットなのかなと推測していますが、それ以上のことはわかりませんでした。
:::

#### deno, cloudflare, vercel

`deno`、`cloudflare`、`vercel`プリセットは、それぞれ次の実行環境向けの設定になります:

* `deno`: [Deno](https://deno.com/)
* `cloudflare`: [Cloudflare Workers](https://developers.cloudflare.com/workers/)
* `vercel`: Vercelの[Edge Runtime](https://vercel.com/docs/functions/runtimes/edge-runtime)

この3つのプリセットは、**alias**と**external**の設定がされています:

* **alias**: Deno、Cloudflare Workersは、Node.jsのビルトインモジュールをインポートする際に、`node:`プレフィックスを付ける必要があります。`node:`プレフィックスを付けていないインポート文を`node:`プレフィックス付きのモジュールとしてインポートするためのエイリアスを設定しています。
* **external**: Node.jsのビルトインモジュールをバンドル対象から除外しています。`node`プリセットで説明した理由と同様で、Deno、Cloudflare Workers、Edge Runtimeで提供されているNode.jsのビルトインモジュールが成果物に含まれるのを防ぐために指定されていると思います[^2][^3][^4]。

[^2]: DenoでサポートされているNode.jsのビルトインモジュールは、[こちらのドキュメント](https://docs.deno.com/runtime/manual/node/compatibility)で確認できます。
[^3]: Cloudflare WorkersでサポートされているNode.jsのビルトインモジュールは、[こちらのドキュメント](https://developers.cloudflare.com/workers/runtime-apis/nodejs/)で確認できます。
[^4]: Edge RuntimeでサポートされているNode.jsのビルトインモジュールは、[こちらのドキュメント](https://vercel.com/docs/functions/runtimes/edge-runtime#compatible-node.js-modules)で確認できます。

:::message
`deno`、`cloudflare`、`vercel`プリセットは、実験段階のプリセットなので変更される可能性があります。
:::

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
