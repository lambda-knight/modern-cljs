# チュートリアル 4 - Modern ClojureScript

このチュートリアルでは、[Larry Ullman] [2]の書籍[Modern JavaScript: Develop and Design][1] からJavaScript（JS）サンプルをCLJSに移植します。 コー
ドは[Larry Ullman][2].の本からダウンロードできます。

この内容をリファレンスとして選択した理由はスムーズに開始できること、その上でJSコーディングに対する堅牢なアプローチを保っているからです。 
私は、JSからClojureScript（CLJS）へのLarryのアプローチをもたらすことは、CLJSにまだ慣れていない人にとっても役立つと思います。
私たちは[前のチュートリアル][4]で設定した即時フィードバック開発環境（IFDE - Immediate Feedback Development）を使用して移植を行います。 
移植のどのフェーズにおいてもIFDEをやめることなく、あなたのお気に入りのプログラミングエディタをbREPLの利用に介在させます。
これは、利用可能なアプローチとツールの中で継続的開発を実際にやってみることになります。

## 序文

[git][5] がインストールされていると仮定して、[previous tutorial][4]の最後から作業を開始する場合は、次のようにします。

```bash
git clone https://github.com/magomimmo/modern-cljs.git
cd modern-cljs
git checkout se-tutorial-03
```

## はじめに

誰もが知っているように、1990年代にJSは主にHTMLフォームの改善と検証に使用されました。その後、2000年代の後半に、JSはサーバー側のリソースへの非
同期要求を行うために使用され始め、数ヶ月でAjaxとWeb 2.0という2つの新しいバズワードがありました。

