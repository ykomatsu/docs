---
title: アプリケーション入門
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

# アプリケーション入門

<!--
This document explains how to make a very simple Pedestal
application.
-->

この文書では、非常に簡単なPedestalアプリケーションを作成する方法を説明します。

<!--
Most applications need to do the same basic things:
-->

多くのアプリケーションにおいて、求められる基本的なことは大体同じです。

<!--
* Receive and process input
* Manage state
* Update the UI when state changes
-->

* 入力を受け取り、処理する
* 状態を管理する
* 状態が変わったときにUIを更新する

<!--
Writing this kind of application in ClojureScript is straightforward.
-->

この種のアプリケーションをClojureScriptで書くのは簡単です。

```clojure
(ns hello-world
  (:require [domina :as dom]))

(def application-state (atom 0))

(defn render [old-state new-state]
  (dom/destroy-children! (dom/by-id "content"))
  (dom/append! (dom/by-id "content")
               (str "<h1>" new-state " Hello Worlds</h1>")))

(add-watch application-state :app-watcher
           (fn [key reference old-state new-state]
             (render old-state new-state)))

(defn receive-input []
  (swap! application-state inc)
  (.setTimeout js/window #(receive-input) 3000))

(defn ^:export main []
  (receive-input))
```

<!--
Here we have all of the basic parts that we need. There is an atom for
storing state and Clojure's update semantics for performing state
transitions. We can easily watch the atom for changes and call a
rendering function passing in the old and new state.
-->

これで私達は必要とする基本的な部品をすべて手に入れました。
状態を保存するためのアトムと、状態遷移を行うためのClojureにおける更新の意味が、そこにはあります。
私達は簡単にアトムの変化を監視し、元の状態と新しい状態を渡してレンダリング関数を呼び出すことができます。

<!--
If you have ever built a ClojureScript application, this is the way
you start.
-->

もし今までにClojureScriptアプリケーションを作成したことがあるのであれば、ここがあなたのスタート地点です。

<!--
As the application above gets more complex, many problems will
arise. How many atoms should we have? What is the structure of the
data that goes into the atoms? When we are handed an old and new
state, how do we figure out what has changed? How do we know if we
should care about a change?
-->

前述のアプリケーションは、複雑になるにつれてたくさんの問題を生むでしょう。
私達はいくつのアトムを用意しなければならないのでしょうか。
アトムに入れるデータの構造はどのようなものでしょうか。
元の状態と新しい状態を渡されたときに、何が変化したのかをどうやって把握すればよいのでしょうか。
ある変化について、それが処理すべきものかどうかをどうやって判断すればよいのでしょうか。

<!--
One of the largest problems with this approach is that the rendering
function is being asked to do a lot of work. It is handed an old and
new value and asked to render it. It has to figure out what has
changed and then figure out the state of the DOM so that it can make
the necessary change. There are only two ways to deal with this; both
are unacceptable.
-->

この手法の大きな問題の一つは、レンダリング関数に仕事を多く任せ過ぎているということです。
それは元の値と新しい値を受け取り、レンダリングを指示されます。
それは何が変化したのかを把握し、そして必要な変更を施すためにDOMの状態を把握しなければなりません。
これを行うには二つの方法しかありません。
そして両方とも受け入れられないものです。

<!--
The first is to look at the DOM and try to figure out what needs to be
modified, the other is to wipe out large sections of the DOM and
re-render everything. The first approach means that we put state in
the DOM. The second does not perform well.
-->

一つは、DOMを調べて必要な変更が何であるかを把握しようとする方法で、もう一つは、DOMの大部分を抹消して、すべてをレンダリングし直す方法です。
一つ目の手法は、状態をDOMの中に保持することを意味します。
二つ目ではうまく処理することができないでしょう。

<!--
Pedestal is designed to help us keep all state out of the DOM and
write applications which perform well.
-->

Pedestalはすべての状態をDOMの外に保持し、うまく処理できるアプリケーションを書く手助けをするよう設計されています。

## 最初のPedestalアプリケーション

<!--
The example below shows the same application written with Pedestal.
-->

次の例は、同じアプリケーションをPedestalを使って書いたものです。

