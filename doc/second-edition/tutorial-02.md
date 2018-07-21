# チュートリアル 2 - 即時フィードバックの原則


このチュートリアルでは、ClojureScriptプロジェクトを即時フィードバックの原則に近づ
けるように設定することを目的としています。[seminal talk][1].

## はじめに

[前のチュートリアル][2]の最後から作業を開始したい場合は、[git][3]がインストールされている前提で次のようにしてください：

```bash
git clone https://github.com/magomimmo/modern-cljs.git
cd modern-cljs
git checkout se-tutorial-01
```

これは、チュートリアルのrepoをクローンし、最初のチュートリアルの最後から開始することができます

> NOTE 1: `se-` プレフィックスは "second edition" を意味します.

## イントロダクション

`boot` ビルディングツールはより最近のツールであり、CLJ開発者のためのある種の標準である  `leiningen` ビルディングツールと比較すると成熟していません。 

しかしながら、`boot` コミュニティは、このギャップを埋めることを目的とし、その先には置き換えを目指して
`boot` の機能である *task* を徐々に充実させるために懸命に働いています。


コミュニティが開発した [tasks for boot][4] を見れば、Bret Victorの即時フィードバックの原則に近づくために必要なものはすべてすでにあることがわかります。

* [`boot-http`][5]: 単純なCLJベースのHTTPサーバを提供する `boot`タスク:
* [`boot-reload`][6]: 静的リソース（CSS、画像など）のライブリロードを提供する `boot`タスク:
* [`boot-cljs-repl`][7]: CLJS開発のためのREPLを提供する `boot`タスク:

    > 注2：前のチュートリアルですでに `boot-cljs`タスクを使用しました。

## CLJベースのhttpサーバ

`boot-http`サーバを` modern-cljs`ホームディレクトリにある `build.boot`ファイルに追加してみましょう。

```clj
(set-env!
 :source-paths #{"src/cljs"}
 :resource-paths #{"html"}

 :dependencies '[[adzerk/boot-cljs "1.7.228-2"]
                 [pandeiro/boot-http "0.7.6"]         ;; add http dependency
                 [org.clojure/tools.nrepl "0.2.12"]]) ;; required by boot-http

(require '[adzerk.boot-cljs :refer [cljs]]
         '[pandeiro.boot-http :refer [serve]]) ;; make serve task visible
```

   > 注3： `boot-http`の現在の` 0.7.6`バージョンのバグのために、 `tools.nrepl`の` 
  > '0.2.12`バージョンも追加しなければなりませんでした。

このように、最新の利用可能な `boot-http`のリリースをプロジェクトの依存関係に追加ました、そして
`require` フォームの中で  `serve` タスクをみえるようにすることで `boot`コマンドから参照できるように
しました。

Note that we're still implicitly exploiting a few `boot` defaults:

我々はまだいくつかの `boot`デフォルトを暗黙的に利用していることに注意してください：

* Clojure 1.7.0の利用が `boot.properties` ファイルに定義されている;
* ClojureScript 1.7.228 が`boot-cljs` のdependencyにより暗黙的にインポートされている:
  ;


これまでのように、新しく追加された `serve`タスクのヘルプドキュメントを見てみましょう：

```bash
boot serve -h
Start a web server on localhost, serving resources and optionally a directory.
Listens on port 3000 by default.

Options:
  -h, --help                Print this help info.
  -d, --dir PATH            PATH sets the directory to serve; created if doesn't exist.
  -H, --handler SYM         SYM sets the ring handler to serve.
  -i, --init SYM            SYM sets a function to run prior to starting the server.
  -c, --cleanup SYM         SYM sets a function to run after the server stops.
  -r, --resource-root ROOT  ROOT sets the root prefix when serving resources from classpath.
  -p, --port PORT           PORT sets the port to listen on. (Default: 3000).
  -k, --httpkit             Use Http-kit server instead of Jetty
  -s, --silent              Silent-mode (don't output anything)
  -t, --ssl                 Serve via Jetty SSL connector on localhost on default port 3443 using cert from ./boot-http-keystore.jks
  -T, --ssl-props SSL       SSL sets override default SSL properties e.g. "{:port 3443, :keystore "boot-http-keystore.jks", :key-password "p@ssw0rd"}".
  -R, --reload              Reload modified namespaces on each request.
  -n, --nrepl REPL          REPL sets nREPL server parameters e.g. "{:port 3001, :bind "0.0.0.0"}".
  -N, --not-found SYM       SYM sets a ring handler for requested resources that aren't in your directory. Useful for pushState.
```

