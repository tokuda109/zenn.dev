---
title: "UnJSについて"
emoji: "😍"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["nuxtjs", "nuxt3", "unjs"]
published: true
publication_name: "comm_vue_nuxt"
---

[Nuxt](https://nuxt.com/)は、ユニバーサルな[Vue.js](https://vuejs.org/)アプリケーションを構築するためのフレームワークです。
Nuxtは、UnJSというプロジェクトによって提供されている様々なライブラリを使って構築されています。Nuxtを深く知るためには、その背後にあるUnJSのライブラリがどのような機能を提供しているのか、Nuxtがそれらをどのように組み合わせて構築されているのかを理解する必要があります。

## シリーズ記事

Vue.jsのバージョン2系のサポートは2023年12月31日に終了し、Nuxtのバージョン2系も2024年6月30日にサポートが終了する予定です[^1][^2]。これからNuxtやVue.jsを使う開発者(私を含め)は、バージョン3系の新しいナレッジを知る必要があります。

勿論NuxtやVue.jsの学習も同時並行で行いますが、UnJSの主要なライブラリを押さえておくことで、Nuxtを用いた開発をスムーズに行えるのではないかと考えました。Nuxtのバージョン3系は開発がアクティブで機能が頻繁に追加・更新されます。またドキュメンテーションで全ての情報を知ることができる訳ではないため、UnJSの主要なライブラリの知識は無駄になることはないと考えています。

しかし、UnJSのライブラリは数が多く、1つの記事で全てのライブラリを説明するのは難しいため、シリーズ記事として分割することにします。

[^1]: [Vue 2 LTS, EOL & Extended Support](https://v2.vuejs.org/lts/)
[^2]: [Nuxt 2 LTS, EOL & Extended Support](https://v2.nuxt.com/lts/)

### 調査予定のライブラリ

現時点でシリーズを通して、次のライブラリを取り上げて調査する予定です。

:::message
個人的な関心や調査の必要性に基づいて対象のライブラリを選定するため、記事を公開していく過程で予定が変更される可能性があります
:::

ライブラリ | 機能(スローガン) | 記事
---|---|---
H3 | 軽量で高速なHTTPサーバー | [H3 (unjs/h3)について](./introducing-unjs-h3)
radix3 | 基数木に基づいて実装された高速なルーター | [radix3 (unjs/radix3)について](./introducing-unjs-radix3)
unenv | 特定の実行環境に依存しないユニバーサルなJavaScriptコードに変換するライブラリ | [unenv (unjs/unenv)について](./introducing-unjs-unenv)
Nitro | どの環境でも動作させることが可能なWebサーバーを構築するライブラリ | Nitro (unjs/nitro)について
ofetch | APIクライアント | ofetch (unjs/ofetch)について
Consola | console | Consola (unjs/consola)について

ここに記載していないライブラリはたくさんあり、どのようなライブラリがあるのか知りたい方は「UnJS にどんなツールがあるのか、上位30件すべて紹介してみた ([前編](https://zenn.dev/ytr0903/articles/c6c42147ed29be) / [後編](https://zenn.dev/ytr0903/articles/6b50bf790c340b))」という記事を一度目を通すことをお勧めします。

それでは前置きが長くなってしまいましたがUnJSのライブラリの紹介は次の記事から紹介し、この記事はUnJSというプロジェクト自体がどのようなものなのかを紹介します。

## UnJSとは?

[UnJS](https://unjs.io/)は*Unified JavaScript Tools*の略で、JavaScriptの開発をより効率的かつ柔軟に行うために設計された、一連のオープンソースライブラリおよびツールを提供しているプロジェクトです。
UnJSで提供されているライブラリは50個以上あり、HTTPサーバー、ロガー、APIリクエスト等のウェブ開発でサードパーティライブラリとして一般的に用いられるものが提供されていて、[こちら](https://unjs.io/packages)のページで全ライブラリを確認することができます。
これらのライブラリは、Nuxtのコアチームやコミュニティにより開発・保守されていますが、**モジュール性が高く再利用可能なことから、Nuxt以外の様々なプロジェクトやユースケースに適応できる**ように作られています。

![UnJSのトップページのスクリーンショット](/images/unjs-website.png)
*プロジェクトのスローガンで「Unleash JavaScript's Potential with the UnJS Ecosystem」と示されているとおり、UnJSによってJavaScript開発を後押しすることを目的としていることが分かります*

次の動画で、UnJSのライブラリのいくつかはNuxtから切り出されたものもあったみたいですが、Nuxtの依存を完全に廃し、ウェブの開発で一般的に共通化できる課題に対してライブラリ化していることが説明されています。

https://www.youtube.com/watch?v=7kWuEFW9hP4

### 運用方針

[UnJS Governance Guide](https://github.com/unjs/governance)で、UnJSプロジェクトの運用方針について述べられていて、ライブラリは[UNIX哲学](https://ja.wikipedia.org/wiki/UNIX哲学)に従って特定の目的のためだけに開発され、その作者やメンテナによって保守されます。

そしてUnJSプロジェクトのゴールとして次のものがあります:

* **高品質で単一目的のJavaScriptユーティリティ**: 各ライブラリは単一目的・特定の課題解決をするためのスコープに限定した機能提供をしています。しかし、各ライブラリは協調性があり、組み合わせることでより大きい課題を解決することもできるように設計されています。
* 良くメンテナンスされたレポジトリによる健全なエコシステム
* 既知かつ信頼性のある解決策の集まり
* **協力的で歓迎的なエコシステム**: コントリビューターからのPull Requestを歓迎しているようです。各ライブラリのIssuesを見ると「PR more than welcome」というコメントをよく見かけます。
* **様々なエコシステムで同様な動作をするための一貫性と互換性**: Node.js以外の様々な環境でも同様に動かすことができたり、どのようなプラットフォームかを意識せずにライブラリを使うことができます。
* **JavaScriptエコシステムの大部分に利益をもたらすための独立した開発**: Nuxtチームと密接な関係はありますが、Nuxtからの依存や影響はなく、UnJSで独立した開発・運用をしています。

Pooya Parsa([@pi0](https://twitter.com/_pi0_))氏はUnJSのリード開発者で、UnJS全体のディレクションをしますが、各ライブラリの開発・保守や運営はライブラリ毎のメンテナに任せられています。各ライブラリに運営の自由度は与えられていますが、上記のUnJSの運営方針やゴールに沿った運用が求められます。

また、UnJSのメンバーの1人であるNozomu Ikuta([@NozomuIkuta](https://twitter.com/NozomuIkuta))氏による「Deep Dive to UnJS and Nuxt 3」というプレゼンテーションにおけるUnJSの紹介は、UnJSの歴史や運営方針を知るうえで参考になるため、是非見てください。

@[speakerdeck](7d5edc3ef4ff462a8b840fb3de1ea107)

### ウェブサイト

UnJSのウェブサイトはパッケージ一覧とブログ記事がコンテンツとしてありますが、2024年からはコンテンツを増やしていくそうです[^3]。チュートリアルやライブラリを使ったプロジェクト作成等がコンテンツとして公開されるみたいです。

また、2024年からはブログを毎月更新していく方針で、最新の開発状況やリリース報告等がブログ記事として公開される予定です。UnJSプロジェクトの状況を把握したい場合は、[RSS](https://unjs.io/rss)を登録するか、[@unjsio](https://twitter.com/unjsio)をフォローすると最新の情報を受け取れます。

[^3]: [A New Home to Drive a Vision](https://unjs.io/blog/2023-12-11-a-new-home-to-drive-a-vision)

### 参考動画

YouTubeでUnJSについて紹介している動画がいくつかあるので、紹介します:

* [UnJS: Nuxt 3 behind the scenes by Pooya Parsa: Nuxt Nation 2022](https://www.youtube.com/watch?v=8c5sNjdkEpU)
* [Alexander Lichter: unjs - Unified JavaScript solutions – vuejs.de Conf 2022](https://www.youtube.com/watch?v=zrSmzD9VH6A)
* [Introducing unjs - VueConf US 2023](https://www.youtube.com/watch?v=jc-42ZtaD_k)
* [Pooya Parsa | Unifying Web Development: Nitro and UnJS | ViteConf 2023](https://www.youtube.com/watch?v=BELNwXNl7bo)

## 変更履歴

* 2024/02/01: 記事が読みやすくなるように微調整しました
