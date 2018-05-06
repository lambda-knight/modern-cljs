# チュートリアル 3 - ハウスキーピング

[前のチュートリアルl][1]では、`boot`ビルドツールを使い、コミュニティによって開発された
いくつかの追加タスクを追加することで、即時フィードバックの原則に近づき始めました：

このチュートリアルでは、ターミナルに提出される`boot`コマンドの長さを最小限に抑える
ことによって開発者との対話を改善し、プログラムの即時フィードバックスタイルをサポートします。

* `boot-cljs`: CLJSソースコードをコンパイルする[Tutorial 1][2]);
* `boot-http`: ページを提供するCLJベースのWebサーバーを実行します;
* `boot-reload`: 変更が保存されたときに静的リソースをリロードする;
* `boot-cljs-repl`: CLJS REPLをJSエンジンに接続します、ブラウザ（bREPL）

このチュートリアルでは、ターミナルに提出される `boot` コマンドの長さを最小限に抑え
ることによって開発者との対話を改善し、プログラムの即時フィードバックスタイルをサポートします。

## 前提事項

[previous tutorial][1] の最後から作業を開始したい場合は、gitがインストール
されていると仮定して、次のようにします。

```bash
git clone https://github.com/magomimmo/modern-cljs.git
cd modern-cljs
git checkout se-tutorial-02
```

## はじめに

即座にフィードバックの原則に近づくために端末に以前に提出した `boot`コマンドを見直
してみましょう。


### CLJSのコンパイル

まず、[チュートリアル1] [2]では、いくつかの環境変数を構成しました。すなわち `：source-paths`と`：resource-paths`です。

そして、以下の `boot`コマンドでCLJSコンパイルを開始しました：

```bash
boot cljs target
```

これまでは利用可能なデフォルトをいくつか利用することで、コンパイルオプションをタスクに
渡しませんでした。 すなわち：

コンパイラ最適化としての `none`;
* `source-map`のデフォルト値は` none`で `true`です
* `main.js`はコンパイラによって生成されたJSファイルの名前です

     >注1：ClojureScriptはHTMLソースマップをサポートしているため、設定オプション   
     > `source-map`を使用してブラウザでClojureScriptを直接デバッグできます。 最適化が  
     > `none`に設定されている場合に有効な値は` true`と `false`です。デフォルトは` true`です。 
     > その他の最適化設定では、ソースマップを書き込むパスを指定する必要があります
     > （[ClojureScriptのコンパイラ最適化][3]を参照）。

### HTTPサーバ

[チュートリアル2][1] では、 `boot`コマンドに` serve`タスクを追加することから始めました。 
`-d target`オプションを渡して、タスクに`target`タスクによってデフォルトとして使われているのと
同一ディレクトリ名を指定しました。 そして、Webサーバーを稼働させ続けるための `wait`タスクも追加する必要がありました。

```bash
boot wait serve -d target cljs target
```

### CLJS再コンパイル

変更されたCLJSソースファイルを保存するたびにCLJSの再コンパイルをトリガするために、 `cljs`タスクの前に配置する必要がある `wait`タスクを `watch`タスクに置き換えました。

```bash
boot serve -d target watch cljs target
```

### bootのリロード

再び、静的リソースのリロードをトリガするために、変更や変更がCLJSソースコードに保存されたとき
にそれらをリンクするために、 `reload`タスクを` cljs`タスクの直前にパイプラインに追加しました。

```bash
boot serve -d target watch reload cljs target
```

### bREPL

最後に、ブラウザベースREPL（bREPL）を提供することで、我々の即時フィードバックの原則の
アプローチをほぼ完成させるために、 `cljs-repl`タスクを追加しなければなりませんでした。 
`cljs-repl`タスクの前に` cljs`タスクを追加することを忘れないでください。

```bash
boot serve -d target watch reload cljs-repl cljs target
```

もし、プロジェクトで遊び始めるためだけにこれらのことを覚えておかなければないと不平を言
うならば、その通りです。

そして このデフォルトは `boot`コア開発者が、あなたのタイピングとあなたの記憶を助ける非
常に快適な方法として提供する理由なのです。

## deftaskの導入


ユーザの観点から見ると、 `boot`の興味深い側面の1つは、`boot`によってあらかじめ定義されている
かどうかに関係なく、新たなタスクの構成可能なことです。 

アーキテクチャをよりよく理解するために`boot`ソースコードを研究するために時間をかけることもできま
すが、私たちはもっと実用的であるべきです。

