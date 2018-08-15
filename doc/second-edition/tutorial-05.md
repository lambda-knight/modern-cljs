# Tutorial 5 - Introducing Domina

このチュートリアルでは、[前のチュートリアル][2]で示したログインフォームの妥当性を改善するために[Domina][1] を紹介します。

## Preamble


前のチュートリアル] [2]
[git][13] がインストールされていると仮定して、[前のチュートリアル][2]の最後から作業を開始したい場合、以下のようにしてください：


```bash
git clone https://github.com/magomimmo/modern-cljs.git
cd modern-cljs
git checkout se-tutorial-04
```

## イントロダクション

[前のチュートリアル][2]では、CLJSの[JS interop][7]機能を使用し、JSからCLJSに直接変換することによって、CLJSでコーディングをはじめました。
今回はよりよいものを試しましょう。

> [Domina][1] はClojureScript用のjQuery触発されたDOM操作ライブラリです。 Dominaはイノベーションを提供しませんが、ClojureScriptで
> 自然なDOM操作の基本的なインタフェースを提供しようとしています。


私がCLJSガイドを検索して最初に `clojurescriptone`に出会ったとき、`Design and templating`について読んだとき、私は次のような論拠
によく合っていました。

多くのClojure WebアプリケーションはHTMLテンプレート作成に[Hiccup][5] を使用しています。プログラマーがデザイナーでもあれば、Hiccupは理想的で
す。しかし、ほとんどの開発者はデザイナーとしては優れていないことが多いです。私たちは、デザインに優れ、Clojureを気にする必要のない人と協力
する必要があります。 ClojureScriptOneは、Hiccupデータ構造や厄介な括弧に目を向けることなく、デザイナーがHTML、CSS、および画像を扱うことを
可能にするテンプレート化のアプローチを提案しています。

> 注1： `clojurescriptone`リポジトリに対応する` Design and templating`ド
> キュメントは▽すでに削除されており、利用できません。


私たちの古い `index.html`フレンドは純粋なHTML / CSSテンプレートとなり、` domina`はより慣用的なCLJSでページのDOMをインターフェースするCLJSライブラリになります。

Our old `index.html` friend is going to be our pure HTML/CSS template and `domina` is going to be our CLJS library to interface the DOM of the page in more idiomatic CLJS.

今日、いわゆるシングルページアプリケーション（SPA）の普及により、Web開発のやり方が徐々に変化しています.1つのdivタグで構成された単一のHTMLページが
すべてのDOM操作をJSに委任するのはめずらしくないです。

Nowadays, the diffusion of the so called Single Page Application (SPA) is progressively changing the web development landscape and it's not unusual to see a single HTML page composed of one `div` tag only delegating all the DOM manipulation to JS.

That said, there is still a very large number of organizations keeping the HTML designers separated from the developers and we should be ready to give life to a bunch of pure static HTML/CSS pages.

## Domina lib

[Domina][1] was one of the first DOM libraries written in CLJS and it
has not been updated to follow the evolution of CLJS compiler. If you
use the canonical `domina` release (i.e., `[domina "1.0.3"]`), during
the compilation you'll get a warning about the fact that it uses a
single segment namespace. Even if those warnings do not affect the
behavior of the lib in the context of this tutorial, I really hate
warnings. So I prepared [a non canonical `domina` release][15] which fixes
those warnings.  That said, even if I would never suggest to use
`domina` in a new CLJS project, this tutorial could still be useful to
understand the way CLJS works.

As usual to use a new library, you need to add it to the dependencies
section of the `build.boot` file living in the home directory of the
project:

```clj
(set-env!
 ...
 :dependencies '[
                 ...
                 [org.clojars.magomimmo/domina "2.0.0-SNAPSHOT"]
                 ])
```

## Launch the Immediate Feedback Development Environment (IFDE)

As we learnt in the [previous tutorial][2], the use of the IFDE allows
us to familiarize ourselves with the CLJS language by evaluating
expressions in the bREPL before extending an application by directly
coding into the source files.

So, let's start the IFDE as usual:

```clj
boot dev
Retrieving domina-2.0.0-20151125.115321-1.jar from https://clojars.org/repo/
...
Compiling ClojureScript...
• js/main.js
Elapsed time: 22.633 sec
```

## Launch the bREPL

As usual, after the `boot dev` command finishes with the CLJ
compilation, visit the `http://localhost:3000` URL and then launch the
bREPL in a new terminal from the project home directory:

```clj
# from a new terminal
cd /path/to/modern-cljs
boot repl -c
...
boot.user=> (start-repl)
...
cljs.user=>
```

## Review the Login form

Before we start bREPLing, let's review the content of the
`html/index.html` file containing the login form we used in
[Tutorial 4][2]:

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
    <form action="login.php" method="post" id="loginForm" novalidate>
        <fieldset>
            <legend>Login</legend>

            <div>
              <label for="email">Email Address</label>
              <input type="email" name="email" id="email" required>
            </div>

            <div>
              <label for="password">Password</label>
              <input type="password" name="password" id="password" required>
            </div>

            <div>
              <label for="submit"></label>
              <input type="submit" value="Login &rarr;" id="submit">
            </div>

        </fieldset>
    </form>
    <script src="js/main.js"></script>
</body>
</html>
```

## Domina selectors

[Domina][1] offers several selector functions: `xpath`, in the
`domina.xpath` namespace, and `sel`, in the `domina.css`
namespace. But it also features the `by-id`, `value` and `set-value!`
functions defined in the `domina.core` namespace, which is the one
we're going to use.

The nice thing about `by-id` and `id`, inherited from the underlying
Google Closure Library (GCL) on top of which `domina` is implemented,
is that they take care of verifying if the passed argument is a
string. As we anticipated, the `domina.core` namespace offers other
useful functions we're going to use: `value`, which returns the value
of the passed element, and `set-value!` which sets its value.

> NOTE 2: as we already saw, when a function modifies an argument
> passed to it, by Clojure naming convention a bang "!" is added at
> the end of the function name.

## bREPLing with the login form

> NOTE 3: Due to a bug of the `boot-cljs-repl` task, we need to first
> require a namespace from a namespace declaration to be able to
> require it in the bREPL as well. We will use `domina` later to update
> our `validate-form` function in `login.cljs`. You can require the
> `domina.core` namespace there.

Change the first line of your `login.cljs` file:

```clj
(ns modern-cljs.login
  (:require [domina.core :refer [by-id value set-value!]]))
