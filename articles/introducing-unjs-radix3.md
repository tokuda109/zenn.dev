---
title: "radix3 (unjs/radix3)について"
emoji: "🌲"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["nuxtjs", "nuxt3", "radix", "router", "unjs"]
published: true
publication_name: "comm_vue_nuxt"
---

## この記事について

この記事は、UnJSの主要なライブラリを調査していくシリーズ記事の1つになります。
シリーズ記事の概要や今後公開される予定の記事の確認は[こちらの記事](./introducing-unjs)を参照してください。今回はradix3というライブラリについて紹介します。

## radix3の特徴

radix3([unjs/radix3](https://github.com/unjs/radix3))は軽量で高速なルーターで、[基数木](https://ja.wikipedia.org/wiki/基数木)(英: [Radix tree](https://en.wikipedia.org/wiki/Radix_tree))というデータ構造に基づいて実装されています。シリーズ記事の[前回の記事](./introducing-unjs-h3)で紹介した`H3`ライブラリのルーター部分で使われています。Charlie Duong氏により開発された[Radix Router](https://github.com/ctdio/radix-router)というライブラリを基にして実装されています。

radix3の主要な特徴として、次のものがあります:

* 基数木というデータ構造に基づいて実装された軽量で高速なルーター
* パスをキーにしてデータを格納したり、取り出したりすることができる**Router**を提供
* `Router`に格納したデータを全探索して取り出すための**Matcher**を提供 (※ なぜ`Matcher`が提供されているのかは後述します)

radix3の説明をする前に、木構造と基数木について説明します。

## 木構造

基数木は木構造の一種であるため、最初に木構造について説明します。[木構造](https://ja.wikipedia.org/wiki/木構造_(データ構造))(英: Tree)は図1のように個々のデータが親子関係を持った0個以上の子データを持ち、木が階層毎に分岐していくようなデータ構造になります。木構造を構成する個々の要素を**ノード**、ノード間の関係を**エッジ**と呼びます。
木構造の最上位の階層の親を持たないノードを**根ノード**、末端にある子ノードを持たないノードを**葉ノード**、そして子ノードを持つノードを**内部ノード**と呼びます。親ノードと子ノードの距離を1とした場合、根ノードから葉ノードまでの最大距離を木の高さと呼びます。このことから、図1の木構造の木の高さは**3**になります。

![木構造](/images/tree-structure.png)
*図1: 木構造のデータ構造と用語*

#### 木構造の種類

木構造には様々な種類があり、代表的なものは次のものがあります:

* [二分木](https://ja.wikipedia.org/wiki/二分木) (英: Binary tree): 各ノードが2つの子ノードしか持たないデータ構造になります
* 多分木 (英: Multi-branch tree): 二分木は2つの子ノードしか持ちませんが、多分木は3つ以上の子ノードを持つデータ構造になります
* [トライ木](https://ja.wikipedia.org/wiki/トライ_(データ構造)) (英: Trie tree): 各ノードがデータとして1つの文字列を格納するデータ構造になります。

この記事で紹介する基数木は、トライ木の一種に分類されます。

## 基数木

基数木はトライ木の一種で、各ノードに文字列を格納するデータ構造になります。トライ木との違いとして、**各ノードに格納する文字列を1文字以上の任意の長さ**にすることができます。ウェブアプリケーションは、ディレクトリ構造によって階層化されたパスを持つことが一般的です。複数文字列を1つのノードとして扱うことが可能な基数木は、最も効率的にパスを格納し、探索することができるデータ構造になります。

基数木がradix3にどのように応用されているのか、[Twitter API](https://developer.twitter.com/en/docs/twitter-api)のいくつかのエンドポイントのパスを用いて説明します。

:::message
ここで**応用**と表現したのは、高速なルーターとしての機能は十分提供できていますが、基数木で求められる要件を全て満たしているわけでないことが調査をしてみて分かったからです。
:::

Twitter APIの中から次のエンドポイントを使用して、radix3にデータを格納してみます。

* `/tweets`
* `/tweets/:id`
* `/tweets/counts/all`
* `/tweets/counts/recent`
* `/users/:id/bookmarks`
* `/users/:id/likes`
* `/users/:id/tweets`

エンドポイントのパスがどのように格納されるかを説明するのが目的なので、HTTPメソッドは考慮していませんが、これらのパスをradix3に格納すると、図2のようになります。

![Twitter APIのデータ構造](/images/radix3-data-structure-demo.png)
*図2: Twitter APIをradix3に格納した時のデータ構造*

パスを`/`で分割して、パスセグメント毎にノードが割り当てられています。ディレクトリによって階層化される一般的なウェブアプリケーションにおいて、各ノードが1つの文字列しか格納できないトライ木より、パスセグメント毎に1つのノードとして扱うことができる基数木の方が効率的にデータを格納できていることが図2からも分かります。

次から`Router`と`Matcher`の使い方について説明します。

## Router

`Router`はパスをディレクトリ階層毎に分割し、パスセグメントをキーとしてデータを格納します。次のように`createRouter`関数を使って初期化します。

```javascript
import { createRouter } from "radix3";

const router = createRouter();
```

`createRouter`関数はオプションを渡して初期化することもできます。

```javascript
const router = createRouter({
  strictTrailingSlash: true,
  routes: {
    "/foo": {},
  },
});
```

`strictTrailingSlash`に`true`を渡すと、パスの末尾のスラッシュの有無は厳密に判定されます。(デフォルトは`false`になり、末尾のスラッシュは取り除かれた上で挿入されます)
`routes`に渡すオブジェクトは、パスと格納するデータのセットを初期化の段階で指定することができます。

初期化した`router`インスタンスは、`insert`、`lookup`、`remove`の3つのメソッドを提供しています。

データを格納するには、`insert`メソッドの第1引数にパス、第2引数に格納するデータを渡して呼び出します。格納するデータは任意のオブジェクトを渡すことができますが、`params`プロパティは予約されていて使うことができません。

```javascript
router.insert("/hello", { payload: "Hello world!" });
```

格納したデータを探索するには、`lookup`メソッドにパスを渡して呼び出します。該当するパスが存在すると、格納しているデータが返却されます。

```javascript
router.lookup("/hello");
// => { payload: "Hello world!" }
```

格納したデータを削除するには、`remove`メソッドにパスを渡して呼び出します。削除に成功すると`true`が返却され、以降は`lookup`メソッドで削除済みのパスを探索しても`null`が返却されます。

```javascript
router.remove("/test");
// => false (存在しないパスを削除すると、falseが返ってきます)

router.remove("/hello");
// => true (削除に成功すると、trueが返ってきます)

router.lookup("/hello");
// => null
```

### Placeholder (`:`)

Placeholderは、パスの一部をパラメータとして受け取るための記述方法で、次のサンプルコードのようにパスセグメントの文字列の先頭にコロン(`:`)を使って記述します。Placeholderによって指定したパスセグメントは、任意の文字列に該当します。
`lookup`メソッドを呼び出して該当するパスが存在すると、格納しているデータの`params`プロパティに結果を詰めて返却します。結果として返却されるオブジェクトは、パスセグメントのコロンを取り除いた残りの文字列がキーになり、探索に使われて実際に該当したパスセグメントの文字列が値になります。

```javascript
router.insert("/hello/:word", { payload: "named route" });
router.lookup("/hello/world");
// => { payload: "named route", params: { word: "world" } }
router.lookup("/hello/everyone");
// => { payload: "named route", params: { word: "everyone" } }

router.insert("/test/:foo/:bar", { payload: "named route" });
router.lookup("/test/foo/bar");
// => { payload: "named route", params: { foo: "foo", bar: "bar" } }
```

また、ドキュメントには小さくしか書かれていませんが、アスタリスクを1つ(`*`)使う方法も存在しています。この方法を使う場合は、`params`のキーはパスの階層順にインクリメントされ、次のサンプルコードのようなデータとして返却されます。

```javascript
router.insert("/test/*/*", { payload: "named route" });
router.lookup("/test/foo/bar");
// => { payload: "named route", params: { _0: "foo", _1: "bar" } }
```

### Wildcard (`**`)

Wildcardは、パスセグメントにアスタリスクを2つ(`**`)繋げて記述することで、以降のパスを全てマッチさせて、パラメータとして受け取ることができます。以降のパスを全てマッチさせることからパスの一番最後のパスセグメントとして指定します。`params`のキーは`_`になり、次のサンプルコードのようなデータが返却されます。

```javascript
router.insert("/test/**", { payload: "wildcard route" });

router.lookup("/test/foo");
// => { payload: "wildcard route", params: { _: 'foo' } }
router.lookup("/test/foo/bar");
// => { payload: "wildcard route", params: { _: 'foo/bar' } }
```

### Shadowed route

`Shadowed route`は、radix3で`Router`が提供されているのになぜ`Matcher`も提供されているのか疑問に思って調査していく際に、H3のソースコード内で発見した仕様になります[^1]。日本語に翻訳すると**隠されたルート**と言ったところでしょうか。

[^1]: [https://github.com/unjs/h3/blob/main/test/router.test.ts#L101-L122](https://github.com/unjs/h3/blob/845eecaaaeffdb415321f76744e31d55074f37b8/test/router.test.ts#L101-L122)

H3のルーターは、次のサンプルコードのようにHTTPメソッドが異なる同じパスに対して、それぞれにイベントハンドラを登録することができます。全てのHTTPメソッドのリクエストを受け取る`use`メソッドに渡したパスは、Wildcardを使って記述しているため、`/hello/world`のパスに該当します。`Shadowed route`は、このような条件で問題になります。

```javascript:Shadowed routeのサンプルコード1
import { createRouter, eventHandler } from "h3";

const router = createRouter()
  .post("/hello/world", eventHandler(() => "[POST] /hello/world"))
  .delete("/hello/world", eventHandler(() => "[DELETE] /hello/world"))
  .use("/hello/**", eventHandler(() => "[ALL] /hello/**"));
```

H3サーバーを起動して、次のようなリクエストを送ります:

```
$ curl -X POST http://localhost:3000/hello/world
[POST] /hello/world

$ curl http://localhost:3000/hello/world
[ALL] /hello/**
```

GETリクエストに対しては、サンプルコード1の`router.use`で登録したイベントハンドラが処理され、レスポンスを返却しています。

H3の`Router`は、内部的にradix3の`Router`を使っています。サンプルコード1のようなルーティングの実装をすると、次のサンプルコード2のように`handlers`プロパティにHTTPメソッドをキーとしてイベントハンドラが格納されます。

```javascript:Shadowed routeのサンプルコード2
import { createRouter } from "radix3";

const router = createRouter()
  .insert("/hello/world", {
    handlers: {
      post: handler, // このhandlerは、`[POST] /hello/world`を返すイベントハンドラ
      delete: handler, // このhandlerは、`[DELETE] /hello/world`を返すイベントハンドラ
    },
  })
  .insert("/hello/**", {
    handlers: {
      all: handler, // このhandlerは、`[ALL] /hello/**"`を返すイベントハンドラ
    },
  });
```

H3サーバーは、`/hello/world`に対してGETリクエストを受け取ると、radix3の`router`インスタンスの`lookup`メソッドに`/hello/world`を渡して呼び出します[^2]。そして、`lookup`メソッドの返り値の中からGETリクエストのイベントハンドラを取り出そうとします[^3]。

[^2]: [https://github.com/unjs/h3/blob/main/src/router.ts#L92](https://github.com/unjs/h3/blob/845eecaaaeffdb415321f76744e31d55074f37b8/src/router.ts#L92)
[^3]: [https://github.com/unjs/h3/blob/main/src/router.ts#L110-L111](https://github.com/unjs/h3/blob/845eecaaaeffdb415321f76744e31d55074f37b8/src/router.ts#L110-L111)

この2つの処理を簡略して説明的に書くと次のようになります:

```javascript
const matched = router.lookup('/hello/world');
// => { handlers: { post: [Function], delete: [Function] } }
const handler = matched.handlers['get'];
// => undefined
```

1行目の`lookup`メソッドによって返却される返り値は、`router.insert("/hello/world")`で登録した値になります。探索処理がどのように行われているのかradix3のソースコードを読むと、`insert`メソッドでデータを格納する時にPlaceholderやWildcardが使われていないパスを指定しない場合、`router`インスタンスのデータ構造に対してデータを格納しますが、`staticRoutesMap`というMapオブジェクトにもデータを追加します[^4]。

[^4]: [https://github.com/unjs/radix3/blob/main/src/router.ts#L155-L159](https://github.com/unjs/radix3/blob/742c7b0ac1bdaf72448462e0ea1f31918c533c5a/src/router.ts#L155-L159)

パスにPlaceholderやWildcardがない場合、データ構造を探索するよりもMapオブジェクトから取り出したほうが処理速度として早いため、このような処理が行われています。`lookup`メソッドの処理の一番最初に`staticRoutesMap`にパスが存在するかを確認していることがソースコードから分かります[^5]。

[^5]: [https://github.com/unjs/radix3/blob/main/src/router.ts#L40-L43](https://github.com/unjs/radix3/blob/742c7b0ac1bdaf72448462e0ea1f31918c533c5a/src/router.ts#L40-L43)

このことから、`router.insert("/hello/world")`から返却される返り値の中には、GETリクエストを処理するイベントハンドラはありません。H3サーバーとしては、該当するパスは存在するが処理をするためのイベントハンドラが存在しないのはイレギュラーなことで、これを解決する手段として、radix3は`router`インスタンスに格納された全てのパスを探索する`Matcher`を提供しています。H3はフォールバック処理として、この`Matcher`を使っています。

## Matcher

`Matcher`は、radix3が提供している`Router`をHTTPサーバーのルーティングとして使用する場合、先程説明した`Shadowed route`の問題が発生するため、別途提供されている機能になります。`Matcher`は、`Router`に登録された全てのパスを探索し、マッチするものを全て返す`matchAll`というメソッドを提供しています。
`Shadowed route`のサンプルコードの`router`インスタンスを`toRouteMatcher`関数に渡して初期化します。次のように`matchAll`メソッドに探索するパスを渡すことで結果を得ることができます。

```javascript
import { toRouteMatcher } from "radix3";

const matcher = toRouteMatcher(router);
const matches = matcher.matchAll("/hello/world");
console.log(matches);
// [
//   { path: '/hello/world', handlers: { post: [Function], delete: [Function] } },
//   { path: '/hello/**', handlers: { all: [Function] } }
// ]
```

## 調査しながら思ったこと

### ブラウザ側のルーティングはどうなっているか?

radix3自体は、UnJSの哲学に基づき、NuxtやVue.jsとの依存はありません。しかし、NuxtはHTTPサーバーのライブラリとしてH3を使っていて、その内部でradix3のルーターを使っています。そして、ブラウザ側で動くルーターライブラリに[Vue Router](https://router.vuejs.org/)を使っています。
Vue Routerのソースコードを簡単に読んでみると、あらかじめ想定していたとおり、radix3は使われていません。サーバーサイドのルーティングで求められる要件として、1台のサーバー辺りのスループットが重要になるため、高速に動作することが重要になりますが、ブラウザサイドでは、処理はユーザーの個々のブラウザ上で行われるため、radix3を使わなくても十分な速度で動作すると思います。
しかし、まだ調査が不十分なので推測で書きますが、Vue RouterとH3(radix3)のルーティングの記法で異なる仕様や挙動がある場合、サーバーサイドとブラウザサイドの違いを吸収するためにNuxtがどのような処理をしているのか気になりました。

### ライブラリの責務

radix3を調査してみて、H3とradix3がそれぞれどのように責務を分けるとより良いのかなと思いました。radix3は元々、`Radix Router`という元になったライブラリがあり、それに倣ってradix3がルーター機能を提供しています。そしてradix3が提供するルーター機能にH3のルーティングに必要な機能を拡張しているため、(今回の事情がなかったとしても必要な機能かもしれませんが、H3の`Router`の機能を満たすために`Matcher`が存在しているように見えています。
今回調査してみて、木構造以外にもスタック・キューやヒープ等の様々なデータ構造が存在することが分かりました。radix3で提供しているルーター機能をH3に移して、radix3はデータ構造全般を扱うライブラリにすると、責務としてスッキリし、尚且つUnJSの哲学との親和性もさらにあがるのかなと思いました。(その場合、radix3という名前を変える必要が出てきますが...)

## 参考

* [木構造 (データ構造) - Wikipedia](https://ja.wikipedia.org/wiki/木構造_(データ構造))
* [基数木 - Wikipedia](https://ja.wikipedia.org/wiki/基数木)
* [幅優先探索 - Wikipedia](https://ja.wikipedia.org/wiki/幅優先探索)
* [深さ優先探索 - Wikipedia](https://ja.wikipedia.org/wiki/深さ優先探索)
