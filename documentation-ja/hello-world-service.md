---
Title: Hello Worldサービス
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

# Hello Worldサービス

<!--
This explains how to create a simple "Hello World" service on
Pedestal. This is the simplest ever service. As you can easily guess,
the service just responds with "Hello, World!" whenever you need a
friendly greeting.
-->

ここでは、簡単な"Hello World"サービスをPedestalで作成する方法を説明します。
これは最も簡単なサービスです。
容易に想像できるように、このサービスは親しみのある挨拶が必要なときにいつでも"Hello, World!"を返してくれるというだけのものです。

## サーバー側のClojureプロジェクトの作成

```
mkdir ~/tmp
cd ~/tmp
lein new pedestal-service helloworld
cd helloworld
```

## project.cljの編集

<!--
The generated project definition looks like this:
-->

生成されたプロジェクトの定義はこのようになります。

```clojure
(defproject helloworld "0.1.0-SNAPSHOT"
  :description "FIXME: write description"
  :url "http://example.com/FIXME"
  :license {:name "Eclipse Public License"
            :url "http://www.eclipse.org/legal/epl-v10.html"}
  :dependencies [[org.clojure/clojure "1.5.0"]
                 [io.pedestal/pedestal.service "0.1.0"]

                 ;; Remove this line and uncomment the next line to
                 ;; use Tomcat instead of Jetty:
                 [io.pedestal/pedestal.jetty "0.1.0"]
                 ;; [io.pedestal/pedestal.tomcat "0.1.0"]

                 ;; Logging
                 [ch.qos.logback/logback-classic "1.0.7"]
                 [org.slf4j/jul-to-slf4j "1.7.2"]
                 [org.slf4j/jcl-over-slf4j "1.7.2"]
                 [org.slf4j/log4j-over-slf4j "1.7.2"]]
  :profiles {:dev {:source-paths ["dev"]}}
  :resource-paths ["config"]
  :main ^{:skip-aot true} helloworld.server)
```

<!--
You may want to change the description, add dependencies, change the
license, or whatever else you'd normally do to project.clj. Once you
finish editing the file, run `lein deps` to fetch any jars you need.
-->

あなたは説明を変更したり、依存関係を追加したり、ライセンスを変更したり、その他あなたが普段project.cljに対してしていることは何でもすることができます。
ファイルの編集が終わったら、必要なjarファイルを取得するために`lein deps`を実行してください。

## service.cljの編集

<!--
Our project name is helloworld, so the template generated two files
under `src/helloworld`. `service.clj` defines the logic of our 
service. `server.clj` creates a server (a daemon) to host that
service.
-->

私達のプロジェクトの名前はhelloworldなので、テンプレートは`src/helloworld`の中に二つのファイルを生成しました。
`service.clj`は私達のサービスのロジックを定義します。
`server.clj`はそのサービスを提供するサーバー（デーモン）を作成します。

<!--
Of course, if you used a different project name, your service.clj
would be src/your-project-name-here/service.clj. Also, the namespace
will be your-project-name-here.service instead of `helloworld.service`.
-->

もちろん、もしあなたが違うプロジェクト名を使っていれば、あなたのservice.cljはsrc/あなたのプロジェクト名/service.cljにあるでしょう。
また、名前空間も`helloworld.service`ではなくて、あなたのプロジェクト名.serviceとなっているでしょう。

<!--
The default service.clj demonstrates a few things, but for now let's
replace the default service.clj with the smallest example that will
work. Edit src/helloworld/service.clj until it looks like this:
-->

初期設定のservice.cljはいくつかのことを実演しますが、とりあえず初期設定のservice.cljを動作する最小の例に置き換えておきましょう。
src/helloworld/service.cljをこのように編集してください。

```clojure
(ns helloworld.service
    (:require [io.pedestal.service.http :as bootstrap]
              [io.pedestal.service.http.route.definition :refer [defroutes]]
              [ring.util.response :refer [response]]))

(defn home-page
  [request]
  (response "Hello World!"))

(defroutes routes
  [[["/" {:get home-page}]]])

;; Consumed by helloworld.server/create-server
(def service {:env :prod
              ::bootstrap/routes routes
              ::bootstrap/type :jetty
              ::bootstrap/port 8080})
```

<!--
The `home-page` function defines the simplest HTTP response to the
browser. In `routes`, we map the URL `/` so it will invoke
`home-page`. Finally, the function `service` describes how to hook
this up to a server. Notice that this just returns a map. `service`
doesn't actually start the server up; it defines how the service will
look when it gets started later.
-->