先ほど触れたように、[Modern JavaScript][1] の本を参照して、[Larry Ullman's][2]アプローチを一種のModern ClojureScriptに変換しようとしま
す。

したがって、最初の例をCLJSに移行することから始めましょう。これはログインフォームです。これは、最近の10年でJSの使用の進化を説明することと、
ClojureやClojureScript自体をあまり知らずにCLJSプログラミングを開始するためには非常に役に立ちます。


## レジストレーションフォーム

[Modern JS][3] コードサンプルをダウンロードした場合は、 `login.html`、` css / styles.css`、 `js / login.js`ファイルが` ch02`ディレクトリにあ
ります。

![Modern ch02 tree][6]

ブラウザで `login.html`を開くと、次のように表示されます：

![Login Form][7]

さて、HTMLを見てみましょう。

```html
<!doctype html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <title>Login</title>
    <!--[if lt IE 9]>
    <script src="http://html5shiv.googlecode.com/svn/trunk/html5.js"></script>
    <![endif]-->
    <link rel="stylesheet" href="css/styles.css">
</head>
<body>
    <!-- login.html -->
    <form action="login.php" method="post" id="loginForm">
        <fieldset>
            <legend>Login</legend>

            <div>
              <label for="email">Email Address</label>
              <input type="email" name="email" id="email" required>

            </div>

            <div>
              <label for="password">Password</label>
              <input type="password" name="password" id="password"
              required>
            </div>

            <div>
              <label for="submit"></label>
              <input type="submit" value="Login &rarr;" id="submit">
            </div>

        </fieldset>
    </form>

    <script src="js/login.js"></script>

</body>
</html>
```
### プログレッシブなエンハンスと控えめなJS

各要素には、 `name`属性と` id`属性の両方があります。 `name`値は、フォームデータがサーバー側に送信されるときに使用されます。 `id`値はJS / CSS
によって使用されます。

`login.php`スクリプトは` form action`に関連付けられています。そして、`login.js`はhtmlページ内でリンクされています。 `login.js`とは別に、`
form`とJSスクリプトとの直接的な接続はありません。

この選択は、ラリー・ウルマン（Larry Ullman）が彼の著書で明確に説明している、いわゆる *プログレッシブ・エンハンスメント*と *控えめなJS* アプロー
チと関係があります。

以下の[sequence diagrams][8] は、これらのアプローチを実際に示しています。

#### サーバーサイドのみのバリデーション

![Login Form Seq DIA 1][9]

ユーザーが送信したフォームは、 `login.php`という名前のサーバー側PHPスクリプトによって検証されます。検証チェックがパスすると、サーバーはユー
ザーにログインします。そうでない場合、サーバーはエラーをユーザーに返して修正します。

JSとAjaxのおかげで、このユーザーエクスペリエンスを大幅に向上させることができます。より良い解決策は、JSを使用してクライアント側の検証を実行す
ることです。これにより、2番目のシーケンス図が表示されます。

#### クライアントサイドバリデーション

![Login Form Seq DIA 2][10]

クライアント側（つまりJS）の検証に合格する場合は、セキュリティ上の理由からサーバー側の検証を
依頼する必要があります。しかし、クライアント側の検証が合格しない場合は、サーバーへのラウンド
トリップを行う必要はなく、エラーをすぐにユーザーに返すことができます。

しかし、まだいくつか問題があります。クライアント側の検証では、ユーザー名が登録されているかどうかを確認できません。これによりAjaxと第3のシーケンス図が得られます。

#### Ajaxインアクション

![login Form Seq DIA 3][11]

ユーザーエクスペリエンスが大幅に強化されました。 Ajaxコールは、（例えば、電子メールアドレスが存在するかどうかを検証するために）サーバと通信し、結果としてより効率的かつ応答性の高いプロセスをもたらします。

このチュートリアルでは、サーバー側の検証またはサーバーへのAjax呼び出しを実装せずに、クライアント側の
検証シナリオに限定します。それ以降のより高度なチュートリアルでこれらを実装します。

## ClojureScriptへのポーティング

ここまで来るのに少し時間がかかりましたが、ようやくログインフォームの検証をJSからCLJSに移植するこ
とができます。 最初に[CLJS interop][13]を基礎となる JavaScript仮想マシン（JSVM）と組み合わせて
JSをCLJSに直接変換することから始めます。

### `login.html` ファイルのコピー

まず、[here][3]からダウンロードしたzipファイルから`login.html`リソースをコピーしてください。


```bash
cd /path/to/modern-cljs
unzip ~/Downloads/modern_javascript_scripts.zip -d ~/Downloads/
cp ~/Downloads/modern_javascript_scripts/ch02/login.html html/index.html
```

`login.html`を` index.html`に名前を変更することで、プロジェクトのホームディレクトリの
`html`サブディレクトリに[Tutorial-01][12].で作成した`index.html`ファイルを上書きします。

先に進む前に、コピーされた `html / index.html`ファイルに2つの小さな変更を加える必要があります：

* `<form>`タグに `novalidate`属性を加えてフォームフィールドのHTML5チェックを無効にします。

* CLJS コンパイラによって生成された JS ファイルをリンクするために`<script>`タグの `src`属性を` js/login.js`から`js / main.js`に更新します。

```html
<!doctype html>
<html lang="en">
...
<body>
    <!-- Script 2.2 - login.html -->
    <form action="login.php" method="post" id="loginForm" novalidate>
    ...
    </form>
    <script src="js/main.js"></script>
</body>
</html>
```

### 即時フィードバック開発環境（IFDE）の立ち上げ

即時フィードバックの原則に近づく開発環境を構築するための前回の3回のチュートリアルでの取り組みを
考えてみましょう。CLJSにログインフォームにJSコードを移植しています。

これまでどおりIFDEを起動します。

```bash
boot dev
Starting reload server on ws://localhost:57153
Writing boot_reload.cljs...
Writing boot_cljs_repl.cljs...
2015-11-15 18:46:58.893:INFO::clojure-agent-send-off-pool-0: Logging initialized @9965ms
2015-11-15 18:46:58.963:INFO:oejs.Server:clojure-agent-send-off-pool-0: jetty-9.2.10.v20150310
2015-11-15 18:46:58.988:INFO:oejs.ServerConnector:clojure-agent-send-off-pool-0: Started ServerConnector@55a25f49{HTTP/1.1}{0.0.0.0:3000}
2015-11-15 18:46:58.989:INFO:oejs.Server:clojure-agent-send-off-pool-0: Started @10061ms
Started Jetty on http://localhost:3000

Starting file watcher (CTRL-C to quit)...

Adding :require adzerk.boot-reload to main.cljs.edn...
nREPL server started on port 57154 on host 127.0.0.1 - nrepl://127.0.0.1:57154
Adding :require adzerk.boot-cljs-repl to main.cljs.edn...
Compiling ClojureScript...
• js/main.js
Writing target dir(s)...
Elapsed time: 17.874 sec
```

これで `http：// localhost：3000`のURLにアクセスしてログインフォームを取得できます。ご覧のとおり、CSSスタイルはリンクされていません。これは、元の `styles.css`をプロジェクトにコピーしなかったためです。

私たちのIFDEが次のように元のCSSスタイルの追加を管理できるかどうかを見てみましょう：


```bash
# from a new terminal
cd /path/to/modern-cljs
mkdir html/css
cp ~/Downloads/modern_javascript_scripts/ch02/css/styles.css html/css/
```

`styles.css`ファイルをプロジェクトの`html/css`ディレクトリにコピーすると、それをロードするIFDEとアップロードされるログインフォームのスタイルが表示されます。

ここまでは順調です。

今度は、元の `login.js`コードの動作をエミュレートしようとすることで、bREPLでCLJSを試してみましょう。始める前に、validateForm（）関数の例を考えてみましょう：

```JS
// Script 2.3 - login.js

// Function called when the form is submitted.
// Function validates the form data and returns a Boolean value.
function validateForm() {
    'use strict';

    // Get references to the form elements:
    var email = document.getElementById('email');
    var password = document.getElementById('password');

    // Validate!
    if ( (email.value.length > 0) && (password.value.length > 0) ) {
        return true;
    } else {
        alert('Please complete the form!');
        return false;
    }

} // End of validateForm() function.
```
ここで、 `validateForm（）`関数はフォーム入力から `email`と`password`IIDを取得し、両方が値を持っていることを確認します（ `length>0`）。

`validateForm（）`関数は、検証が合格すると `true`を返し、そうでなければ` false`を返します。

わかった、それは非常に基本的な検証ですが、それはこのアプローチを実証するのに役立つ一種のプレースホルダーです。


### CLJSによるbREPLing 

いつものようにbREPLを起動してください：


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
boot.user=> (start-repl)
<< started Weasel server on ws://127.0.0.1:57267 >>
<< waiting for client to connect ... Connection is ws://localhost:57267
Writing boot_cljs_repl.cljs...
 connected! >>
To quit, type: :cljs/quit
nil
cljs.user=>
```


> 注記1：[`leiningen`][15] ビルドツールと比較した主な`boot`の利点は JVMクラスローダを
利用することによってのみ、1つのJVMを使用できることです。

> `nrepl`準拠のエディタを使って作業する場合、新しいJVMインスタンスを起動せずに` boot dev`
コマンドで起動された `nrepl`サーバに接続できるはずです。
> 個人的に私は `emacs`エディタと[cider][16] を使用しています。
> なぜなら、私は年長者であるので、`emacs`は設定と使い方に関してベストなエディタだからです。
> EmacsやCIDERについてもっと知りたい方は[ここにいくつかの参考文献があります]

(https://github.com/magomimmo/modern-cljs/blob/master/doc/supplemental-material/emacs-cider-references.md).

bREPLでCLJS式を評価できるようになりましたが、最初にブラウザDOMにアクセスする方法が必要です。

```clj
cljs.user=> js/window
#object[Window [object Window]]
cljs.user=> js/document
#object[HTMLDocument [object HTMLDocument]]
cljs.user> js/console
#js {:debug #object[debug "function debug() {
    [native code]
}"], :error #object[error "function error() {
    [native code]
}"], :log #object[log "function log() {
    [native code]
}"],
...
     :groupEnd #object[groupEnd "function groupEnd() {
    [native code]
}"]}

