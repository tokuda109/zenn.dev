---
title: "H3 (unjs/h3)について"
emoji: "⚡"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["h3", "nuxtjs", "nuxt3", "unjs"]
published: true
publication_name: "comm_vue_nuxt"
---

## この記事について

この記事は、UnJSの主要なライブラリを調査していくシリーズ記事の1つになります。
シリーズ記事の概要や今後公開される予定の記事の確認は[こちらの記事](./introducing-unjs)を参照してください。今回はH3というライブラリについて紹介します。

### 前提条件

この記事の説明で使うライブラリのバージョンは次のものを使っています:

* `H3`: v1.10.0

## H3の特徴

H3([unjs/h3](https://github.com/unjs/h3))は軽量で高速なHTTPサーバーのフレームワークで、HTTPサーバー部分の実装としてNuxt(Nitro)に含まれていますが、H3単体で使うこともできます。

H3の主要な特徴として、次のものがあります:

* サーバーレス環境、エッジ環境、Node.js/Bun/Deno等の様々な環境で動作し、移植性が高い
* 軽量なフレームワークで、パフォーマンス性能が高い
* [unjs/radix3](https://github.com/unjs/radix3)を使った高速なルーティング (※ radix3については[こちらの記事](./introducing-unjs-radix3)で紹介しています)
* Web標準に準拠したシンプルなAPI
* node/connect/expressのミドルウェアとの互換性を提供

## なぜHTTPサーバーのライブラリが必要になるのか?

Node.jsで動作するHTTPサーバーとして、フロントエンド開発でよく使われる[Express.js](https://expressjs.com/)や高速で動作する[Fastify](https://fastify.dev/)等、様々なHTTPサーバーのライブラリが開発されています。
なぜこれらのライブラリが開発者に必要とされているかと言うと、Node.jsがコアモジュールとして提供している[http](https://nodejs.org/api/http.html)モジュールは最小限の機能しか提供していないため、`http`モジュールだけで開発する場合、次の2点が問題になります。

* `http`モジュールの開発体験の低さ
* `http`モジュールの移植性の低さ

### httpモジュールの開発体験の低さ

Node.jsは「コア(Node.jsそのものに付属する機能)は最小限に保つべきである」というスモールコアの哲学に基づき開発されています[^1]。
Node.jsの`http`モジュールが提供するAPIは低水準で必要最低限のものしかないため、実際の開発現場ではルーティングのネスト機能やダイナミックルーティング等のより高水準なAPIが必要となります。`http`モジュールの実装は次のように記述する必要があり、Express.jsと比べてコードが冗長になります。

[^1]: [ハンズオンNode.js](https://www.oreilly.co.jp//books/9784873119236/) オライリー・ジャパン、1.1.2: スモールコアとnpm

```javascript
// Node.jsのhttpモジュールの場合
import { createServer } from "node:http";

const server = http.createServer();

server.on("request", (req, res) => {
  if (req.url === "/" && req.method === "GET") {
    res.setHeader("Content-Type", "application/json");
    return res.end("Hello world!");
  }

  res.statusCode = 404;
  res.end("404 not found");
});

server.listen(8080);

// Express.jsを使った場合
const express = require("express");

const app = express();

app.get("/", (_, res) => {
  res.send("Hello World!")
})

app.use((err, _, res) => {
  res.status(404).send("404 not found")
});

app.listen(8080);
```

HTTPサーバーのライブラリを使わずに`http`モジュールで実装する方が、追加の処理を行わないため動作速度としては有利になります。しかし、アプリケーション開発の規模がある程度大きくなると、何かしらのHTTPサーバーのライブラリを使って開発生産性を上げることが必要となるでしょう。

### httpモジュールの移植性の低さ

`http`モジュールの[IncomingMessage](https://nodejs.org/api/http.html#class-httpincomingmessage)と[OutgoingMessage](https://nodejs.org/api/http.html#class-httpoutgoingmessage)は、Web標準APIで策定されているFetch APIの[Request](https://developer.mozilla.org/en-US/docs/Web/API/Request)と[Response](https://developer.mozilla.org/en-US/docs/Web/API/Response)に対して互換性がありません。

Node.jsはFetch APIの仕様が策定されるよりも前に開発が開始され、さらに実装上の都合から導入に時間がかかっていました[^2]。Node.jsで使えるようになったのは、2022年に公開されたバージョン18になってからになります[^3]。バージョン18以降のNode.jsではFetch APIを使うことができますが、`http`モジュールに関しては後方互換性を維持する必要もあるため、Fetch APIの`Request`や`Response`等と統一して使えるように変更されることは無いのかなと考えています。(この辺りに詳しい方は教えて下さい)

実行環境としてNode.js以外にもDeno/Bun等の新しいランタイム環境やエッジ環境等が近年登場し、Web標準APIに準拠しているこれらの環境の上でNode.jsアプリケーションを動かす機運が高まっています。このような状況から、特定の実行環境のみに依存しない移植性の高さが必要となります。

既に様々なHTTPサーバーのライブラリがあるなかでH3が開発されたのは、H3のどのような環境でも同様に動かすことができるというコンセプトは、H3の開発が始まったときは新しい試みでした[^4]。

[^2]: [The Fetch API is finally stable in Node.js](https://blog.logrocket.com/fetch-api-node-js/)
[^3]: [Node.js 18 is now available!](https://nodejs.org/en/blog/announcements/v18-release-announce)
[^4]: [H3 1.8 - Towards the Edge of the Web](https://unjs.io/blog/2023-08-15-h3-towards-the-edge-of-the-web)

### HTTPサーバーライブラリに求められる要件

以上のことから、**`http`モジュールに開発体験や移植性を上げるための機能を追加するが、可能な限りパフォーマンスは劣化させない**ことがHTTPサーバーのライブラリに求められます。
この点において、H3はこの要件を満たしているHTTPサーバーライブラリの1つになります。

## H3の機能

ここからはH3の機能で重要なものを1つずつ紹介していきます。

### App

H3サーバーは1つ以上の`App`型のオブジェクトを持ち、`createApp`関数を使って初期化します。

```javascript
import { createApp } from "h3";

const app = createApp();
```

初期化した`app`インスタンスにリクエストを受け付けた時の処理(以下、イベントハンドラという)を記述し、`app`インスタンスに登録するのが基本的な使い方になります。イベントハンドラを登録するには`app.use`メソッドに`eventHandler`を渡して次のように記述します。

```javascript
import { eventHandler } from "h3";

app.use(eventHandler(() => "Hello world!"));
```

そして最後にNode.jsで動かす場合は、`app`インスタンスをNode.jsに渡すための関数(サンプルコードの`toNodeListener`)を使って`http`モジュールの`createServer`に次のように渡すことでNode.jsのHTTPサーバーとして動作します。(`toNodeListener`はアダプタという機能で、Node.js以外の環境で動かすためのものもあります。後ほど詳しく説明します)

```javascript
import { createServer } from "node:http";
import { toNodeListener } from "h3";

createServer(toNodeListener(app)).listen(8080);
```

#### スタック (`app.stack`)

`app`インスタンスは`stack`というプロパティを持っています。値としては`Layer`型のオブジェクトを配列で保持するものになります。イベントハンドラを登録すると`stack`に要素追加されます。

```typescript
type Stack = Layer[];
```

#### オプション (`app.options`)

`createApp`関数に渡すオプションとして`debug`や`onRequest`、`onError`等のグローバルフックがあり、デバッグ機能を有効にするには次のように記述します。実装を見てみると、デバッグ機能を有効にすることでレスポンスにJSONデータを返却する場合にデータを整形してくれます。

```javascript
const app = createApp({
  debug: true,
});
```

グローバルフックは以下のコードに記載している4つのプロパティが用意されていて、プロパティに関数を指定すると`app`インスタンスがリクエストを受け取った際に特定のタイミングで呼ばれます。各関数がどのようなタイミングで呼び出されるかは次のサンプルコードにコメントで記載しています。

```javascript
const app = createApp({
  onError: (error, event) => {
    // ハンドラが処理中に何かしらのエラーが発生した際に呼び出され、
    // H3Error(`error`)とH3Event(`event`)が渡ってくる。
  },
  onRequest: (event) => {
    // 全てのリクエストに対して呼び出され、H3Event(`event`)が渡ってくる。
  },
  onBeforeResponse: (event, response) => {
    // リクエストがレスポンスとして返却するデータがある場合、返却する前に呼び出され、
    // H3Event(`event`)と返却する前のデータ(`response`)が渡ってくる。
  },
  onAfterResponse: (event, response) => {
    // リクエストがレスポンスとして返却するデータがある場合、返却した後に呼び出され、
    // H3Event(`event`)と返却したデータ(`response`)が渡ってくる。
  },
});
```

#### `use`メソッド (`app.use`)

`app`インスタンスにイベントハンドラを登録する`use`メソッドの型は次のように定義されていて、引数の型として関数型が3つ宣言されています。

```typescript
interface AppUse {
  (
    route: string | string[],
    handler: EventHandler | EventHandler[],
    options?: Partial<InputLayer>
  ): App;
  (
    handler: EventHandler | EventHandler[],
    options?: Partial<InputLayer>
  ): App;
  (options: InputLayer): App;
}
```

具体的にどのように引数を指定できるのか一目見て理解できなかったので調べました。

##### パターン1

第1引数にパス、第2引数にイベントハンドラを取り、第3引数が任意でオプションを取ります。第1引数と第2引数は配列として渡すこともできます。

第1引数を配列で指定した場合、配列で渡した全てのパスにマッチしてイベントハンドラが呼ばれます。また第2引数を配列で指定した場合、レスポンスデータとして返り値を返すイベントハンドラに当たるまで、配列の先頭のイベントハンドラから順に呼び出されます。

:::message alert
第1引数として渡すパスは前方一致でマッチするため、`/hello` を指定すると、`/hello` にも `/hello/world` にもマッチするので注意してください。
:::

```javascript
app.use("/hello", eventHandler(() => "Hello world!"));

// Route Array
app.use(["/ok", "/okok"], eventHandler(() => "OK!"));

// Handler Array
app.use("/1", [eventHandler(() => "1"), eventHandler(() => "2")]); // => 1
app.use("/2", [eventHandler(() => {}), eventHandler(() => "2")]); // => 2
```

サンプルコードのデモアプリを作成したので、次のプレビューから動作確認をすることができます。しかし、ルートパス(`/`)にマッチするイベントハンドラを登録していないため、サンプルコードの何れかのパスを指定して下さい。

@[stackblitz](https://stackblitz.com/edit/unjs-h3-appuse-demo-pattern1?ctl=1&embed=1&file=package.json&hideExplorer=1&view=preview)

##### パターン2

第1引数にイベントハンドラを取り、第2引数が任意でオプションを取ります。パターン1と同様に第1引数は配列として渡すことができます。
パスを指定せずに、イベントハンドラだけを登録するため、全てのリクエストに対してイベントハンドラが呼ばれます。

```javascript
app.use(eventHandler(() => "Hello world!"));

// Handler Array
app.use([eventHandler(() => "1"), eventHandler(() => "2")]); // => 1
app.use([eventHandler(() => {}), eventHandler(() => "2")]); // => 2
```

全てのリクエストに対して登録順にイベントハンドラを呼び出すため、パターン2のサンプルコードは一番上のイベントハンドラを処理した時点でレスポンスデータを返却しているため、全てのパスに対して`Hello world!`を返却します。(2番目と3番目のイベントハンドラは`app`インスタンスに登録はされていますが、呼び出されることはありません)
次のプレビューで任意のパスを指定して動作確認ができます。

@[stackblitz](https://stackblitz.com/edit/unjs-h3-appuse-demo-pattern2?ctl=1&embed=1&file=app.ts&hideExplorer=1&view=preview)

##### パターン3

パターン1とパターン2の使い方は、Express.js等の他のHTTPサーバーライブラリを既に使ったことがあるなら簡単な内容でした。しかし、パターン3は少し読み進めないと把握できなかったです。

パターン3は、第1引数に`InputLayer`型のオブジェクトを取ります。`InputLayer`は2つの使われ方をしますが、それを1つの型として定義しています。イベントハンドラを登録する際のオプションとして使うのがまず1つ、`handler`プロパティにイベントハンドラが指定されているオブジェクトを受け取って`app`インスタンスにイベントハンドラを登録するのがもう1つになります。
まずは`InputLayer`インターフェイスがどのように宣言されているか見てみましょう。

```typescript
interface InputLayer {
  route?: string;
  match?: Matcher;
  handler: EventHandler;
  lazy?: boolean;
}
```

`handler`プロパティだけ必須で、残りのプロパティはオプショナルになります。パターン1の第3引数とパターン2の第2引数はオプショナルで`InputLayer`型のオブジェクトを渡しますが、`Partial<InputLayer>`のように組み込みの`Partial`型によって全てのプロパティがオプショナルにされています。
パターン1とパターン2のオプションとして渡すことができるプロパティはドキュメントや実装から調べるしかありません。

パターン1の第3引数として渡すことができるプロパティは`match`と`lazy`の2つになります。`match`の使い方はドキュメントに記載されていて、次のようになります。

```javascript
app.use(
  "/",
  eventHandler(() => "This is odd!"),
  { match: (url) => url.substr(1) % 2 },
);
```

第1引数で指定したパスがマッチするかどうかを追加で評価することができ、`false`を返すとマッチしません。
`lazy`の使い方もドキュメントに記載されていて、次のようになります。

```javascript
app.use("/big", () => import("./big-handler"), { lazy: true });
```

このオプションを有効にすると、指定したパスにマッチするまでイベントハンドラをインポートしないため、サーバーの起動時間を短縮することができます。また、パターン2の第2引数はパターン1の第3引数と同様のプロパティを渡すことができます。

`handler`プロパティを持つオブジェクトを渡すケースは、ルーター機能を使う時になります。[`Router`インターフェイス](https://github.com/unjs/h3/blob/ae91fc8658315dca75da22b665eda4175eef7ea8/src/router.ts#L30-L34)はプロパティに`handler`を持ち、`app.use`メソッドに渡すことで、ルーターに登録したイベントハンドラを`app`インスタンスに登録します。

### ルーター

ルーターは、HTTPリクエストをパスやHTTPメソッドに応じて、登録されているイベントハンドラに適切に振り分けます。ルーター機能自体はradix3([unjs/radix3](https://github.com/unjs/radix3))で実装されていて、[基数木](https://ja.wikipedia.org/wiki/基数木)(Radix tree)というデータ構造を基にしてパスデータを格納しているため、高速なパス探索を可能にしています。基本的な使い方は次のようになります。

:::message
radix3や基数木は、この記事では詳細に触れず、別の記事で調査をする予定です。ここではパスの探索を高速に行えるということだけ覚えておいて下さい。
:::

:::message
次のサンプルコードは`app.use`に`router`インスタンスを渡しているので、先ほど説明した`app.use`の使い方のパターン3に該当します。
:::

```javascript
import { createServer } from "node:http";
import { createApp, createRouter, eventHandler, toNodeListener } from "h3";

const app = createApp();
const router = createRouter();

router.use("/", eventHandler(() => "Hello world!"));
app.use(router);

createServer(toNodeListener(app)).listen(8080);
```

`createRouter`関数を使って初期化します。初期化した`router`インスタンスの`use`メソッドに対して、`app.use`にイベントハンドラを登録した時と同様にイベントハンドラを登録します。
違いとしては、`app.use`に渡したパスは前方一致でマッチしますが、`router.use`はリクエストのパスと一致しないとマッチしません。

最後に`router`インスタンスを`app.use`に渡すことで、`router`インスタンスに対して登録したルーティング情報を`app`インスタンスに登録することになります。

#### ルーターメソッド

`router`インスタンスには、[HTTPメソッド](https://www.rfc-editor.org/rfc/rfc7231#section-4.1)に対応したメソッドを提供していて、同じパスでHTTPメソッド毎に処理を分ける場合に使えます。例として、トップページでGETリクエストを受け付ける場合、`router.get("/", eventHandler(() => { ... }))`のようにします。さらに同じパスでPOSTリクエストも受け付ける場合、`router.post("/", eventHandler(() => { ... }))`のようにします。
Node.jsだと`server.on("request", () => {})`のコールバック関数内でHTTPメソッドによる分岐をすることになりますが、サンプルコードの方が直感的で分かりやすい実装だと思います。

```javascript
const router = createRouter()
  .get("/", eventHandler(() => "GET: Hello world!"))
  .post("/", eventHandler(() => "POST: Hello world!"))
  .put("/", eventHandler(() => "PUT: Hello world!"))
  .delete("/", eventHandler(() => "DELETE: Hello world!"))
  .patch("/", eventHandler(() => "PATCH: Hello world!"))
  .head("/", eventHandler(() => "HEAD: Hello world!"));
```

#### ルートパラメータ

動的なルーティングのために、コロン(`:`)を使ってURLセグメントを記述することでパラメータとして受け取ることができます。例として、複数人のユーザー詳細ページが存在するウェブサイトがある場合、次のように実装します。

```javascript
import { createServer } from "node:http";
import { createApp, createRouter, eventHandler, toNodeListener } from "h3";

const app = createApp();
const router = createRouter()
  .get("/users/:name", eventHandler((event) => `Hello ${event.context.params.name}!`));

app.use(router);

createServer(toNodeListener(app)).listen(8080);
```

`router`インスタンスの`get`の第1引数にパスとして`/users/:name`が指定されています。`:name`の部分が任意の文字列を受け取ることができるため、`/users/Yamada`でも`/users/Tanaka`でもマッチして、第2引数に渡したイベントハンドラが呼ばれます。イベントハンドラ内でパラメータを受け取るには、イベントハンドラのコールバック関数に渡ってくる`event`から参照できます。
次のプレビューで`/users/Yamada`のパスを指定して動作確認ができます。

@[stackblitz](https://stackblitz.com/edit/unjs-h3-route-params-demo?ctl=1&embed=1&file=app.ts&hideExplorer=1)

#### ネスティングルート

ルーティングをリソース種別毎に分けて管理したい場合、ルーティングをネストさせることができます。

```javascript
import { createApp, createRouter, eventHandler, useBase } from "h3";

export const app = createApp();

const indexRouter = createRouter()
  .get("/", eventHandler(() => "Hello world!"))

app.use(indexRouter);

const usersRouter = createRouter()
  .get("/", eventHandler(() => {
    // ユーザー一覧を取得して返却する処理
    return [];
  }))
  .get("/:id", eventHandler((event) => {
    // パラメータで指定されたユーザーを取得して返却する処理
    // 存在しなければ404を返却するようにする
    return {};
  }));

indexRouter.use('/users/**', useBase('/users', usersRouter.handler));
```

サンプルコードのように、`/users/`はユーザー一覧を返却し、`/users/:id`でユーザー別の詳細情報を返却する場合、`/users`のルーティングを扱うためだけの別の`usersRouter`インスタンスを作成し、登録することができます。
次のプレビューで`/users/1`のパスを指定して下さい。IDが1のユーザー情報が返ってきます。

:::message
次のプレビューに`/users/`のパスを指定すれば、本来ならユーザー一覧が返ってくる想定でしたが、404になります。これはバグのような気がするので、もう少し調査してみます。
H3のリポジトリに`nested-router.ts`というサンプルコードがありますが、こちらのサンプルアプリケーションに`/api/`でレスポンスを返すイベントハンドラを登録してみましたが、同じ結果になりました。
:::

@[stackblitz](https://stackblitz.com/edit/unjs-h3-route-nesting-demo?ctl=1&embed=1&file=app.ts&hideExplorer=1&view=preview)

### イベント

H3サーバーは、リクエストとして受け取って、レスポンスとして返却するまでの一連の処理をイベントデータ(内部的には`H3Event`型の値)として受け渡すことで処理を行います。内部的には`http`モジュールの`IncomingMessage`と`ServerResponse`を保持しています。簡略的に書くと次のようになります。

```typescript
class H3Event {
  node: {
    req: IncomingMessage;
    res: ServerResponse;
  };

  constructor(req: IncomingMessage, res: ServerResponse) {
    this.node = { req, res };
  }
}
```

Node.jsでH3を動かす場合は、Node.jsで作成された`IncomingMessage`と`ServerResponse`が渡されます。Web標準APIに準拠した環境では、`unenv`による軽量化された`IncomingMessage`と`ServerResponse`のオブジェクトが作成され、渡されます。

:::message
unenvは、この記事では詳細に触れず、別の記事で調査をする予定です。
:::

### イベントハンドラ

イベントハンドラは、`App`インスタンスや`Router`インスタンスに登録され、リクエストを受け付けた時にパスがマッチすると呼ばれます。イベントハンドラ内で値を返すことで、適切なレスポンスデータに変換され、レスポンスとして返却されます。
イベントハンドラで返す値の種別によって次のようにレスポンスの内容が変わります。

* JSONオブジェクトかシリアライズされた値を返すと、`Content-Type`が`application/json`で返却されます
* 文字列で値を返すと、`Content-Type`が`application/html`で返却されます
* `null`を返すと、`204 - No Content`のステータスコードで返却されます

[ReadableStream](https://developer.mozilla.org/en-US/docs/Web/API/ReadableStream)、[ArrayBuffer](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/ArrayBuffer)、[Response](https://developer.mozilla.org/en-US/docs/Web/API/Response)を返却することもできます。

また、イベントハンドラは`async/await`をサポートしていて、返り値として`Promise`を返すこともできます。

```javascript
app.use("/api", eventHandler(async (event) => ({ url: event.node.req.url })));
```

イベントハンドラは、ここまで`eventHandler`を使って実装していますが、`defineEventHandler`も使えます。

```javascript
const eventHandler = defineEventHandler;
```

H3で`App`インスタンスや`Router`インスタンスにイベントハンドラを登録する場合は`eventHandler`を使い、Nuxtでは`defineEventHandler`が使われています。使い分けのルールを把握していませんが、Nuxtプロジェクトだと`defineNuxtConfig`をはじめとして至るところで`define`プリフィックスが付いた関数を使うため、`defineEventHandler`の方が使われているのかもしれません。

### アダプタ

H3はアダプタという機能を持っていて、`app`インスタンスを実行環境に適用させるための変換を行います。
アダプタの種類は次の3つがあります:

* Node.js用のアダプタ
* Web標準APIに準拠した環境用のアダプタ (Web Adapter)
* プレーンアダプタ (Plain Adapter)

Node.js用のアダプタの使い方はここまでで何回か紹介しているため、ここでは省略します。Web標準APIに準拠した環境で動かすには次のようにします。

```javascript:index.js
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

[Bun](https://bun.sh/)で実行するには次のコマンドを実行します。

```
$ bun run index.js
```

Bun以外にもDeno、Cloudflare、Netlify等で動かすためのサンプルは[ドキュメント](https://h3-7i3.pages.dev/runtimes/web)で紹介されているため、そちらを参照して下さい。

:::message
プレーンアダプタは全てのアダプタを抽象化した基底レイヤとして実装されているのかなと思ったのですが、完全には理解できませんでした。
:::

### ユーティリティ

ここからはH3で提供されている使用頻度が高い、便利なユーティリティをいくつか紹介します。

#### getQuery

クエリパラメーターをオブジェクトで取得することができます。

```javascript
console.log(event.path);
// => /?bool=true&name=string&number=1
console.log(getQuery(event));
// => { bool: 'true', name: 'string', number: '1' }
```

#### getRouterParams, getRouterParam

```javascript
console.log(event.context.matchedRoute.path, event.path);
// => /:id /1
console.log(getRouterParams(event));
// => { id: '1' }

console.log(event.context.matchedRoute.path, event.path);
// => /:id /1
console.log(getRouterParam(event, "id"));
// => 1
```

## パフォーマンス

Fastifyがベンチマーク計測ツールを提供していて、[計測結果](https://github.com/fastify/benchmarks?tab=readme-ov-file#benchmarks)を見ることができます。
2023年12月25日時点では、H3はハイパフォーマンスで有名なFastifyよりも高い秒間リクエストを記録していて、パフォーマンス性能が高いHTTPサーバーと言えます。

![](/images/fastify-benchmark-score.png)

### 早い要因

ドキュメントやソースコードを読んでみて、次の点がH3がパフォーマンススコアが良い要因だと感じました(他にも要因があればコメントで教えて下さい):

* 基数木というデータ構造に基づくルーティングの探索 (※ 基数木については[こちらの記事](./introducing-unjs-radix3)で紹介しています)
* `unenv`によるNode.js環境と互換性のある軽量なオブジェクトでの代用 (※ `unenv`は別で記事を公開する予定です)
* 実装がシンプル (注意: 個人的な感想です)

1番目と2番目は別で調査する予定なので、この記事では深堀りはしません。
実装がシンプルというのは、他のライブラリを全て見たわけではないので、個人の感想になりますが、H3はソースコードを読んで、読みにくかった箇所は比較的少なかったです。それくらい単純で小さな規模の実装を保つことができているのかなと思いました。
また、規模が小さいということは余分な処理が発生しにくく、これがパフォーマンスにつながっていると感じました。

## 参考

* [公式ドキュメンテーションサイト](https://h3.unjs.io/)

## 変更履歴

* 2024/03/02: 公式の[ドキュメンテーションサイト](https://h3.unjs.io/)が公開されたため、URLを差し替えて、前提条件を追記しました