```clojure
(ns hello-world
  (:require [io.pedestal.app.protocols :as p]
            [io.pedestal.app :as app]
            [io.pedestal.app.messages :as msg]
            [io.pedestal.app.render :as render]
            [domina :as dom]))

(defn count-model [old-state message]
  (condp = (msg/type message)
    msg/init (:value message)
    :inc (inc old-state)))

(defmulti render (fn [& args] (first args)))

(defmethod render :default [_] nil)

(defmethod render :value [_ _ old-value new-value]
  (dom/destroy-children! (dom/by-id "content"))
  (dom/append! (dom/by-id "content")
               (str "<h1>" new-value " Hello Worlds</h1>")))

(defn render-fn [deltas input-queue]
  (doseq [d deltas] (apply render d)))

(def count-app {:models {:count {:init 0 :fn count-model}}})

(defn receive-input [input-queue]
  (p/put-message input-queue {msg/topic :count msg/type :inc})
  (.setTimeout js/window #(receive-input input-queue) 3000))

(defn ^:export main []
  (let [app (app/build count-app)]
    (render/consume-app-model app render-fn)
    (receive-input (:input app))
    (app/begin app)))
```

<!--
This is a lot more code to do the same thing. What are the benefits of
this code over the example above?
-->

同じことをするために、もっとたくさんのコードが使われています。
前述の例と比べた場合の、このコードの利点は何でしょうか。

<!--
The first thing to notice is that there is no explicit state
management. There is no atom, no watcher and no swap!. The application
that is built in `main` manages state transitions. The `count-model`
function receives its old state and produces a new state without
having to produce side effects.
-->

最初に気付くことは、明示的な状態管理がないということです。
ここにはアトムはありませんし、ウォッチャーもswap!もありません。
`main`で作成されたアプリケーションが状態遷移を管理するのです。
`count-model`関数は、元の状態を受け取り、副作用を生むことなく新しい状態を生成します。

<!--
Because state transitions are handled by the application engine, most
of our application logic can be written as pure functions.
-->

状態遷移はアプリケーションエンジンによって処理されるので、私達のアプリケーションのロジックのほとんどは、純粋な関数として書くことができます。

<!--
The state is no longer a global thing that can be updated from
anywhere. All updates to the data model happen in one place. In this
small application, they take place within the `count-model` function.
-->

状態はもはや、どこからでも更新できるグローバルなものではありません。
データモデルのすべての更新は、一つの場所で起こります。
この小さなアプリケーションでは、それらは`count-model`関数の中で起こります。

<!--
The application data flow is described with a map, `count-app`. This
map is then used to build the application. The description of
functions and inputs have been separated from the execution strategy.
This separation provides many benefits:
-->

アプリケーションのデータの流れは`count-app`というマップを使って記述します。
このマップはアプリケーションを作成するために使用されます。
関数と入力の記述は、実行計画とは分離されています。
この分離には、たくさんの利点があります。

<!--
* multiple execution strategies for the same set of functions
* amazing debugging tools
* easy to test
* easy to reuse
* forces better design
-->

* 同じ関数の組に対する複数の実行計画
* すばらしいデバッグツール
* テストしやすい
* 再利用しやすい
* よりよいデザインの強制

<!--
This separation forces developers to think about functions
independent from execution order.
-->

この分離によって開発者は、実行順序からの関数の独立について考えざるを得なくなります。

<!--
In the code above, renderers receive deltas instead of two
states. Because renderers know exactly what has changed they can
usually be written as very simple functions.
-->

前述のコードでは、レンダラーは二つの状態の代わりにデルタを受け取ります。
レンダラーは何が変化したかを正確に知っているので、それらは普通、非常に簡単な関数として書くことができます。

## Pedestalのプッシュレンダラーの利用

<!--
In the example above, the rendering code is doing all the work of
dispatching deltas to the correct rendering functions. This shows that
it is easy to do this kind of thing yourself. Pedestal includes a
rendering function implementation which is helpful when writing
applications where the rendering is controlled by pushing changes out
to the renderer from the underlying models. This is implemented in
`io.pedestal.app.render.push`.
-->

前述の例では、デルタを適切なレンダリング関数に割り振る仕事はすべてレンダリングのコードが行なっています。
つまり、このようなことを自分でするのは簡単だということです。
Pedestalは、基礎にあるモデルからレンダラーへと変更をプッシュすることによってレンダリングを制御するアプリケーションを書くときに便利な、レンダリング関数の実装を用意しています。
これは、`io.pedestal.app.render.push`に実装されています。

<!--
The example below shows the same application using the push renderer.
-->

次の例は、プッシュレンダラーを使った同じアプリケーションです。