```

ご覧のように、ブラウザのJSVMによってホストされている場合、CLJSは、グローバル空間で定義されたJSオブジェクトにアクセスするための特別な名前空間 `js` を定義します。

* `js/window`: ブラウザのウィンドウを表します
* `js/document`: ドキュメントオブジェクト（DOM）を表します。
* `js/console`: ブラウザのJSコンソールを表します。

ゲストプログラミング言語であるCLJSは、special formを通してその配下のJSエンジンと相互運用
できます`（object.function * args）`：

```clj
cljs.user> (js/console.log "Hello from ClojureScript!")
nil
```

> 注記2：コンソールログがbREPLから呼び出されても、コンソールに出力されても、
> `Iceweasel` Webブラウザはエラーメッセージを投げます。このエラーメッセージは、
> bREPLが `Chromium`ウェブブラウザに接続しても発生しません。


ここでは、 特別な `js` 名前空間にある`console`オブジェクトの `log`関数へ
`Hello from ClojureScript！'`文字列を引数として渡すことで` log`関数を呼
び出しました。 `/` 文字は、名前空間自体で定義されているシンボルから分離された名前空
間名を保持します。


ブラウザのJSコンソールに `Hello from ClojureScript！` というメッセージが表示されます。

次に、同じの相互運用可能なspecial formを使用して、ログインフォームのDOM要素を取得するとしましょう。

