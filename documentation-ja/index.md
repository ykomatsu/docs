---
title: Pedestalドキュメント
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

## Pedestalとは何か

<!--
Pedestal is a collection of
interacting libraries that together create a pathway for developing
a specific kind of application. It empowers developers to use
Clojure to build internet applications requiring real-time
collaboration and targeting multiple platforms.
-->

Pedestalは、特定の種類のアプリケーションを開発する道筋を作るための相互作用するライブラリーの集まりです。
Pedestalによって、開発者はリアルタイムコラボレーションの求められる、複数のプラットフォームを対象とするインターネットアプリケーションの作成にClojureを使うことができるようになります。

<!--
In short: Pedestal provides a better, cohesive way to build
rich client web applications in Clojure.
-->

要するに、PedestalはリッチクライアントウェブアプリケーションをClojureで作成するためのよりよい、まとまりのある方法を提供してくれるのです。

## 誰のためのものか

<!--
Clojurists looking for a standard way to build internet
applications will love Pedestal. Rather than composing art
out of found objects, they will now be able to mold a single,
consistent form to match their vision.
-->

インターネットアプリケーションを作成する標準的な方法を探しているClojuristは、Pedestalを気に入るでしょう。
見付けたオブジェクトを使って作品を組み立てるのではなく、彼らはそのビジョンに合った一つの一貫した形を作り上げることができるようになるでしょう。

<!--
Pedestal may also appeal to developers who have been nervously
approaching a functional language but who haven't yet mustered the
courage to ask it out on a date. It provides a sterling example
of how to use the Clojure ecosystem to its best advantage, reducing
the friction usually associated with a language switch.
-->

関数型言語に恐る恐る近付いてはいるのだけど、まだデートに招待する勇気を奮い起こせていない開発者にとっても、Pedestalは魅力的かもしれません。
Pedestalは、Clojureのエコシステムを存分に利用する方法についての最高品質の例を提供し、言語を変えるときに普通は伴う摩擦を減らします。

## どこから始めればよいか

<!--
In the _Getting Started_ section, you will find a walk-through
that introduces you to all of Pedestal's moving parts via the
creation of a new application. It explains the features
you'll find in the `pedestal.app` namespace.
-->

_さあ始めよう_の節に、新しいアプリケーションの作成を通じてPedestalの可動部品のすべてを紹介するウォークスルーがあるでしょう。
ウォークスルーは、あなたが`pedestal.app`名前空間で見付けるであろう機能を説明します。

<!--
_App Docs_ covers higher-level topics, and pulls back the
curtain on the reasoning behind Pedestal's approach.
-->

_アプリケーションドキュメント_はより高いレベルの話題を取り扱っていて、Pedestalの手法の背後にある理論を覆うカーテンを開きます。

<!--
_Service Docs_ gets down and dirty with the inner workings of
the `pedestal.service` layer.
-->

_サービスドキュメント_は`pedestal.service`レイヤーの内部的な動作で泥だらけです。

## APIドキュメントについてはどうか

<!--
To generate literate-programming-style documentation for the `app` and
`service` libraries, add the [lein plugin for
marginalia](https://github.com/fogus/lein-marginalia) to your lein user
profile. After installing the pedestal libraries you can then `cd` into the
`app` or `service` directories and run `lein marg`.
-->

`app`ライブラリーと`service`ライブラリーについての文芸的プログラミング型のドキュメントを生成するため、leinのユーザープロファイルに[marginaliaのleinプラグイン](https://github.com/fogus/lein-marginalia)を追加しましょう。
pedestalライブラリーをインストールした後、あなたは`app`ディレクトリーや`service`ディレクトリーに`cd`して、`lein marg`を実行することができます。

```bash
cat ~/.lein/profiles.clj
# {:user {:plugins [[lein-marginalia "0.7.1"]]}}

git clone https://github.com/pedestal/pedestal.git
cd pedestal
lein sub install
( cd app && lein marg )
( cd service && lein marg )
```

<!--
This will create the documentation for `pedestal.app` and
`pedestal.service` in their respective `docs` directory.
-->

これによって、`pedestal.app`と`pedestal.service`のドキュメントがそれぞれの`docs`ディレクトリーに作成されるでしょう。