`-d`オプションはディレクトリを設定するために使用されます。 もし、そのディレクトリが存在しなければ作成されます。 ターミナルで次の `boot`コマンドを試してみましょう：

```bash
boot serve -d target
2015-10-26 21:38:48.489:INFO:oejs.Server:jetty-7.6.13.v20130916
2015-10-26 21:38:48.549:INFO:oejs.AbstractConnector:Started SelectChannelConnector@0.0.0.0:3000
<< started Jetty on http://localhost:3000 >>
```

httpサーバ（Jetty）の起動後にコマンドが終了することに注意してください。 ブラウザのURLバーに `http：// localhost：3000`と入力するとエラーになります。 これは `serve`タスクがブロックされないためです。

この問題に対処するためには、 `boot`にすでに含まれている定義済みのタスクである `wait` を追加する必要があります。

```bash
boot wait -h
Wait before calling the next handler.

Waits forever if the --time option is not specified.

Options:
  -h, --help       Print this help info.
  -t, --time MSEC  Set the interval in milliseconds to MSEC.
```

この解決方法を見てみましょう：


```bash
boot wait serve -d target
2015-10-26 21:40:54.695:INFO:oejs.Server:jetty-7.6.13.v20130916
2015-10-26 21:40:54.772:INFO:oejs.AbstractConnector:Started SelectChannelConnector@0.0.0.0:3000
<< started Jetty on http://localhost:3000 >>
```

`boot`コマンドはもはや終了することなく、ブラウザから` http：//localhost：3000`に接続することができます。 サーバーを強制終了（ `CTRL-C`）します。

`boot`タスクは簡単に連鎖できます：

```bash
boot wait serve -d target cljs target
2016-01-03 11:14:09.949:INFO::clojure-agent-send-off-pool-0: Logging initialized @7356ms
Directory 'target' was not found. Creating it...2016-01-03 11:14:10.011:INFO:oejs.Server:clojure-agent-send-off-pool-0: jetty-9.2.10.v20150310
2016-01-03 11:14:10.048:INFO:oejs.ServerConnector:clojure-agent-send-off-pool-0: Started ServerConnector@31c694ca{HTTP/1.1}{0.0.0.0:3000}
2016-01-03 11:14:10.049:INFO:oejs.Server:clojure-agent-send-off-pool-0: Started @7455ms
Started Jetty on http://localhost:3000
Writing main.cljs.edn...
Compiling ClojureScript...
• main.js
Writing target dir(s)...
```

`http://localhost：3000`のURLにアクセスし、ブラウザのDeveloper Toolを開き、` Hello、World！ `文字列がコンソールに表示されていることを確認してください。 次のステップに進む前に、現在の `boot`プロセス（` CTRL-C`）を終了してください。

## CLJSソースの再コンパイル

即時フィードバックの原則にアプローチするのであれば、変更されたCLJSソースコードは `.cljs`ファイルを更新すると直ちに再コンパイルされる必要があります。

`watch`タスクは既に` boot`に含まれている数多くの定義済みタスクの一つです。

```bash
boot watch -h
Call the next handler when source files change.

Options:
  -h, --help           Print this help info.
  -q, --quiet          Suppress all output from running jobs.
  -v, --verbose        Print which files have changed.
  -M, --manual         Use a manual trigger instead of a file watcher.
  -d, --debounce MS    MS sets debounce time (how long to wait for filesystem events) in milliseconds.
  -i, --include REGEX  Conj REGEX onto the set of regexes the paths of changed files must match for watch to fire.
  -e, --exclude REGEX  Conj REGEX onto the set of regexes the paths of changed files must not match for watch to fire.
```