```clj
cljs.user> (js/document.getElementById "loginForm")
#object[HTMLFormElement [object HTMLFormElement]]
cljs.user> (js/document.getElementById "email")
#object[HTMLInputElement [object HTMLInputElement]]
cljs.user> (js/document.getElementById "password")
#object[HTMLInputElement [object HTMLInputElement]]
cljs.user> (js/document.getElementById "submit")
#object[HTMLInputElement [object HTMLInputElement]]
```

それほど悪くはありませんが、CLJSの構文は、以前の相互運用可能なシナリオである
`（.function + args）` のシンタックスシュガーをサポートしています。このシンタックスシ
ュガーの形態は、関数を暗黙の引数としてオブジェクトを使用しないので、Clojariansによってより
慣用的と見なされます。試してみよう：

Not so bad, but CLJS syntax supports some syntactic sugar for the previous interoperable scenario: `(.function +args)`. This syntactic sugar form is considered more idiomatic by Clojarians because it does not use the object as an implicit argument of the function. Let's try it:

```clj
cljs.user> (.log js/console "Hello from ClojureScript!")
nil
```

Here we called the `log` function, passing it both the `console`
object and the `"Hello from ClojureScript!"` string as arguments.

You should see the `Hello from ClojureScript!` message printed again to
the JS console of the browser.

Now apply the above idiomatic interoperable form on the DOM elements
as well:

```clj
cljs.user> (.getElementById js/document "loginForm")
#object[HTMLFormElement [object HTMLFormElement]]
cljs.user> (.getElementById js/document "email")
#object[HTMLInputElement [object HTMLInputElement]]
cljs.user> (.getElementById js/document "password")
#object[HTMLInputElement [object HTMLInputElement]]
cljs.user> (.getElementById js/document "submit")
#object[HTMLInputElement [object HTMLInputElement]]
```

To resemble the original JS code we also need to access the `value`
property of the `email` and `password` elements. CLJS offers the `.-`
special form for these cases as well: `(.-property object)`:

```clj
cljs.user> (.-value (.getElementById js/document "email"))
""
cljs.user> (.-value (.getElementById js/document "password"))
""
```

As you see the two calls both return the void string `""`. Now fill in
both the `email` and `password` fields in the login form and evaluate
again the above expressions:

```clj
cljs.user> (.-value (.getElementById js/document "email"))
"you@yourdomain.com"
cljs.user> (.-value (.getElementById js/document "password"))
"bjSgwMd24J"
```

We still need other things. We need a way to set a value for a DOM
element. Again, CLJS offers a way to do that via the `(set! (.-property
object) arg)` form. Let's try to set the `value` property of the
`password` element and then get it back:

```clj
cljs.user> (set! (.-value (.getElementById js/document "password"))
                 "weakpassword")
"weakpassword"
cljs.user> (.-value (.getElementById js/document "password"))
"weakpassword"
```