現時点では、ターミナルから `boot`コマンドを動かしながら、タスク名とその順序を記憶する必
要性を減らすことにのみ関心があります。

そのためにやるべきことは `build.boot`を開き、次のように`deftask`マクロを使って新しいタスクを
他のタスクの順序付けられたコンポジションとして定義するだけです。

```clj
(set-env!
  ...
  ...)

(require ...
         ...)

;; define dev task as composition of subtasks
(deftask dev
  "Launch Immediate Feedback Development Environment"
  []
  (comp
   (serve :dir "target")
   (watch)
   (reload)
   (cljs-repl) ;; before cljs task
   (cljs)
   (target :dir #{"target"})))
```

Note the use of the `comp` function in the body of the newly created
`dev` via the `deftask` macro. The `comp` function composes the same
tasks in the same order we saw in the `boot` command. The only
differences are the way the `"target"` values are passed to `serve`
and to `target` subtasks as a `:dir` keyword argument instead of as an
argument at the command-line. Moreover, the value passed to the
`target` task is not just a simple string (i.e., `"target"`), but a
set of strings (i.e., `#{"target"}`). At the moment we're not
interested in understanding what could be the reason of this choice,
even if you could suspect that `boot` is able to generate output in
more than one directory.

The newly defined `dev` task is now available to be used at the
command-line:

```bash
boot -h
...
         dev                        Launch Immediate Feedback Development Environment
...
Do `boot <task> -h` to see usage info and TASK_OPTS for <task>.
```

You can now launch the `dev` task as usual from the command line to
get the same effect you previously obtained by composing its subtasks:

```bash
boot dev
Starting reload server on ws://localhost:60083
Writing boot_reload.cljs...
Writing boot_cljs_repl.cljs...
2015-11-04 08:58:55.742:INFO:oejs.Server:jetty-7.6.13.v20130916
2015-11-04 08:58:55.788:INFO:oejs.AbstractConnector:Started SelectChannelConnector@0.0.0.0:3000
<< started Jetty on http://localhost:3000 >>

Starting file watcher (CTRL-C to quit)...

nREPL server started on port 60086 on host 127.0.0.1 - nrepl://127.0.0.1:60086
Writing main.cljs.edn...
Compiling ClojureScript...
• main.js
Writing target dir(s)...
Elapsed time: 19.929 sec
```

You can then repeat all the tests already shown in [Tutorial 2][1]
including the bREPL creation from a nrepl client.

Not bad. You now have a very simple command, `boot dev`, to launch a
development environment approaching the Immediate Feedback Principle.

When you finish with your experimenting, kill everything.

## Defaults overwriting

As told from the very beginning, we tried to adhere as much as
possible to some `boot` tasks' defaults with the intent of
simplifying the structure layout of the project. That said, even if
some defaults are sane enough to be adopted for a very simple project
like the one we started from, as soon as the project gets more
articulated you'll need to overwrite a few of them.

Most webapps organize their HTML/CSS/JS resources to keep them
separated from each other. Just to make an example, most of the
webapps group their JS resources under a `js` subdirectory living in
the directory served by the web server and link those `js` resources in
the corresponding `<script>` html tag.

Let's try to adhere to this convention by first editing the
`html/index.html` file and modifying the link to the `main.js` file
generated by the CLJS compilation phase.

```html
<!doctype html>
<html>
  <head>
    <title>Hello, World!</title>
  </head>
  <body>
    <script src="js/main.js"></script>
  </body>
</html>
```

If you take a look at the [clojurescript compiler options][4] you'll
discover that you need two options to be passed to the CLJS compiler
to support this new files organization:

* [`:output-to`][5]: sets the path to the JavaScript file that will be
  output
* [`:asset-path`][6]: sets a path relative to the directory served by
  the web server.

How could we pass these options to the `boot-cljs` which launches the
CLJS compiler? You could easily go crazy trying to solve
this very simple problem by reading the `boot-cljs`
documentation. Some compiler options can be set at the task level,
others can be set by defining appropriate [`edn`][8] files located in
the right position in the resources directories (e.g. into the `html`
directory in our project).

Let's not go into too many details about the reasons that `boot-cljs` supports
the compiler options the way it does; instead, let's be pragmatic again and
just list what you need to do:

First you have to create the `js` subdirectory in the `html` directory
containing the html pages.

```bash
cd pathname/to/modern-cljs
mkdir html/js
```

Then create a `cljs.edn` file in the newly created `js` directory
reflecting the name of the JS file generated by the CLJS compiler.

```bash
touch html/js/main.cljs.edn
```