CLJSソースコードの変更が保存されるたびにCLJSの再コンパイルの実行をトリガーするのとは別に、 `watch`タスクはどちらもブロックしていないので` wait`タスクを置き換えることもできます。

呼び出す前に `watch`タスクを挿入するだけで`cljs`タスクでは、ソースの再コンパイルをトリガできます。

```bash
boot serve -d target watch cljs target
2017-03-09 23:07:18.959:INFO::clojure-agent-send-off-pool-0: Logging initialized @7461ms
2017-03-09 23:07:19.024:INFO:oejs.Server:clojure-agent-send-off-pool-0: jetty-9.2.10.v20150310
2017-03-09 23:07:19.075:INFO:oejs.ServerConnector:clojure-agent-send-off-pool-0: Started ServerConnector@693339d8{HTTP/1.1}{0.0.0.0:3000}
2017-03-09 23:07:19.080:INFO:oejs.Server:clojure-agent-send-off-pool-0: Started @7582ms
Started Jetty on http://localhost:3000

Starting file watcher (CTRL-C to quit)...

Writing main.cljs.edn...
Compiling ClojureScript...
• main.js
Writing target dir(s)...
Elapsed time: 8.787 sec
```

ブラウザでもう一度 `http：// localhost：3000`を入力して確認してください
それは "Hello, World!" がJSコンソールにプリントされています。 次に、お気に入りのエディタで
src / cljs / modern_cljs / core.cljs`ソースファイルを開き、プリントするメッセージを修正して ファイルに保存します。 新しいCLJSコンパイルタスクが端末でトリガーされていることがわかります。

```bash
Writing main.cljs.edn...
Compiling ClojureScript...
• main.js
Writing target dir(s)...
Elapsed time: 0.174 sec
```

最後に、htmlページをリロードして、リンクされた`main.js`ファイルがブラウザコンソールで
表示されたメッセージを見て更新されたことを確認してください。 ここまでは順調ですね。

次のステップに進む前に、 `boot`プロセスをkillしてください（` CTRL-C`）。


## リソースのリロード

CLJSソースファイルを変更するときはいつでも、あなたのコーディングの効果を検証するために、
手動でHTMLページを指し示すHTMLページを再ロードしなければなりません。 でも、即時フィードバックの原則にもっと近づけたいと考えています。

さいわいなことに静的リソースの再ロードを自動化するためにコミュニティが開発した `boot`タスクがあ
ります[`boot-reload`][6]。ここでも、新しいタスクをプロジェクトの依存関係に追加し、主なコマ
ンドを要求することによって `boot`からみえる必要があります。

```clj
(set-env!
 :source-paths #{"src/cljs"}
 :resource-paths #{"html"}

 :dependencies '[[adzerk/boot-cljs "1.7.228-2"]
                 [pandeiro/boot-http "0.7.6"]
                 [org.clojure/tools.nrepl "0.2.12"]
                 [adzerk/boot-reload "0.5.1"]]) ;; add boot-reload

