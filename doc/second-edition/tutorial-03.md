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

`deftask`マクロを介して新しく作成された` dev`の本体で `comp`関数を使うことに注意してください。  `comp` 関数は `boot` コマンドと同じ順序で同じタスクを
構成します。 

唯一の違いは  `target`  値がコマンドラインで引数 としてではなく `serve` や`target` サブタスクに `：dir` キーワードの引数として が渡される方法である。

さらに `target`タスクに渡される値は単純な文字列（例えば `target`）ではなく、文字列のセット（ `#{""target"}`）であることです。

現時点では、`boot`が複数のディレクトリに出力を生成できても、この選択の理由が何であるかを理解することに興味がありません。

新しく定義された `dev`タスクは、コマンドラインで利用できるようになりました：




```bash
boot -h
...
         dev                        Launch Immediate Feedback Development Environment
...
Do `boot <task> -h` to see usage info and TASK_OPTS for <task>.
```

コマンドラインからいつものように `dev`タスクを起動して、以前にサブタスクを作成することで得たことと同じ効果を得ることができます：

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

nreplクライアントからbREPLの生成を含む [Tutorial 2][1]  にすでに示されているすべてのテストを繰り返すことができます。
悪くはないです。即時フィードバックの原理に近づく開発環境を立ち上げるための非常に単純なコマンドである `boot dev`があります。

あなたがこの実験を終えたら、すべてをkillして下さい。

## デフォルトのオーバライト

最初から述べたように、私たちは、プロジェクトのレイアウト構造を簡素化する目的で、できるだけ多くの `boot` タスクのデフォルトに堅持することに努めました。
つまり、たとえ初期設定のような非常にシンプルなプロジェクトにデフォルト設定が採用されていても、プロジェクトがより具体的になってくるとすぐにそれら
を上書きする必要があります。

ほとんどのWebアプリケーションは、HTML / CSS / JS リソースをお互いに分離して構成しています。例を挙げると、ほとんどのWebアプリケーションは、
JSリソースをWebサーバーが提供するディレクトリにある `js`サブディレクトリの下にグループ化し、それらの` js`リソースを対応する `<script>` html
タグにリンクします。

最初に `html / index.html`ファイルを編集し、CLJSのコンパイル・フェーズで生成された` main.js`ファイルへのリンクを変更することで、この規約に従っ
てみましょう。

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

[clojurescript compiler options][4] を見ると、この新しいファイル構成をサポートするためにCLJSコンパイラに渡す2つのオプションが必要であること
がわかります。


* [`:output-to`][5]: 出力されるJavaScriptファイルへのパスを設定します
* [`:asset-path`][6]: Webサーバによって提供されるディレクトリに対する相対パスを設定します。

これらのオプションをCLJSコンパイラを起動する `boot-cljs`にどのように渡すことができるのでしょうか。 `boot-cljs`のドキュメントを読むことで、この非常に
単純な問題を解決しようとすると、簡単に狂ってしまうことがあります。いくつかのコンパイラオプションはタスクレベルで設定できますが、リソースディ
レクトリの適切な位置にある適切な[`edn`] [8]ファイルをプロジェクトの`html`ディレクトリに定義することで設定できます。

いくつかのコンパイラオプションはタスクレベルで設定することができ、他のコンパイルオプションはresourcesディレクトリの適切な位置にある適切な
[`edn`][8] ファイルを定義することで設定できます（例えば、プロジェクトの `html`ディレクトリに）。

`boot-cljs`がコンパイラオプションをサポートする理由について、あまりにも多くの詳細には触れません。代わりに、もう一度現実的に
何をする必要があるのか​​を列挙しましょう。

まず、htmlページを含む `html`ディレクトリに` js`サブディレクトリを作成する必要があります。


```bash
cd pathname/to/modern-cljs
mkdir html/js
```
CLJSコンパイラによって生成されたJSファイルの名前を反映して新しく作成された `js`ディレクトリに` cljs.edn`ファイルを作成します。


```bash
touch html/js/main.cljs.edn
```

次に、そのファイルを次のように編集します。

```clj
{:require [modern-cljs.core]
 :compiler-options {:asset-path "js/main.out"}}
```

ご覧のように、プロジェクトの `modern-cljs.core`名前空間だけを必要とし、` asset-path`オプションをCLJSコンパイラに設定しました。 `output-to`オ
プションが指定されていない場合、デフォルト値は[relative path of the .cljs.edn file][9]に基づいて作成されます。このようにしてCLJSコンパイラの設定が
完了しました。これでおしまいです。

CLJSのコンパイルを実行できるようになりました。非常に冗長なCLJSコンパイルタスクの出力を見るには、 `boot`コマンドの` -vv`オプションを次のよう
に使います：

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

こうすることで、CLJSコンパイラが `main.js`ファイルを` js`サブディレクトリに正しく生成されたことを確認することができます。
そしてこのサブディレクトリはWebサーバーが提供するディレクトリからの相対パスで、 `js/main.out`サブディレクトリを `asset-path` 
コンパイラオプションの値として設定します。

次に、 `target`ディレクトリのレイアウトを見て、コンパイラオプションが期待通りに機能していることを確認してください


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
最後に、[Tutorial 2][1]で紹介したすべての手動テストを繰り返して、即時フィードバック開発環境がまだ機能していることを確認します。


`boot dev`コマンドを起動します：


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

そして `bREPL` を起動してみて下さい:

```clj
boot.user=> (start-repl)
<< started Weasel server on ws://127.0.0.1:52439 >>
<< waiting for client to connect ... Connection is ws://localhost:52439
Writing boot_cljs_repl.cljs...
```

最後に `http：//localhost：3000` のURLにアクセスしてbREPLをアクティブにしてください。



```
 connected! >>
To quit, type: :cljs/quit
nil
cljs.user=>
```

あなたが好きなように楽しんでください。終了したら、すべて終了し、gitブランチをリセットします。

```bash
git reset --hard
```

## 次のステップ - [Tutorial 4: Modern ClojureScript][7]

[next tutorial 4][7] では、CLJSでフォームのバリデーションを導入することで楽しむことができます。



# ライセンス

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