`home-page`関数はブラウザーへの最も簡単なHTTPレスポンスを定義します。
`routes`では、私達は`/`URLを`home-page`の呼出しにマッピングします。
最後に、`service`関数はこれをサーバーにつなぐ方法を記述します。
これは単にマップを返すだけであるということに注意してください。
`service`は実際にサーバーを開始させるわけではありません。
それはサービスが、後でそれがいつ開始するかを調べる方法を定義します。

<!--
There's nothing magic about these function names. There are no
required names here. One of our design principles in Pedestal is that
all the connections between parts of your application should be
_evident_. You should be able to trace functions from call to
definition without any "magic" or "action at a distance"
metaprogramming.
-->

これらの関数の名前について、魔術は特にありません。
ここでは、必要な名前というものはありません。
Pedestalにおける私達の設計原則の一つは、アプリケーションの部品の間の接続はすべて_明白_でなければならないというものです。
「魔術」や「遠隔作用」のようなメタプログラミングを使うことなく、関数を呼出しから定義まで追跡することができるべきです。

<!--
Take a peek into `src/helloworld/server.clj`. We won't be changing it,
but it's interesting to look at the create-server function:
-->

`src/helloworld/server.clj`を覗いてみてください。
私達はそれを変更するつもりはありませんが、create-server関数を調べるのは面白いです。

``` clojure
(ns helloworld.server
  (:require [helloworld.service :as service]
            [io.pedestal.service.http :as bootstrap]))

;; ...

(defn create-server
  "Standalone dev/prod mode."
  [& [opts]]
  (alter-var-root #'service-instance
                  (constantly (bootstrap/create-server (merge service/service opts)))))

;; ...

```

<!--
You can see that `create-server` calls `helloworld.service/service` to
get that map we just looked at. `create-server` merges that map with
any per-invocation options, and then creates the actual server by
calling `bootstrap/create-server`.
-->

あなたは、私達が先程見たマップを取得するために、`create-server`が`helloworld.service/service`を呼び出していることに気が付くかもしれません。
`create-server`はそのマップと呼出しごとのオプションとをマージして、そして`bootstrap/create-server`を呼び出すことで実際のサーバーを作成します。

## 開発モードでの実行

<!--
We'll start the server from a repl, which is how we will normally run in development mode.
-->

私達はreplからサーバーを開始しようとしていますが、それは普通、開発モードで実行する方法です。

```
$ lein repl

nREPL server started on port 52471
REPL-y 0.1.6
Clojure 1.5.0
    Exit: Control+D or (exit) or (quit)
Commands: (user/help)
    Docs: (doc function-name-here)
          (find-doc "part-of-name-here")
  Source: (source function-name-here)
          (user/sourcery function-name-here)
 Javadoc: (javadoc java-object-or-class-here)
Examples from clojuredocs.org: [clojuredocs or cdoc]
          (user/clojuredocs name-here)
          (user/clojuredocs "ns-here" "name-here")
```

<!--
To make life easier in the repl, pedestal generated a "dev.clj" file with some convenience functions. We'll use one to start the server:
-->

replでの作業を楽にするために、pedestalはいくつかの便利な関数を含む"dev.clj"ファイルを生成しています。
私達はサーバーを開始するためにそのうちの一つを使うでしょう。

```clojure
helloworld.server=> (use 'dev)
nil
helloworld.server=> (start)
nil

```

<!--
Now let's see "Hello World!"
-->

それでは"Hello World!"を見てみましょう。

<!--
Go to [http://localhost:8080/](http://localhost:8080/)  and you'll see a shiny "Hello World!" in your browser.
-->

[http://localhost:8080/](http://localhost:8080/)に行けば、あなたのブラウザー上で輝く"Hello World!"を見ることができるでしょう。

<!--
Done! Let's stop the server.
-->

以上です！
サーバーを停止させましょう。

```
helloworld.server=> (stop)
nil
```

## 次の一歩

<!--
For more about building out the server side, you can look at
[Routing and Linking](/documentation/service-routing/) or
[Connecting to Datomic](/documentation/connecting-to-datomic/).
-->

サーバー側の構築についてもっと知りたいのであれば、[ルートとリンク](/documentation/service-routing/)や[Datomicへの接続](/documentation/connecting-to-datomic/)を見てください。