> NOTE 3: in CLJ/CLJS when a function changes the state of a mutable
> object, it is idiomatic to append the `!` char to it. This happens
> very frequently in interoperable scenarios with the underlining
> hosting platforms (i.e. JVM, JSVM and CLR at the moment).

Lastly, we need a function for counting the length of a string. The
`count` function works on any kind of collection and on strings as
well:

```clj
cljs.user> (count (.-value (.getElementById js/document "email")))
18
cljs.user> (count (.-value (.getElementById js/document "password")))
12
```

So far so good.

### Define the `validate-form` function

Now that we have done a few experiments in the bREPL on the interoperability
between CLJS and JS, we should be able to define a `validate-form`
CLJS function cloning the corresponding JS `validateForm` behavior.

Create a new CLJS file named `login.cljs` in the
`src/cljs/modern_cljs` directory which already hosts the `core.cljs`
source created in the [Tutorial-01][12]. Copy the following content
into it:

```clj
(ns modern-cljs.login)

;; define the function to be attached to form submission event
(defn validate-form []
  ;; get email and password element from their ids in the HTML form
  (let [email (.getElementById js/document "email")
        password (.getElementById js/document "password")]
    (if (and (> (count (.-value email)) 0)
             (> (count (.-value password)) 0))
      true
      (do (js/alert "Please, complete the form!")
          false))))
```

> NOTE 4: Note that in CLJ/CLJS the use of CamelCase to name things is
> not idiomatic. That's why we translated `validateForm` to
> `validate-form`.

Every CLJS file needs to define a namespace. Here we create the
`modern-cljs.login` namespace in which we defined the `validate-form`
function with the `defn` macro.

Inside the `validate-form` definition we used the `let` form to define
a kind of local variable, like `var` in the corresponding JS code. We
used the "."  and ".-" JS interoperable forms to call JS native
functions (i.e.  `.getElementById`) and to get/set JS object
properties (i.e.  `.-value`).

If you're curious about `defn`, `>`, `if`, `and` and `let` forms, just ask
for the internal documentation from the bREPL as follows:

```clj
cljs.user> (doc defn)
-------------------------
cljs.core/defn
([name doc-string? attr-map? [params*] prepost-map? body] [name doc-string? attr-map? ([params*] prepost-map? body) + attr-map?])
Macro
  Same as (def name (core/fn [params* ] exprs*)) or (def
    name (core/fn ([params* ] exprs*)+)) with any doc-string or attrs added
    to the var metadata. prepost-map defines a map with optional keys
    :pre and :post that contain collections of pre or post conditions.
nil
cljs.user> (doc >)
-------------------------
cljs.core/>
([x] [x y] [x y & more])
  Returns non-nil if nums are in monotonically decreasing order,
  otherwise false.
nil
cljs.user> (doc if)
-------------------------
if
   (if test then else?)
Special Form
  Evaluates test. If not the singular values nil or false,
  evaluates and yields then, otherwise, evaluates and yields else. If
  else is not supplied it defaults to nil.

  Please see http://clojure.org/special_forms#if
nil
cljs.user> (doc and)
-------------------------
cljs.core/and
([] [x] [x & next])
Macro
  Evaluates exprs one at a time, from left to right. If a form
  returns logical false (nil or false), and returns that value and
  doesn't evaluate any of the other expressions, otherwise it returns
  the value of the last expr. (and) returns true.
nil
cljs.user> (doc let)
-------------------------
cljs.core/let
([bindings & body])
Macro
  binding => binding-form init-expr

  Evaluates the exprs in a lexical context in which the symbols in
  the binding-forms are bound to their respective init-exprs or parts
  therein.
nil
```

Pretty handy. As you see `defn`, `and` and `let` forms are macro
expressions, while `>` is a regular function and finally `if` is a
special form. In the first position of a list expression you'll always
find one of those three forms, unless the list expression is quoted
with the `quote` special form which stands for preventing the form
evaluation. I don't know about you, but after many decades I've
yet to see a programming language with a syntax easier to
remember than a LISP, regardless of dialect.