```

Let's now familiarize ourselves in the bREPL with the above `domina`
functions.

First we need to require the `domina.core` namespace:

```clj
cljs.user=> (require '[domina.core :refer [by-id value set-value!]])
nil
```

As you see, instead of aliasing the required namespace as we did in
the [previous tutorial][2], we're now directly interning `by-id`,
`value` and `set-value!` symbols into the `cljs.user` namespace loaded
by default by the bREPL in such a way that we can use them from the
bREPL without specifying any namespace's name or alias:

```clj
cljs.user=> (doc by-id)
-------------------------
cljs.user/by-id
([id])
  Returns content containing a single node by looking up the given ID
nil
cljs.user=> (by-id "email")
#object[HTMLInputElement [object HTMLInputElement]]
```

Pretty easy. Let's familiarize ourselves with `set-value!` and `value`
symbols as well:

```clj
cljs.user=> (doc set-value!)
-------------------------
cljs.user/set-value!
([content value])
  Sets the value of all the nodes (presumably form fields) in the given content.
nil
cljs.user=> (set-value! (by-id "email") "you@yourdomain.com")
#object[HTMLInputElement [object HTMLInputElement]]
cljs.user=> (set-value! (by-id "password") "weakpassword")
#object[HTMLInputElement [object HTMLInputElement]]
cljs.user=> (doc value)
-------------------------
cljs.user/value
([content])
  Returns the value of a node (presumably a form field). Assumes content is a single node.
nil
cljs.user=> (value (by-id "email"))
"you@yourdomain.com"
cljs.user=> (value (by-id "password"))
"weakpassword"
cljs.user=> (set-value! (by-id "password") "")
#object[HTMLInputElement [object HTMLInputElement]]
cljs.user=> (set-value! (by-id "email") "")
#object[HTMLInputElement [object HTMLInputElement]]
```

Have you noted that as soon as the `set-value!` forms are evaluated
the corresponding fields in the login form have been updated?

## Update `login.cljs`

Now that we better understand a few `domina` functions, we are going to
update the `validate-form` function. Open the
`login.cljs` file and update both the namespace declaration and the
`validate-form` function definition as follows:

```clj
(ns modern-cljs.login
  (:require [domina.core :refer [by-id value]]))

(defn validate-form []
  (if (and (> (count (value (by-id "email"))) 0)
           (> (count (value (by-id "password"))) 0))
    true
    (do (js/alert "Please, complete the form!")
        false)))
```

As you can see, by using `domina` the code is now more fluid and
idiomatic than before.  Leave the rest of the file as is.

As soon as you save the changes, the IFDE triggers the CLJS compiler
and reloads the `index.html` file as well.

You can safely interact with the login form which is now managed via the
`domina` lib.

Require the `modern-cljs.login` namespace:
```clj
cljs.user=> (require '[modern-cljs.login :as l] :reload)
nil
```

Then repeat the kind of experiments we did in [the previous tutorial][2]:

```clj
cljs.user=> l/validate-form
#object[modern_cljs$login$validate_form "function modern_cljs$login$validate_form(){
if(((cljs.core.count.call(null,domina.value.call(null,domina.by_id.call(null,"email"))) > (0))) && ((cljs.core.count.call(null,domina.value.call(null,domina.by_id.call(null,"password"))) > (0)))){
return true;
} else {
alert("Please, complete the form!");

return false;
}
}"]
cljs.user=> (l/validate-form)
false
cljs.user=> (set-value! (by-id "email") "you@yourdomain.com")
#object[HTMLInputElement [object HTMLInputElement]]
cljs.user=> (set-value! (by-id "password") "weakpassword")
#object[HTMLInputElement [object HTMLInputElement]]
cljs.user=> (l/validate-form)
true
```

As already shown in the [previous tutorial][2], the evaluation of the
`l/validate-form` symbol returns the JS translation generated by the
CLJS compiler from the corresponding `validate-form` CLJS function
definition.

Then, when we call the `validate-form` while the `email` and
`password` fields are empty, it returns the `false` boolean value.

Finally, after having set a non-void string for both fields, the
`validate-form` evaluation returns, as expected, the `true` boolean
value.

## Shopping calculator sample

Now let's try to port a second example from Larry Ullman's
[Modern JavaScript][6] book to CLJS: a kind of e-commerce tool that will
calculate the total of an order, including tax, minus any discount.

### Pure HTML/CSS page

Here is the `shopping.html` content which is in line with the
`clojurescriptone approach` and [Larry Ullman][6] to keep the design
of the HTML/CSS/images separated from the code which is going to
implement its behavior. Save it in your `html` directory:

```html
<!doctype html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <title>Shopping Calculator</title>
    <!--[if lt IE 9]>
    <script src="http://html5shiv.googlecode.com/svn/trunk/html5.js"></script>
    <![endif]-->
    <link rel="stylesheet" href="css/styles.css">
</head>
<body>
  <!-- shopping.html -->
  <form action="" method="post" id="shoppingForm" novalidate>
    <legend> Shopping Calculator</legend>
    <fieldset>
      <div>
        <label for="quantity">Quantity</label>
        <input type="number"
               name="quantity"
               id="quantity"
               value="1"
               min="1" required>
      </div>
      <div>
        <label for="price">Price Per Unit</label>
        <input type="text"
               name="price"
               id="price"
               value="1.00"
               required>
      </div>
      <div>
        <label for="tax">Tax Rate (%)</label>
        <input type="text"
               name="tax"
               id="tax"
               value="0.0"
               required>
      </div>
      <div>
        <label for="discount">Discount</label>
        <input type="text"
               name="discount"
               id="discount"
               value="0.00" required>
      </div>
      <div>
        <label for="total">Total</label>
        <input type="text"
               name="total"
               id="total"
               value="0.00">
      </div>
      <div>
        <input type="submit"
               value="Calculate"
               id="submit">
      </div>
    </fieldset>
  </form>
  <script src="js/main.js"></script>
</body>
</html>
```

As before, we included the link to `js/main.js`, the  external JS file
which will be generated by the CLJS compilation. Note that this time
we have not attached any value to the `action` attribute of the
form. That's because in this new example there is no server-side form
submission.

Again, as soon as you save the file, the IFDE will reload it as you
can confirm by visiting the `http://localhost:3000/shopping.html` URL.

![Shopping Page][9]

If your Developer Tool is open, the console will notify you about the
following error:

```bash
Uncaught TypeError: Cannot set property 'onsubmit' of null
```

pertaining the `login.cljs` CLJS source file. This is because
`shopping.html` does not have any `loginForm` id available on which to
set the value for the `onsubmit` property. At the moment, we don't care
about this issue.

### bREPLing with the Shopping Calculator

Before starting to define the function for calculating the total
amount you're going to spend, let's play a little bit by bREPLing
with the Shopping Calculator form and its field ids:

```clj
cljs.user=> (value (by-id "quantity"))
"1"
cljs.user=> (value (by-id "price"))
"1.00"
cljs.user=> (value (by-id "tax"))
"0.0"
cljs.user=> (value (by-id "discount"))
"0.00"
```

You can even test the calculation of the total amount from the bREPL:

```clj
cljs.user> (let [quantity (value (by-id "quantity"))
                 price (value (by-id "price"))
                 tax (value (by-id "tax"))
                 discount (value (by-id "discount"))]
             (-> (* quantity price)
                             (* (+ 1 (/ tax 100)))
                             (- discount)
                             (.toFixed 2)))
"1.00"
```

Not so bad, eh!

If you don't know about the thread macros `->` and `->>`, I strongly suggest to
take some time to watch this awesome [video tutorial][14].

## Create `shopping.cljs`

Now it's time to freeze the above experimental tests into a source
file. Remember that we have to write back the total amount into the
shopping form and return `false` to prevent the browser from
attempting to submit the data to a server-side script.

Create the `shopping.cljs` file in the `src/cljs/modern_cljs` directory and
enter the following code:

```clj
(ns modern-cljs.shopping
  (:require [domina.core :refer [by-id value set-value!]]))

(defn calculate []
  (let [quantity (value (by-id "quantity"))
        price (value (by-id "price"))
        tax (value (by-id "tax"))
        discount (value (by-id "discount"))]
    (set-value! (by-id "total") (-> (* quantity price)
                                    (* (+ 1 (/ tax 100)))
                                    (- discount)
                                    (.toFixed 2)))
    false))

(defn init []
  (if (and js/document
           (.-getElementById js/document))
    (let [the-form (by-id "shoppingForm")]
      (set! (.-onsubmit the-form) calculate))))

(set! (.-onload js/window) init)
```

Here we created the `modern-cljs.shopping` namespace and required the
`domina.core` namespace, referring `by-id`, `value`, and
`set-value!` into the current namespace.

We then retrieved the values required to calculate the total amount, setting
it as the value of the `total` field of the form, before finally returning
the `false` boolean value.

The `init` function is almost identical to the one defined for the
previous login form. At last we set the `init` function itself as the
value of the `onload` property of the `window` object defined in the
`js` special namespace.

As soon as you save the `shopping.cljs` file, it gets recompiled and
the corresponding `shopping.html` page is reloaded.

But we still have to make another change. We need to add the newly
created `modern-cljs.shopping` namespace to the required section of
the `main.cljs.edn` file living in the `html/js` directory.

Open the `html/js/main.cljs.edn` file and enter the above addition:

```clj
{:require [modern-cljs.core modern-cljs.login modern-cljs.shopping]
 :compiler-options {:asset-path "js/main.out"}}
```

> NOTE 4: this an incidental complexity produced by the `boot-cljs`
> task. Do you remember when at the beginning of this series I wrote
> about the idiosyncrasies of the building tools when you walk a
> little bit out of its defaults? This is one of them. Here you have to
> choose between two alternatives: either you adhere to the very widespread
> convention of keeping the `js` resources confined in a `js`
> subdirectory of the directory where your html pages live; or, to
> not incur the above incidental complexity, you stay with the
> defaults of `boot-cljs` task and abandon an almost
> universal convention of the web.

### bREPLing with the calculator

Go back to the bREPL and require the newly created namespace to play
with the shopping calculator:

```clj
cljs.user> (require '[modern-cljs.shopping :as s] :reload)
nil
cljs.user> s/calculate
#object[modern_cljs$shopping$calculate "function modern_cljs$shopping$calculate(){
var quantity = domina.value.call(null,domina.by_id.call(null,"quantity"));
var price = domina.value.call(null,domina.by_id.call(null,"price"));
var tax = domina.value.call(null,domina.by_id.call(null,"tax"));
var discount = domina.value.call(null,domina.by_id.call(null,"discount"));
domina.set_value_BANG_.call(null,domina.by_id.call(null,"total"),(((quantity * price) * ((1) + (tax / (100)))) - discount).toFixed((2)));

return false;
}"]
cljs.user>(s/calculate)
false
```

As you see, the `calculate` function displays the correct value in the
shopping form and returns `false` as expected.

But now there is a bad surprise waiting for you at the corner that has
to do with the `onsubmit` error we received before.

### A short troubleshooting session

Let's go back to the login form by visiting the
`http://localhost:3000/index.html` URL and click the Login button.

Oops, you got the `Page Not Found` notification. What happened?

Revisit the `index.html` page and go back to your active bREPL
session. Just to be sure, require again the `modern-cljs.login`
namespace and call the `l/validate-form` function:

```clj
cljs.user> (require '[modern-cljs.login :as l] :reload)
nil
cljs.user> (l/validate-form)
false
cljs.user>
```

As you see that function is still working as expected and the previous
error is not very informative. We have not yet introduced any
debugging tools to be used in such a case, so we try troubleshooting
with what we have in our hands.

As we saw the `validate-form` function is correctly called in the
bREPL, but not by the `Login` button of the form. Let's see if the
bREPL may help us in investigating the problem we have:

```clj
cljs.user> (.-onsubmit (by-id "loginForm"))
nil
```

Oops, the `onsubmit` property of the `loginForm` form element has no
value. `validate-form` should have been set as its value by the `init`
function which, in turn, should have been set as the value of the
`onload` property of the `window` object. Let's now see what's the
value of the `onload` property of the `window` object:

```clj
cljs.user=> (.-onload js/window)
#object[modern_cljs$login$init "function modern_cljs$login$init(){
if(cljs.core.truth_((function (){var and__7101__auto__ = document;
if(cljs.core.truth_(and__7101__auto__)){
return document.getElementById;
} else {
return and__7101__auto__;
}
})())){
var login_form = document.getElementById("loginForm");
return login_form.onsubmit = modern_cljs.login.validate_form;
} else {
return null;
}
}"]
```

Even more mysteriously, the `init` function is the correct one, i.e. the one associated to the login form.

What just happened has to do with the CLJS/Google Closure Compiler
pair. They get every CLJS file from the `:source-paths` we set in the
very [first tutorial][10] and compile all of them into the single
`js/main.js` file we set in the same tutorial.

To temporarily solve this problem, evaluate the `init` function in
the bREPL as follows:

```clj
cljs.user> (l/init)
#object[modern_cljs$login$validate_form "function modern_cljs$login$validate_form(){
if(((cljs.core.count.call(null,domina.value.call(null,domina.by_id.call(null,"email"))) > (0))) && ((cljs.core.count.call(null,domina.value.call(null,domina.by_id.call(null,"password"))) > (0)))){
return true;
} else {
alert("Please, complete the form!");

return false;
}
}"]
```

If you now evaluate again the above expression to get the function
associated with the `onsubmit` property of the `loginForm`, you'll get
the right answer:

```clj
cljs.user> (.-onsubmit (by-id "loginForm"))
#object[modern_cljs$login$validate_form "function modern_cljs$login$validate_form(){
if(((cljs.core.count.call(null,domina.value.call(null,domina.by_id.call(null,"email"))) > (0))) && ((cljs.core.count.call(null,domina.value.call(null,domina.by_id.call(null,"password"))) > (0)))){
return true;
} else {
alert("Please, complete the form!");

return false;
}
}"]
```

You can now use the login form as usual by clicking its Login button.

You should now stop any `boot` related process and reset your git
repository:

```bash
git reset --hard
```

# Next Step [Tutorial 6: The Easy Made Complex and Simple Made Easy][12]

In the [next tutorial][12] we're going to investigate and solve in two
different ways the problem we have just met.

# License

Copyright © Mimmo Cosenza, 2012-2015. Released under the Eclipse Public
License, the same as Clojure.

[1]: https://github.com/levand/domina
[2]: https://github.com/magomimmo/modern-cljs/blob/master/doc/second-edition/tutorial-04.md
[3]: https://github.com/brentonashworth/one
[4]: https://github.com/brentonashworth/one/wiki/Design-and-templating
[5]: https://github.com/weavejester/hiccup
[6]: http://www.larryullman.com/books/modern-javascript-develop-and-design/
[7]: https://github.com/clojure/clojurescript/wiki/Differences-from-Clojure#host-interop
[8]: https://github.com/clojure/clojurescript/wiki/Differences-from-Clojure#namespaces
[9]: https://raw.github.com/magomimmo/modern-cljs/master/doc/images/shopping.png
[10]: https://github.com/magomimmo/modern-cljs/blob/master/doc/second-edition/tutorial-01.md
[11]: https://raw.github.com/magomimmo/modern-cljs/master/doc/images/total.png
[12]: https://github.com/magomimmo/modern-cljs/blob/master/doc/second-edition/tutorial-06.md
[13]: https://help.github.com/articles/set-up-git
[14]: https://youtu.be/qxE5wDbt964
[15]: https://clojars.org/org.clojars.magomimmo/domina