(require '[adzerk.boot-cljs :refer [cljs]]
         '[pandeiro.boot-http :refer [serve]]
         '[adzerk.boot-reload :refer [reload]]) ;; make reload visible
```

このタスクは、 `cljs`コンパイルの直前に` boot`コマンドに挿入する必要があります。試してみます：

```bash
boot serve -d target watch reload cljs target
Starting reload server on ws://localhost:58020
Writing boot_reload.cljs...
2016-01-03 11:22:21.323:INFO::clojure-agent-send-off-pool-0: Logging initialized @9526ms
2016-01-03 11:22:21.387:INFO:oejs.Server:clojure-agent-send-off-pool-0: jetty-9.2.10.v20150310
2016-01-03 11:22:21.410:INFO:oejs.ServerConnector:clojure-agent-send-off-pool-0: Started ServerConnector@3117f174{HTTP/1.1}{0.0.0.0:3000}
2016-01-03 11:22:21.411:INFO:oejs.Server:clojure-agent-send-off-pool-0: Started @9614ms
Started Jetty on http://localhost:3000

Starting file watcher (CTRL-C to quit)...

Writing main.cljs.edn...
Compiling ClojureScript...
• main.js
Writing target dir(s)...
Elapsed time: 8.281 sec
```

ブラウザで通常のURLをリロードし、ブラウザのコンソールにプリントするメッセージを変更して
上記の手順を繰り返します。 前述のように、`core.cljs`ファイルを保存するとすぐに、CLJSの再コンパイルがトリガーされ
ます。 今回は、 `boot-reload`タスクのおかげで、ページも再読み込みされます。

これは、新しいメッセージがブラウザのコンソールに表示されているかどうかを確認することで確認できます。

HTMLソースファイルを変更して、ブラウザからほぼ即座にフィードバックを得ることさえできます。

いいですね。 次のレベルに進む前に `boot`コマンドをもう一度kill（` CTRL-C`）してください。


HTMLソースファイルを変更して、ブラウザからほぼ即座にフィードバックを得ることさえできます。
いい感じです。次のレベルに進む前に `boot`コマンドをもう一度kill（`CTRL-C`）してください。


## ブラウザ REPL (bREPL)

CLJのようなLISP方言を使用する主な理由の1つは、非常にインタラクティブなプログラミングスタイルを
可能にするREPL（Read Eval Print Loop）です。 

CLJSコミュニティは多大なハードワークによってCLJで利用できると同様のREPLベースのプログラミング経験
を可能としました。そして、CLJSREPLをブラウザ組み込みエンジンを含むほぼすべてのJSエンジンに接続す
る方法を開発しました。 

このスタイルのプログラミングでは、REPLでCLJSフォームを評価し、REPLが接続されているブラウザで即座
にフィードバックを得ることができます。

`boot`コミュニティはここでも提供可能なタスクを持っています。その名前は `boot-cljs-repl`です。 `boot`には含まれていない他のタスクのためにす
でに行ったように、` boot-cljs-repl`を `build.boot`プロジェクトファイルの依存関係に追加する必要があります。それから、いつものように端末の
`boot`コマンドでそれらを見えるようにするために、主タスク（`cljs-repl`と `start-repl`）を要求する必要があります。


```clj
(set-env!
 :source-paths #{"src/cljs"}
 :resource-paths #{"html"}

 :dependencies '[[adzerk/boot-cljs "1.7.228-2"]
                 [pandeiro/boot-http "0.7.6"]
                 [org.clojure/tools.nrepl "0.2.12"]
                 [adzerk/boot-reload "0.5.1"]
                 [adzerk/boot-cljs-repl "0.3.3"]])

(require '[adzerk.boot-cljs :refer [cljs]]
         '[pandeiro.boot-http :refer [serve]]
         '[adzerk.boot-reload :refer [reload]]
         '[adzerk.boot-cljs-repl :refer [cljs-repl start-repl]]) ;; make it visible
```

ふたたび、高度なオプションに関するドキュメントを読むには、 `boot cljs-repl -h`コマンドを実行してください。

```bash
boot cljs-repl -h
Retrieving boot-cljs-repl-0.3.3.pom from https://repo.clojars.org/ (1k)
Retrieving boot-cljs-repl-0.3.3.jar from https://repo.clojars.org/ (4k)
Start a ClojureScript REPL server.

The default configuration starts a websocket server on a random available
port on localhost.

Options:
  -h, --help                   Print this help info.
  -b, --ids BUILD_IDS          Conj [BUILD IDS] onto only inject reloading into these builds (= .cljs.edn files)
  -i, --ip ADDR                ADDR sets the IP address for the server to listen on.
  -n, --nrepl-opts NREPL_OPTS  NREPL_OPTS sets options passed to the `repl` task.
  -p, --port PORT              PORT sets the port the websocket server listens on.
  -w, --ws-host WSADDR         WSADDR sets the (optional) websocket host address to pass to clients.
  -s, --secure                 Flag to indicate whether the client should connect via wss. Defaults to false.
```

`cljs-repl`タスクは` cljs`タスクの直前に置かなければなりません。


`cljs-repl` の開発者は` build.boot`ビルドファイルの依存関係セクションに追加される `Clojure`と` ClojureScript`リリースを明示的に指定することを推奨しています。


```clj
(set-env!
 :source-paths #{"src/cljs"}
 :resource-paths #{"html"}

 :dependencies '[[org.clojure/clojure "1.8.0"]         ;; add CLJ
                 [org.clojure/clojurescript "1.9.473"] ;; add CLJS
                 [adzerk/boot-cljs "1.7.228-2"]
                 [pandeiro/boot-http "0.7.6"]
                 [org.clojure/tools.nrepl "0.2.12"]
                 [adzerk/boot-reload "0.5.1"]
                 [adzerk/boot-cljs-repl "0.3.3"]
                 ])