You can even ask at the bREPL for the source code definitions of
symbols which are defined as macros or regular functions.

```clj
cljs.user> (source let)
(core/defmacro let
  "binding => binding-form init-expr

  Evaluates the exprs in a lexical context in which the symbols in
  the binding-forms are bound to their respective init-exprs or parts
  therein."
  [bindings & body]
  (assert-args let
     (vector? bindings) "a vector for its binding"
     (even? (count bindings)) "an even number of forms in binding vector")
  `(let* ~(destructure bindings) ~@body))
nil
cljs.user> (source >)
(defn ^boolean >
  "Returns non-nil if nums are in monotonically decreasing order,
  otherwise false."
  ([x] true)
  ([x y] (cljs.core/> x y))
  ([x y & more]
   (if (cljs.core/> x y)
     (if (next more)
       (recur y (first more) (next more))
       (cljs.core/> y (first more)))
     false)))
nil
```

All that said, as soon as you save the above file, the IFDE recompiles
it and reloads the `index.html` file that links it. But we still have
to modify the `html/js/main.cljs.edn` to inform our IFDE that we added
a new CLJS namespace.

### Update the `main.cljs.edn` file

Edit the `html/js/main.cljs.edn` file to add the `modern-cljs.login`
newly created namespace as follows:

```clj
{:require [modern-cljs.core modern-cljs.login]
 :compiler-options {:asset-path "js/main.out"}}
```

As soon as you save the change, the IFDE triggers the CLJS
recompilation and reloads the `index.html` file linking the
`js/main.js` generated file.

### bREPLing again

We can now verify if our `validate-form` function works as
expected. Go back to your bREPL and require the newly created
`modern-cljs.login` namespace as follows:

```clj
cljs.user> (require '[modern-cljs.login :as l] :reload)
nil
```

Here we required the `modern-cljs.login` namespace by aliasing it as
`l` into the current special `cljs.user` default namespace. The
`:reload` option forces loading of the `modern-cljs.login` namespace
even if it is already loaded.

First take a look at the value associated with the `l/validate-form`
symbol.


```clj
cljs.user> l/validate-form
#object[modern_cljs$login$validate_form "function modern_cljs$login$validate_form(){
var email = document.getElementById("email");
var password = document.getElementById("password");
if(((cljs.core.count.call(null,email.value) > (0))) && ((cljs.core.count.call(null,password.value) > (0)))){
return true;
} else {
alert("Please, complete the form!");

return false;
}
}"]
```

As you see the CLJS compiler translated the original CLJS code into
corresponding JS code. Even if the `source-map` compiler option allows
you to debug your CLJS code in the Developer Tools of your browser, the
understanding of the CLJS to JS translation could be very effective in
identifying and solving bugs in your code.

Obviously you can still see the CLJS `validate-form` definition by
using the `source` macro:

```clj
cljs.user> (source l/validate-form)
(defn validate-form []
  ;; get email and password element from their ids in the HTML form
  (let [email (.getElementById js/document "email")
        password (.getElementById js/document "password")]
    (if (and (> (count (.-value email)) 0)
             (> (count (.-value password)) 0))
      true
      (do (js/alert "Please, complete the form!")
          false))))
nil
```

Enough side talks. Let's now delete any value from the `email` and the
`password` fields and call the `validate-form` function.

```clj
cljs.user> (set! (.-value (.getElementById js/document "email")) "")
""
cljs.user> (set! (.-value (.getElementById js/document "password")) "")
""
cljs.user> (l/validate-form)
false
```

The call to `validate-form` opens the Alert Dialog and as soon as you
click the ok button, it will return `false` to the bREPL.

Now fill the above fields with some values and call the
`validate-form` function again.

```clj
cljs.user> (set! (.-value (.getElementById js/document "email")) "you@yourdomain.com")
"you@yourdomain.com"
cljs.user> (set! (.-value (.getElementById js/document "password")) "weakpassword")
"weakpassword"
cljs.user> (l/validate-form)
true
```

As expected, this time the `validate-form` function call will
immediately return `true`.

### Port the `init` function

After a little bit of bREPLing to get familiarized with some stuff, we were
able to define a CLJS function resembling the corresponding JS
`validateForm` function. But we still have to attach it to the
`submit` button of the login form when the `index.html` page is loaded
into the browser.

Here is the original JS code:

```js
// Function called when the window has been loaded.
// Function needs to add an event listener to the form.
function init() {
    'use strict';

    // Confirm that document.getElementById() can be used:
    if (document && document.getElementById) {
        var loginForm = document.getElementById('loginForm');
        loginForm.onsubmit = validateForm;
    }

} // End of init() function.

