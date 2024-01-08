---
title: "UnJSについて"
emoji: "😍"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["nuxtjs", "nuxt3", "unjs"]
published: true
---

## シリーズ記事の概要

[Nuxt](https://nuxt.com/)は、ユニバーサルな[Vue.js](https://vuejs.org/)アプリケーションを構築するためのフレームワークです。
Nuxtは、UnJSというプロジェクトによって提供されている様々なライブラリを使って構築されています。Nuxtを深く知るためには、その背後にあるUnJSのライブラリがどのような機能を提供しているのか、Nuxtがそれらをどのように組み合わせているのかを理解する必要があります。

UnJSのライブラリは数が多く、1つの記事で全てを説明するのは難しいため、シリーズ記事として分割することにしました。
まず初回となる今回の記事では、UnJSというプロジェクトがどのようなものなのかを紹介します。

:::message
シリーズ記事を通してNuxt、Vue.jsという表記を使いますが、どちらもバージョンは3系を使うことを前提としています。
:::

## UnJSとは?

[UnJS](https://unjs.io/)は*Unified JavaScript Tools*の略で、JavaScriptの開発をより効率的かつ柔軟に行うために設計された、一連のオープンソースライブラリおよびツールを提供しているプロジェクトです。
UnJSで提供されているライブラリは50個以上あり、HTTPサーバー、ロガー、APIリクエスト等のウェブ開発でサードパーティライブラリとして一般的に用いられるものが提供されていて、[こちら](https://unjs.io/packages)のページで全ライブラリを確認することができます。
これらのライブラリは、Nuxtのコアチームやコミュニティにより開発・保守されていますが、**モジュール性が高く再利用可能なことから、Nuxt以外の様々なプロジェクトやユースケースに適応できる**ようになっています。

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

Pooya Parsa([@pi0](https://twitter.com/_pi0_))氏はUnJSのリード開発者で、UnJS全体のディレクションをしますが、各ライブラリの開発・保守や運営はライブラリ毎のメンテナに任せられています。各ライブラリに運営の自由度は与えられていますが、上述のUnJSの運営方針やゴールに沿った運用が求められます。

また、UnJSのメンバーの1人であるNozomu Ikuta([@NozomuIkuta](https://twitter.com/NozomuIkuta))氏による「Deep Dive to UnJS and Nuxt 3」というプレゼンテーションにおけるUnJSの紹介もUnJSの歴史や運営方針を知るうえで参考になるため、是非見てください。

@[speakerdeck](7d5edc3ef4ff462a8b840fb3de1ea107)

### ウェブサイト

UnJSのウェブサイトはパッケージ一覧とブログ記事がコンテンツとしてありますが、2024年からはコンテンツを増やしていくそうです[^1]。チュートリアルやライブラリを使ったプロジェクト作成等がコンテンツとして公開されるみたいです。

また、2024年からはブログを毎月更新していく方針で、最新の開発状況やリリース報告等がブログ記事として公開される予定です。UnJSプロジェクトの状況を把握したい場合は、[RSS](https://unjs.io/rss)を登録するか、[@unjsio](https://twitter.com/unjsio)をフォローすると最新の情報を受け取れます。

[^1]: [A New Home to Drive a Vision](https://unjs.io/blog/2023-12-11-a-new-home-to-drive-a-vision)

## 調査予定のライブラリ

シリーズを通して、現時点で次のライブラリを取り上げる予定で、順番に調査した上で記事として公開していきます。(個人的な関心や調査の必要性に基づいて対象のライブラリを選定するため、記事を公開していく過程で予定が変更される可能性があります)

1. H3: Nuxtの[Server](https://nuxt.com/docs/getting-started/server)の実装でH3の関数を用いるため
2. radix3: H3の高速なルーティングの理由とどのようなルーティングの機能が使えるのか理解するため
3. unenv
4. Nitro
5. ofetch
6. Consola
7. ufo

UnJSの主要なライブラリの概要を知りたい場合は、「UnJS にどんなツールがあるのか、上位30件すべて紹介してみた ([前編](https://zenn.dev/ytr0903/articles/c6c42147ed29be) / [後編](https://zenn.dev/ytr0903/articles/6b50bf790c340b))」という記事や先ほどの「Deep Dive to UnJS and Nuxt 3」のスライドを参照してください。

## 参考

* [What is unJS long](https://www.youtube.com/watch?v=7kWuEFW9hP4)
* [UnJS: Nuxt 3 behind the scenes by Pooya Parsa: Nuxt Nation 2022](https://www.youtube.com/watch?v=8c5sNjdkEpU)
* [Alexander Lichter: unjs - Unified JavaScript solutions – vuejs.de Conf 2022](https://www.youtube.com/watch?v=zrSmzD9VH6A)
* [Introducing unjs](https://www.vuemastery.com/conferences/vueconf-us-2023/introducing-unjs/)