Now edit that file as follows:

```clj
{:require [modern-cljs.core]
 :compiler-options {:asset-path "js/main.out"}}
```

As you see we just required the `modern-cljs.core` namespace of the
project and set the `asset-path` option to the CLJS compiler. If the `output-to` option is not specified default value is created based on [relative path of the .cljs.edn file][9]. This way we have completed configuration for CLJS compiler. That's it.

You can now run the CLJS compilation. To see the very verbose output
of the CLJS compilation task, use the `-vv` option of the `boot`
command as follows:

```bash
boot -vv cljs target
...
Compiling ClojureScript...
...
• js/main.js
CLJS options:
{:asset-path "js/main.out",
 :output-dir
 "/Users/mimmo/.boot/cache/tmp/Users/mimmo/tmp/modern-cljs/3c5/-w7zxdu/js/main.out",
 :output-to
 "/Users/mimmo/.boot/cache/tmp/Users/mimmo/tmp/modern-cljs/3c5/-w7zxdu/js/main.js",
 :main boot.cljs.main583}
...
```

This way you can verify that the CLJS compiler correctly generated the
`main.js` file in the `js` subdirectory relative to the directory
served by the web server and that it sets the `js/main.out`
subdirectory as the value of the `asset-path` compiler option.

Now take a look at the layout of the `target` directory to confirm
that the compiler options worked as expected

```bash
target
├── index.html
└── js
    ├── main.cljs.edn
    ├── main.js
    └── main.out
        ├── boot
        ...
        └── modern_cljs
            ├── core.cljs
            ├── core.cljs.cache.edn
            ├── core.js
            └── core.js.map
```

Finally, repeat all the manual tests we introduced in the
[Tutorial 2][1] to verify that the Immediate Feedback Development
Environment is still working.

Launch the `boot dev` command:

```bash
boot dev
Starting reload server on ws://localhost:52434
Writing boot_reload.cljs...
Writing boot_cljs_repl.cljs...
2015-11-04 17:26:11.291:INFO:oejs.Server:jetty-7.6.13.v20130916
2015-11-04 17:26:11.445:INFO:oejs.AbstractConnector:Started SelectChannelConnector@0.0.0.0:3000
<< started Jetty on http://localhost:3000 >>

Starting file watcher (CTRL-C to quit)...

Adding :require adzerk.boot-reload to main.cljs.edn...
nREPL server started on port 52437 on host 127.0.0.1 - nrepl://127.0.0.1:52437
Adding :require adzerk.boot-cljs-repl to main.cljs.edn...
Compiling ClojureScript...
• js/main.js
Writing target dir(s)...
Elapsed time: 25.922 sec
```

Now launch the `nrepl` client in a new terminal:

```clj
boot repl -c
REPL-y 0.3.7, nREPL 0.2.12
Clojure 1.7.0
Java HotSpot(TM) 64-Bit Server VM 1.8.0_66-b17
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

Then launch the `bREPL`:

```clj
boot.user=> (start-repl)
<< started Weasel server on ws://127.0.0.1:52439 >>
<< waiting for client to connect ... Connection is ws://localhost:52439
Writing boot_cljs_repl.cljs...
```

An finally visit the `http://localhost:3000` URL to activate the bREPL

```
 connected! >>
To quit, type: :cljs/quit
nil
cljs.user=>
```

Play around as you like. When finished, quit everything and reset your
git branch.

```bash
git reset --hard
```

## Next step - [Tutorial 4: Modern ClojureScript][7]

In the [next tutorial 4][7] we're going to have some fun introducing
form validation in CLJS.

# License

Copyright © Mimmo Cosenza, 2012-2017. Released under the Eclipse Public
License, the same as Clojure.

[1]: https://github.com/magomimmo/modern-cljs/blob/master/doc/second-edition/tutorial-02.md
[2]: https://github.com/magomimmo/modern-cljs/blob/master/doc/second-edition/tutorial-01.md
[3]: https://github.com/clojure/clojurescript/wiki/Compiler-Options#source-map
[4]: https://github.com/clojure/clojurescript/wiki/Compiler-Options
[5]: https://github.com/clojure/clojurescript/wiki/Compiler-Options#output-to
[6]: https://github.com/clojure/clojurescript/wiki/Compiler-Options#asset-path
[7]: https://github.com/magomimmo/modern-cljs/blob/master/doc/second-edition/tutorial-04.md
[8]: https://github.com/edn-format/edn
[9]: https://github.com/boot-clj/boot-cljs/blob/master/docs/compiler-options.md