// Assign an event listener to the window's load event:
window.onload = init;
```

Open the `login.cljs` file again and add the `init` function at the
end as follows:

```clj
;; define the function to attach validate-form to onsubmit event of
;; the form
(defn init []
  ;; verify that js/document exists and that it has a getElementById
  ;; property
  (if (and js/document
           (.-getElementById js/document))
    ;; get loginForm by element id and set its onsubmit property to
    ;; our validate-form function
    (let [login-form (.getElementById js/document "loginForm")]
      (set! (.-onsubmit login-form) validate-form))))
```

As in the corresponding JS code, here we have associated the
`validate-form` function to the `onsubmit` property of the
`login-form`.

There is one last thing to be done to complete the porting of the
original JS code: we have to associate the `init` function with the
`onload` property of the `window` object.

Add the following code at the end of the `login.cljs` file.

```clj
;; initialize the HTML page in unobtrusive way
(set! (.-onload js/window) init)
```

As soon as you save the `login.cljs` everything is recompiled. But
this time you have to reload the Login Page because we attached the
`init` function to the `onload` event.

You can now safely play with the Login form from the browser itself:

* if you click the `Login` button before having filled both the
`email` and the `password` fields, you should see the alert dialog
popping up and asking you to complete the form;
* if you click the `Login` button after having filled both the `email`
and the `password` fields you should see the browser error page saying
that the page `localhost:3000/login.php` was not found. That's because the
action attribute of the html form still references `login.php` as the
server-side validation script. The server-side validation will be
implemented in CLJ in a subsequent tutorial.

Have you noted that we never had to stop/restart the IFDE during the
porting of the login form validation from JS to CLJS?

That's it for this tutorial and you can kill any `boot` related
process and reset your git repository.

```bash
git reset --hard
```

# Next Step [Tutorial 5: Introducing Domina][17]

In the [next tutorial][17] we're going to use the domina library to make our
login form validation more Clojure-ish.

# License

Copyright © Mimmo Cosenza, 2012-15. Released under the Eclipse Public
License, the same as Clojure.

[1]: http://www.larryullman.com/books/modern-javascript-develop-and-design/
[2]: http://www.larryullman.com/
[3]: http://www.larryullman.com/downloads/modern_javascript_scripts.zip
[4]: https://github.com/magomimmo/modern-cljs/blob/master/doc/second-edition/tutorial-03.md
[5]: https://help.github.com/articles/set-up-git
[6]: https://raw.github.com/magomimmo/modern-cljs/master/doc/images/ch02-tree.png
[7]: https://raw.github.com/magomimmo/modern-cljs/master/doc/images/login-form.png
[8]: http://en.wikipedia.org/wiki/Sequence_diagram
[9]: https://raw.github.com/magomimmo/modern-cljs/master/doc/images/login-01-dia.png
[10]: https://raw.github.com/magomimmo/modern-cljs/master/doc/images/login-02-dia.png
[11]: https://raw.github.com/magomimmo/modern-cljs/master/doc/images/login-03-dia.png
[12]: https://github.com/magomimmo/modern-cljs/blob/master/doc/second-edition/tutorial-01.md
[13]: https://github.com/clojure/clojurescript/wiki/Differences-from-Clojure#host-interop
[14]: https://github.com/clojure/clojurescript/wiki/Differences-from-Clojure
[15]: http://leiningen.org/
[16]: https://github.com/clojure-emacs/cider
[17]: https://github.com/magomimmo/modern-cljs/blob/master/doc/second-edition/tutorial-05.md