```clojure
(ns hello-world
  (:require [io.pedestal.app.protocols :as p]
            [io.pedestal.app :as app]
            [io.pedestal.app.messages :as msg]
            [io.pedestal.app.render :as render]
            [io.pedestal.app.render.push :as push]
            [domina :as dom]))

(defn count-model [old-state message]
  (condp = (msg/type message)
    msg/init (:value message)
    :inc (inc old-state)))

(defn render-value [renderer [_ _ old-value new-value] input-queue]
  (dom/destroy-children! (dom/by-id "content"))
  (dom/append! (dom/by-id "content")
               (str "<h1>" new-value " Hello Worlds</h1>")))

(def count-app {:models {:count {:init 0 :fn count-model}}})

(defn receive-input [input-queue]
  (p/put-message input-queue {msg/topic :count msg/type :inc})
  (.setTimeout js/window #(receive-input input-queue) 3000))

(defn ^:export main []
  (let [app (app/build count-app)
        render-fn (push/renderer "content" [[:value [:*] render-value]])]
    (render/consume-app-model app render-fn)
    (receive-input (:input app))
    (app/begin app)))
```

<!--
There are two big differences in this implementation: all dispatching
is handled by the provided rendering function and the rendering
handler, `render-value`, receives a renderer object and the
input-queue. The renderer can be used to help in mapping changes to the
DOM and the input-queue is used to send messages back to the
application.
-->

この実装には二つの大きな違いがあります。
すべての割り振りはあらかじめ用意されたレンダリング関数によって処理されるということと、`render-value`というレンダリングハンドラーはレンダラーオブジェクトと入力キューを受け取るということです。
レンダラーは、変更をDOMにマッピングするために便利に使うことができ、入力キューはメッセージをアプリケーションに送信するのに使われます。

<!--
The renderer is configured to send all `:value` changes to the
`render-value` function.
-->

レンダラーはすべての`:value`変更を`:render-value`関数に送信するように設定されています。

## テンプレートの利用

<!--
The only remaining problem with this code is that we have included a
lot of specific HTML and formatting in with the Clojure code. It would
be good to extract this to a template.
-->

このコードに残っている最後の問題は、たくさんの具体的なHTMLがClojureのコードの中に埋め込まれていることです。
これはテンプレートに展開した方がよさそうです。

<!--
The example below shows the same application using templates to both
generate and update HTML.
-->

次の例は、同じアプリケーションについて、HTMLの生成と更新の両方でテンプレートを使うようにしたものです。

```clojure
(ns hello-world
  (:require [io.pedestal.app.protocols :as p]
            [io.pedestal.app :as app]
            [io.pedestal.app.messages :as msg]
            [io.pedestal.app.render :as render]
            [io.pedestal.app.render.push :as push]
            [domina :as dom])
  (:require-macros [hello-world.html-templates :as html-templates]))

(def templates (html-templates/hello-world-templates))

(defn count-model [old-state message]
  (condp = (msg/type message)
    msg/init (:value message)
    :inc (inc old-state)))

(defn render-page [renderer [_ path] input-queue]
  (let [parent (push/get-parent-id renderer path)
        html (templates/add-template renderer path (:hello-world-page templates))]
    (dom/append! (dom/by-id parent) (html {:message ""}))))

(defn render-value [renderer [_ path old-value new-value] input-queue]
  (templates/update-t renderer path {:message (str new-value)}))

(def render-config
  [[:node-create [:*] render-page]
   [:value       [:*] render-value]])

(def count-app {:models {:count {:init 0 :fn count-model}}})

(defn receive-input [input-queue]
  (p/put-message input-queue {msg/topic :count msg/type :inc})
  (.setTimeout js/window #(receive-input input-queue) 3000))

(defn ^:export main []
  (let [app (app/build count-app)
        render-fn (push/renderer "content" render-config)]
    (render/consume-app-model app render-fn)
    (receive-input (:input app))
    (app/begin app)))

```

<!--
In this example, we have extracted the render-config and we now
respond to two kinds of updates: `:node-create` and `:value`. When we
receive a `:node-create` delta, we render the page using a
template. When we receive a `:value` delta, we simply update the value
in the existing template.
-->

この例では、render-configを展開し、2種類の更新を受け取るようになっています。
`:node-create`と`:value`です。
`:node-create`デルタを受け取ったときは、テンプレートを利用してページをレンダリングします。
`:value`デルタを受け取ったときは、既にレンダリングされたテンプレートの中の値を更新するだけです。
