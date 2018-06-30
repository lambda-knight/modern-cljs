# チュートリアル 1 - 基礎

この最初のチュートリアルでは、最小のClojurescript([CLJS][1]) のプロジェクトを
[boot][2]というビルドツールを用いて構成します。

## 要求事項

このチュートリアルでは `java` と `boot` がコンピュータにインストールされていることが前提です。

`java` をインストレールするためには [instructions][3] のそれぞれのオペーレーティングシステム
の手順に従いましょう。
`boot` をインストールするためには非常に簡単な手順の[README][4]の対応するセクションに従いましょう。


ターミナルからの `boot -h` コマンドによりインストールのテストが出来ます。そして、 `boot -u` 
コマンドにより最新の `boot` のアップデートを得ることができます。

> NOTE 1: Java 8を利用することを強く推奨します。もし、Java 7を利用しているのであれば、以下を
> 言及しておく価値があります。
> https://github.com/boot-clj/boot/wiki/JVM-Options#permgen-errors

## プロジェクト構造の作成

最小のCLJSのwebプロジェクトは3つのファイルより構成されます。

* 1つのhtmlページ;
* 1つのCLJSソースコード;
* CLJSソースコードをコンパイルするための1つの `boot` ビルドファイル

CLJSが特定のディレクトリ構造を指示していない場合でも、プロジェクト構成要素とその構造を
誰でも簡単に理解できるように、数か月間はこのような形でプロジェクトを構成することをお勧めします。

さらに各々のビルディングツールは特有の表現方法をデフォルトとして保持しています。そのため利用してい
るツールのデフォルトルールを守ることにより、プロジェクトを管理する際に苦労することは少なくなります。

これらの前提を考慮して、新しいプロジェクトのディレクトリ構造を `modern-cljs`という名前で作成
しましょう。できるだけ` boot` のデフォルトに守らなければなりません。


推奨しているファイルレイアウトは以下の通りです。

```bash
modern-cljs/
├── build.boot
├── html
│   └── index.html
└── src
    └── cljs
        └── modern_cljs
            └── core.cljs
```

* `modern-cljs` プロジェクトのホームディレクトリ;
* `src/cljs/` はCLJSのソースファイルを保持している:
* `html` htmlのリソースを保持している;

> NOTE 2: 単一セグメントのネームスペースは
> [CLJ/CLJSにおいてやる気がそがれます][5]. これが`modern_cljs` というディレクトリ名に
> した理由です。 パッケージ名におけるハイフン "-" (やそれ以外の特殊文字)を管理す
> る[Javaの困難さ][6] により、対応するディレクトリ名はハイフン(`-`) をアンダースコア(`_`)に置き
> 替えます。

> NOTE 3: ClojureScriptのファイル拡張子は **cljs** であって **clj** ではないことに注意して下さい。

ターミナルにおいて以下のコマンドを発行して下さい。

```bash
mkdir -p modern-cljs/{src/cljs/modern_cljs,html}
```

3つの必要なファイルを以下のコマンドにより発行しましょう。

コマンド:

```bash
cd modern-cljs
touch html/index.html src/cljs/modern_cljs/core.cljs build.boot
```

## CLJSにおけるHello World

ここで最初のCLJSのコードを記述する。 `src/cljs/modern_cljs/core.cljs` ファイルをお気に入り
のエディタで開いて以下のCLJSのコードをタイプします。

```clj
;; メインプロジェクトのnamespaceを作成する
(ns modern-cljs.core)

;; cljsに対してブラウザのJSコンソールへのプリントを可能とする
(enable-console-print!)

;; コンソールへのプリント
(println "Hello, World!")
```

すべてのCLJ/CLJSファイルはあなたのディスク上のパスとマッチするネームスペース宣言で開始されるべきです。

`modern-cljs.core` <--> `modern_cljs/core.cljs`


`(enable-console-print!)` の表現はすべてのプリント出力をブラウザコンソールにリダイレクトします。
例えば `(println "Hello, world!")` はコンソールに `Hello, World!` をプリントします。

### 最小のbuild.boot


`core.cljs` JSにコンパイルして`index.html` ページとリンクさせる必要があります。
まず、 `html/index.html` をオープンして以下のように編集します。


```html
<!doctype html>
<html>
  <head>
    <title>Hello, World!</title>
  </head>
  <body>
    <script src="main.js"></script>
  </body>
</html>
```

ここにはCLJSファイルに対して何も参照がないことに注意しましょう。
`index.html` ページには `<script>` タグでは `main.js` へのリンクがあるだけです。
このjsファイルは上記の `core.cljs` ソースコードを`boot` ビルディングツールによってコンパイルする過程で生成されます。

`core.cljs` ファイルをコンパイルするためには `boot` コマンドの `build.boot` ファイル
を編集しなければなりません。 このファイルは拡張子がことなる通常のCLJファイルです。


```clj
(set-env!
 :source-paths #{"src/cljs"}
 :resource-paths #{"html"}

 :dependencies '[[adzerk/boot-cljs "1.7.228-2"]])

(require '[adzerk.boot-cljs :refer [cljs]])
```

なんとミニマルなのでしょう! `set-env!` 関数は `:source-paths` と`:resource-paths` 
オプションに我々が作成した上記のプロジェクト構造に対応する値を設定しています。

次に、 `boot-cljs`コンパイルタスクを`：dependencies`キーワードに追加することで、
プロジェクトの唯一の明示的な依存関係として指定します。


'clojure'と'clojurescript'の依存関係が含まれていなくても、'boot'は自動的に対応する
リリースを自動的にダウンロードできるようになることに注意してください。

最後に、 `require`形式は` boot`コマンドに `cljs`タスクを可視化します。