(require '[adzerk.boot-cljs :refer [cljs]]
         '[pandeiro.boot-http :refer [serve]]
         '[adzerk.boot-reload :refer [reload]]
         '[adzerk.boot-cljs-repl :refer [cljs-repl start-repl]])
```

新たに追加された `cljs-repl`タスクを試してみる前に、前のチュートリアルで紹介した` boot.properties`ファイルで `boot`自身が使うclojureコンパイラのバージョンを以下のように編集してください：


```bash
#http://boot-clj.com
#Thu Mar 09 21:18:39 CET 2017
BOOT_CLOJURE_NAME=org.clojure/clojure
BOOT_CLOJURE_VERSION=1.8.0
BOOT_VERSION=2.7.1
```

以下ような `boot`コマンドを起動すると、警告とエラーが表示されます：

```bash
boot serve -d target watch reload cljs-repl cljs target
Retrieving
...
Starting reload server on ws://localhost:53582
You are missing necessary dependencies for boot-cljs-repl.
Please add the following dependencies to your project:
[com.cemerick/piggieback "0.2.1" :scope "test"]
[weasel "0.7.0" :scope "test"]
...
java.io.FileNotFoundException: Could not locate cemerick/piggieback__init.class or cemerick/piggieback.clj on classpath.
Elapsed time: 0.734 sec
```

これは、 `boot-cljs-repl`には、 `build.boot`ファイルの`：dependencies`セクションに明示的に追加する必要があります:


```clj

(set-env!
 :source-paths #{"src/cljs"}
 :resource-paths #{"html"}

 :dependencies '[[org.clojure/clojure "1.8.0"]         ;; add CLJ
                 [org.clojure/clojurescript "1.9.473"] ;; add CLJS
                 [adzerk/boot-cljs "1.7.228-2"]
                 [pandeiro/boot-http "0.7.6"]
                 [org.clojure/tools.nrepl "0.2.12"]
                 [adzerk/boot-reload "0.5.1"]
                 [adzerk/boot-cljs-repl "0.3.3"]
                 [com.cemerick/piggieback "0.2.1"]     ;; needed by bREPL
                 [weasel "0.7.0"]])                    ;; needed by bREPL

