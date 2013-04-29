---
title: 'Hello World'のDatomicへの接続
---

<!--
 Copyright 2013 Relevance, Inc.

 The use and distribution terms for this software are covered by the
 Eclipse Public License 1.0 (http://opensource.org/licenses/eclipse-1.0)
 which can be found in the file epl-v10.html at the root of this distribution.

 By using this software in any fashion, you are agreeing to be bound by
 the terms of this license.

 You must not remove this notice, or any other, from this software.
-->

# "Hello World"のDatomicへの接続

<!--
This tutorial extends the
[Hello World Service](/documentation/hello-world-service/) to use
strings retrieved from [Datomic]. We will start from the "helloworld"
service that we created in our first tutorial.
-->

このチュートリアルでは、[Datomic]から取得した文字列を利用するように[Hello Worldサービス](/documentation/hello-world-service/)を拡張します。
最初のチュートリアルで作成した"helloworld"サービスから始めることにしましょう。

<!--
This tutorial assumes that you have Datomic running already. If not,
hop over to the
[download site](http://www.datomic.com/get-datomic.html) and download
the free edition.
-->

このチュートリアルは、あなたが既にDatomicを動かしていることを前提にしています。
もしまだであれば、[ダウンロードサイト](http://www.datomic.com/get-datomic.html)に行って、無料版をダウンロードしてください。

## プロジェクトへのDatomicの追加

<!--
There are just a couple of steps to get our service hooked up to
Datomic. First, we need to add the dependency to our project.clj.
-->

私達のサービスをDatomicに接続するには、いくつかの段階があります。
最初に、私達は自分のproject.cljに依存関係を追加しなければなりません。

```
(defproject helloworld "0.1.0-SNAPSHOT"
  :description "FIXME: write description"
  :url "http://example.com/FIXME"
  :license {:name "Eclipse Public License"
            :url "http://www.eclipse.org/legal/epl-v10.html"}
  :dependencies [[org.clojure/clojure "1.5.0"]
                 [io.pedestal/pedestal.service "0.1.0"]
                 [org.slf4j/jul-to-slf4j "1.7.2"]
                 [org.slf4j/jcl-over-slf4j "1.7.2"]
                 [org.slf4j/log4j-over-slf4j "1.7.2"]
                 [com.datomic/datomic-free "0.8.3826"]]
  :profiles {:dev {:source-paths ["dev"]}}
  :resource-paths ["config"]
  :main ^{:skip-aot true} helloworld.server)
```

<!--
Run `lein deps` to fetch any jars you need.
-->

必要なjarファイルを取得するために`lein deps`を実行してください。

## スキーマと種となるデータの作成

<!--
On Pedestal app, a suitable place to put a schema is the
resources/[your-project-name-here] directory. In this sample, the project
name is "helloworld", so place the Datomic schema file below in
`resources/helloworld/schema.edn`. The schema is also pretty simple:
it has just one attribute.
-->

Pedestalアプリケーションにおいて、スキーマを配置するのに適切な場所はresources/[あなたのプロジェクト名]ディレクトリーです。
この例では、プロジェクト名は"helloworld"なので、Datomicのスキーマファイルを`resources/helloworld/schema.edn`に配置してください。
そのスキーマは非常に単純です。
それは一つの属性しか持っていません。

```clj
[
  {:db/id #db/id[:db.part/db]
  :db/ident :hello/color
  :db/valueType :db.type/string
  :db/cardinality :db.cardinality/one
  :db/fulltext true
  :db/doc "Today's color"
  :db.install/_attribute :db.part/db}
]
```

<!--
This application doesn't have any way to write new data into Datomic
yet. So, in order to have something to show in a browser, we'll put
some seed data into Datomic. This file can also reside under the
resources/helloworld directory. Create the file
`resources/helloworld/seed-data.edn` with the following contents.
-->

このアプリケーションは新しいデータをDatomicに書き込む手段をまだ持っていません。
そのため、ブラウザーに表示するものを用意するために、いくつかの種となるデータをDatomicに入れておくことにしましょう。
このファイルもresources/helloworldディレクトリーに配置することができます。
次の内容で`resources/helloworld/seed-data.edn`ファイルを作成してください。

```clj
[
{:db/id #db/id[:db.part/user -1], :hello/color "True Mint"}
{:db/id #db/id[:db.part/user -2], :hello/color "Yellowish White"}
{:db/id #db/id[:db.part/user -3], :hello/color "Orange Red"}
{:db/id #db/id[:db.part/user -4], :hello/color "Olive Green"}
]
```

<!--
The funny looking negative IDs are a way to ask Datomic to assign
entity IDs automatically.
-->

負の値をIDにしている点が変わっていますが、これはDatomicに要素のIDを自動的に割り振らせるための方法です。

## いくつかのデータ関数の作成

<!--
Now we need create functions to establish a connection to Datomic,
define the schema, and retrieve data.  This code is nothing new, just
a simple Datomic sample. Put the following code into
`src/helloworld/peer.clj`.
-->

さて、私達はDatomicとの接続を確立し、スキーマを定義し、データを取得するための各関数を作成しなければなりません。
このコードは何ら目新しいものではなく、単純なDatomicの例に過ぎません。
次のコードを`src/helloworld/peer.clj`に配置してください。

```clj
(ns helloworld.peer
  (:require [datomic.api :as d :refer (q)]))

(def uri "datomic:mem://helloworld")

(def schema-tx (read-string (slurp "resources/helloworld/schema.edn")))
(def data-tx (read-string (slurp "resources/helloworld/seed-data.edn")))

(defn init-db []
  (when (d/create-database uri)
    (let [conn (d/connect uri)]
      @(d/transact conn schema-tx)
      @(d/transact conn data-tx))))

(defn results []
  (init-db)
  (let [conn (d/connect uri)]
    (q '[:find ?c :where [?e :hello/color ?c]] (d/db conn))))

```

# サービスにおけるデータベースの結果の利用

<!--
Next, we need to use results from the database. For this, we will add
a function to `service.clj`. This new function will use `peer.clj` to
access data.
-->

次に、私達はデータベースからの結果を利用しなければなりません。
このために、関数を`service.clj`に追加することにしましょう。
この新しい関数はデータにアクセスするために`peer.clj`を使います。

<!--
Open up `src/helloworld/service.clj` again and modify the `ns` macro to
reference helloworld.peer:
-->

`src/helloworld/service.clj`をもう一度開き、helloworld.peerを参照するように`ns`マクロを変更してください。

```clj
(ns helloworld.service
    (:require [io.pedestal.service.http :as bootstrap]
              [io.pedestal.service.http.route.definition :refer [defroutes]]
              [ring.util.response :refer [response]]
              [helloworld.peer :as peer :refer [results]]))
```

<!--
Let's now rewrite the `home-page` function in `service.clj` so that we
see the output from Datomic.
-->

それでは、私達がDatomicからの出力を見ることができるように`service.clj`の中の`home-page`関数を書き換えましょう。

```clj
(defn home-page
  [request]
  (response (str "Hello Colors! " (results))))
```

<!--
If you still have the service running from
[Hello World Service](/documentation/hello-world-service/), then you
will need to exit the REPL. Restart the service the same way as
before: `lein repl`, `(use 'dev)`, and `(start)`.
-->

もしあなたがまだ[Hello Worldサービス](/documentation/hello-world-service/)のサービスを実行したままであれば、あなたはREPLを終了しなければなりません。
前と同じ方法でサービスをもう一度開始してください。
`lein repl`、`(use 'dev)`、そして`(start)`です。

<!--
Now point your browser at
[http://localhost:8080/](http://localhost:8080) and you will see the
thrilling string:
-->

これで、あなたのブラウザーで[http://localhost:8080/](http://localhost:8080)にアクセスすれば、あなたはスリリングな文字列を目にするはずです。

```clj
Hello Colors! [["True Mint"], ["Olive Green"], ["Orange Red"], ["Yellowish White"]]
```

<!--
Because `home-page` returns a string, the HTTP response will be sent
with a content type of "text/plain", as we can see by using "curl" to
access the server.
-->

`home-page`は文字列を返すので、私達がサーバーにアクセスするのに"curl"を使うことによって分かるように、HTTPレスポンスは"text/plain"というコンテンツタイプで送られます。

``` bash
$ curl -i http://localhost:8080/
HTTP/1.1 200 OK
Date: Fri, 22 Feb 2013 20:31:06 GMT
Content-Type: text/plain
Content-Length: 82
Server: Jetty(8.1.9.v20130131)

Hello Colors! [["True Mint"], ["Orange Red"], ["Olive Green"], ["Yellowish White"]]
```

# 次の一歩

<!--
For more about Datomic, check out [datomic.com][datomic].
-->

Datomicについてもっと知りたいのであれば、[datomic.com][datomic]を見てください。

[datomic]: http://www.datomic.com