そして、ターミナルから `boot -h`コマンドを実行すると、` cljs`タスクが `boot`で利用できるようになります。


```bash
boot -h

...

Tasks:   ...
         target                      Writes output files to the given dir...
         ...
         zip                         Build a zip file for the project.

         cljs                        Compile ClojureScript applications.

...

Do `boot <task> -h` to see usage info and TASK_OPTS for <task>.
```

次のコマンドを発行することで、 `cljs`タスクに関する詳細を知りたいかもしれません：

You now may want to have more information about the `cljs` task by
issuing the following command:


```bash
boot cljs -h
Compile ClojureScript applications.
...
Available --optimization levels (default 'none'):
...
```

As you see, the default optimization directive for the CLJS compiler
is `none`. Another tutorial will explain the different CLJS compilation
optimizations (i.e. `none`, `whitespace`, `simple` and `advanced`). At
the moment stay with `none`, which is commonly used during development
cycles. Let's see `boot cljs` at work:

```bash
boot cljs
Writing main.cljs.edn...
Compiling ClojureScript...
• main.js
```

The `cljs` task compiled your CLJS code by producing a `main.js` JS
file and you now know why we called the JS file included in the
`<script>` tag of the `index.html` page `main.js`: just to adhere to
an easy default.

That said, if you look for it the directory structure of the project,
you'll be surprised to not find it:

```bash
tree
.
├── build.boot
├── html
│   └── index.html
└── src
    └── cljs
        └── modern_cljs
            └── core.cljs

4 directories, 3 files
```

The fact is that by default `boot` does not create an output file if
you don't explicitely instruct it to do so. Let's see the command line
help for the `target` predefined task:

```bash
boot target -h
Writes output files to the given directory on the filesystem.

Options:
  -h, --help      Print this help info.
  -d, --dir PATH  Conj PATH onto the set of directories to write to (target).
  -m, --mode VAL  VAL sets the mode of written files in 'rwxrwxrwx' format.
  -L, --no-link   Don't create hard links.
  -C, --no-clean  Don't clean target before writing project files.
```

Interesting. The `target` predefined task of `boot` is able to write
the output of a task (e.g. `cljs`) to a given directory. The above
help doesn't tell you, but if you do not specify a directory it writes
the output file to the a default `target` subdirectory of your project's home
directory, as you can verify as follows:

```bash
boot cljs target
Writing main.cljs.edn...
Compiling ClojureScript...
• main.js
Writing target dir(s)...
```

```bash
tree
.
├── boot.properties
├── build.boot
├── html
│   └── index.html
├── src
│   └── cljs
│       └── modern_cljs
│           └── core.cljs
└── target
    ├── index.html
    ├── main.js
    └── main.out
        ├── boot
        │   └── cljs
        │       ├── main312.cljs
        │       ├── main312.cljs.cache.edn
        │       ├── main312.js
        │       └── main312.js.map
        ├── cljs
        │   ├── core.cljs
        │   ├── core.js
        │   └── core.js.map
        ├── cljs_deps.js
        ├── goog
        │   ├── array
        │   │   └── array.js
        │   ├── asserts
        │   │   └── asserts.js
        │   ├── base.js
        │   ├── debug
        │   │   └── error.js
        │   ├── deps.js
        │   ├── dom
        │   │   └── nodetype.js
        │   ├── object
        │   │   └── object.js
        │   └── string
        │       ├── string.js
        │       └── stringbuffer.js
        └── modern_cljs
            ├── core.cljs
            ├── core.cljs.cache.edn
            ├── core.js
            └── core.js.map

17 directories, 27 files
```

A lot of stuff. We're not digging into it right now. At the
moment we're only interested in noting a few things:

* the original directory structure living in `html` and `src` is
  untouched
* everything, even the `index.html` resource, has been generated into
  the new `target` directory. 

Considering that `boot` is under continuous development, I strongly
suggest you to pin the current stable release in your project by creating a new
`boot.properties` file as follows:

```bash
boot -V > boot.properties
```

which should now have the following content:

```bash
cat boot.properties
#http://boot-clj.com
#Thu Mar 09 14:56:52 CET 2017
BOOT_CLOJURE_NAME=org.clojure/clojure
BOOT_CLOJURE_VERSION=1.7.0
BOOT_VERSION=2.7.1
```

## index.htmlをみる

`boot` uses a pretty neat approach in taking apart the input of the
project from the corresponding output generated by its tasks. You'll
never see an input file from the `:source-paths` and the
`:resource-path` original directories be modified by any `boot`
task. Aside from internally generated temporary directories,
everything happens in the explicit `target` directory.

Open a browser and visit the local `target/index.html` file. Now open
the console in your Development Tool (e.g. Chrome Development Tool).
If everything went ok, you should see "Hello, World!" printed at the
console.

## 次のステップ - [Tutorial 2: Immediate Feedback Principle][7]

In the next [tutorial][7] we're going to adhere as closely as possible to
the [Bret Victor Immediate Feedback Principle][8] to build a very
interactive development environment.

# ライセンス

Copyright © Mimmo Cosenza, 2012-2015. Released under the Eclipse Public
License, the same as Clojure.

[1]: https://github.com/clojure/clojurescript.git
[2]: http://boot-clj.com/
[3]: https://java.com/en/download/help/index_installing.xml?os=All+Platforms&j=8&n=20
[4]: https://github.com/boot-clj/boot#install
[5]: http://stackoverflow.com/questions/13567078/whats-wrong-with-single-segment-namespaces
[6]: http://docs.oracle.com/javase/specs/jls/se8/html/jls-6.html
[7]: https://github.com/magomimmo/modern-cljs/blob/master/doc/second-edition/tutorial-02.md
[8]: https://vimeo.com/36579366