(require '[adzerk.boot-cljs :refer [cljs]]
         '[pandeiro.boot-http :refer [serve]]
         '[adzerk.boot-reload :refer [reload]]
         '[adzerk.boot-cljs-repl :refer [cljs-repl start-repl]])

```

NOTE 3：現時点では、依存関係の `：scope 'を考慮しません。 後のチュートリアルでこの指定をします。


前のプロセスを終了した後、ターミナルで 間違えないように `boot`コマンドを以下のように実行します：


```bash
boot serve -d target watch reload cljs-repl cljs target
Starting reload server on ws://localhost:58051
Writing boot_reload.cljs...
Writing boot_cljs_repl.cljs...
2016-01-03 11:32:46.553:INFO::clojure-agent-send-off-pool-0: Logging initialized @9256ms
2016-01-03 11:32:46.613:INFO:oejs.Server:clojure-agent-send-off-pool-0: jetty-9.2.10.v20150310
2016-01-03 11:32:46.636:INFO:oejs.ServerConnector:clojure-agent-send-off-pool-0: Started ServerConnector@4a94a06{HTTP/1.1}{0.0.0.0:3000}
2016-01-03 11:32:46.637:INFO:oejs.Server:clojure-agent-send-off-pool-0: Started @9340ms
Started Jetty on http://localhost:3000

Starting file watcher (CTRL-C to quit)...

nREPL server started on port 58053 on host 127.0.0.1 - nrepl://127.0.0.1:58053
Writing main.cljs.edn...
Compiling ClojureScript...
• main.js
Writing target dir(s)...
Elapsed time: 18.949 sec
```

コマンド出力は、[`nrepl server`][8] がローカルホスト上でポート番号で起動されたことを通知します（ポート番号は異なります）。 エディタが `nrepl`をサポートしているなら、この情報を使って、` nrepl client`で現在実行中の `nrepl server 'に接続します。

> NOTE: Emacs と CIDER は上記をサポートしています。 これらはこちらから学ぶことができます。
[resources](https://github.com/magomimmo/modern-cljs/blob/master/doc/supplemental-material/emacs-cider-references.md).

現時点では、 `boot`に含まれている定義済みの` repl`タスクを起動し、`-c`（クライアント）オプションを渡すことで、第2の端末から`
cljs-repl`を実行できることで十分です：


```bash
# in a new terminal
cd /path/to/modern-cljs
boot repl -c
REPL-y 0.3.7, nREPL 0.2.12
Clojure 1.8.0
Java HotSpot(TM) 64-Bit Server VM 1.8.0_112-b16
        Exit: Control+D or (exit) or (quit)
    Commands: (user/help)
        Docs: (doc function-name-here)
              (find-doc "part-of-name-here")
Find by Name: (find-name "part-of-name-here")
      Source: (source function-name-here)
     Javadoc: (javadoc java-object-or-class-here)
    Examples from clojuredocs.org: [clojuredocs or cdoc]
              (user/clojuredocs name-here)
              (user/clojuredocs "ns-here" "name-here")
boot.user=>
```

これは `boot.user`名前空間にデフォルト設定された標準のCLJ REPLです。ここから、ブラウザベースのCLJS REPL（bREPL）を次のように起動できます。

```cljs
boot.user=> (start-repl)
<< started Weasel server on ws://127.0.0.1:49358 >>
<< waiting for client to connect ... Connection is ws://localhost:49358
Writing boot_cljs_repl.cljs...
```

端末は、ブラウザからのクライアント接続を待っています。 bREPL接続を有効
にするには、通常のhttp://localhost:3000 であるURLを参照してください。


```cljs
 connected! >>
To quit, type: :cljs/quit
nil
cljs.user=>
```

bREPLからCLJSフォームを評価できることを確認するには、アラート関数をブラウザに送信します。


```cljs
(js/alert "Hello, ClojureScript")
nil
```

bREPLを停止するには、 `：cljs / quit`式を実行してください。その後、CLJ REPLを停止します（CTRL-Dまたは `（exit）`または `（quit）`）。最後に
`boot`（CTRL-C）を止めてください。

次のチュートリアルに進む前に、gitリポジトリをリセットしてください。

```bash
git reset --hard
```

## 次のステップ - [Tutorial 3: House Keeping][9]

次の [tutorial][9] では、即時フィードバック開発環境（IFDE）にアプローチするための `boot`コマンドの起動を自動化します。

# ライセンス

Copyright © Mimmo Cosenza, 2012-2015. Released under the Eclipse Public
License, the same as Clojure.

[1]: https://vimeo.com/36579366
[2]: https://github.com/magomimmo/modern-cljs/blob/master/doc/second-edition/tutorial-01.md
[3]: https://git-scm.com/
[4]: https://github.com/boot-clj/boot/wiki/Community-Tasks
[5]: https://github.com/pandeiro/boot-http
[6]: https://github.com/adzerk-oss/boot-reload
[7]: https://github.com/adzerk-oss/boot-cljs-repl
[8]: https://github.com/clojure/tools.nrepl
[9]: https://github.com/magomimmo/modern-cljs/blob/master/doc/second-edition/tutorial-03.md
