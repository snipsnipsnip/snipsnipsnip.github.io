---
layout: page
title: dollar-expression ($式)
date: 2013-11-24 02:32
excerpt: 
  $式(どるしき)は、閉じ括弧を省略できるS式の構文糖です。
---

~~~
$ define $ partition pred lis
  $ let recur $ $ lis lis
    $ if $ null-list? lis
      $ values lis lis
      $ let $ $ elt  $ car lis
              $ tail $ cdr lis
        $ receive (in out) $ recur tail
          $ if $ pred elt
            $ values $ if (pair? out) (cons elt in) lis
                     out
            $ values in
                     $ if (pair? in) (cons elt out) lis

~~~

## もくじ
{:.no_toc}

* toc
{:toc}

## 概要 ##

[上のような文法を普通のS式にする変換器をGaucheで書いてみた](https://gist.github.com/snipsnipsnip/7620314)。

$式(どるしき)は、閉じ括弧を省略できるS式の構文糖。

$式の字句構造はほぼS式と同様。
ただし、`$`を「開き括弧マーカー」と呼び、特別に扱う。

マーカは、そこよりも右下に始点があるトークンをすべて括弧に囲む。
トークンの始点とは、そのトークンを構成する最初の文字のこと。
構文糖の解釈後マーカ自身は消えてなくなり、代わりに開き括弧が置かれる。
それに対応する閉じ括弧は、`$`よりも左下か真下に始点があるトークンが現れたとき、その手前に挿入される。

たとえば、以下の内容は

~~~
$ "foo
bar" $ baz
   quu
"qux"

~~~
次のように書いたのと同様になる。

~~~
( "foo
bar" (baz)
  quu)
"qux"

~~~
基本的には「$から右下方向に矩形選択する」という感じだが、文字列などがぶつかっても大丈夫にするために始点だけを見るようにしている。

## Q&A ##

元がふわふわしたアイデアなので、細かい所にかなり抜けがある。Q&A方式で文法の細かいところを考えていく。

組み合わせによって必要な処理が変わってくるので、ここでは実装がラクなこと、移行がラクなこと、snipsnipsnipの好みを選択の基準とする。

Q&Aが他にあれば追記をお願いします。([fork me!]({{site.repos_url}}/{{page.path}}))

### 括弧の中で構文糖は使えますか？

A1 (採用)
: いいえ。内部では$式の構文糖は使えません。通常のS式として解釈されます。

A2 (没)
: はい。内部でも$式の構文糖が使えます。

A3 (没)
: そもそも括弧は使えません。`$`だけでやってください。

これによって$式はS式のスーパーセットになるので、混ぜて書けるようになる。

閉じ括弧の対応を考えなくてすむ。

既存のread手続きが再利用できるので実装がラクになる。

### シンボルの`$`を使いたいときはどうしますか？

A1 (採用)
: `|$|`を代わりに使って下さい。または、$を使いたい時は括弧の中で使ってください。

A2 (没)
: 使えません。

A3 (没)
: `$$`を代わりに使って下さい。`$$`を`$`、`$$$`を`$$`のように`$`を一つ減らして解釈します。

理不尽さや制限が他の案に比べて少ない。

Gaucheでは`$`というマクロがあるが、この構文糖自体と目的がかぶっているのであまり困らないはず。

### `$hoge`や`hoge$`と書いたらどうなりますか？


A1 (採用)
: シンボル`$hoge`や`hoge$`として扱われます。`$`は空白で区切られる必要があります。

A2 (没)
: `(hoge ...`や`hoge(...`として扱われます。`$`はどこでも開き括弧と同様に扱われます。

`$`はシンボルに許される文字なので、シンボルに隣接した場合エスケープの必要がある。

Gaucheの`pa$`や`reduce$`を個人的によく使うので不便。

A1にすると後述する落とし穴をある程度防ぐことができる。

### `$$`と書いたらどうなりますか？

A1 (採用)
: シンボル`$$`として扱われます。

A2 (没)
: `((`として扱われます。

上の動作との一貫性。また、後述する落とし穴をある程度防ぐことができる。

### `'$`や`` `$ ``などはどうなりますか？

A1 (採用)
: 構文糖にはなりません。`(quote |$|)`として読まれます。
    構文糖で準クォートのリストを書きたい場合は`$ quasiquote $ ...`と平で書いて下さい。

A2 (没)
: 構文糖になります。`(quote (マーカーに囲まれる内容))`として読まれます。

`'`は`quote`の構文糖なので、これは構文糖どうしの優先順位の話になる。

こう書いた時`()`と`$`のどちらが表示されるかという話。A1では $、A2では () がプリントされる。

~~~
$ write '$
~~~

どちらもありえるが、個人的な好みで上にした。

また、`'foo`が`(quote foo)`の構文糖と考えると、
A1のほうが「括弧の内部には$式の構文糖は及ばない」というルールと一貫性がある。

### `#$`と書いたらベクタリテラル`#( ... )`の構文糖になりますか？

A1 (採用)
: いいえ。構文糖で準クォートのリストを書きたい場合は $ vector ... と平で書いて下さい。

A2 (没)
: はい。`#$ 1 2 3`とすれば`#( 1 2 3 )`として構文糖になります。

便利そうだが、#$をどうエスケープしていいかわからないので見送り。

### `$ )`と書いたらどうなりますか？

A1 (採用)
: `( ) )`と解釈され、エラーになります。

A2 (没)
: `( )`と解釈され、エラーは起こりません。

`$`を`)`で閉じられたら便利かもしれない。でも(今更だが)エディタが混乱するので見送り。

### 行が右に長くなってきたら折り返しはできますか？

A1 (採用)
: いいえ。折り返したい場合は括弧の中に囲んで通常のS式として書いて下さい。

A2 (没)
: はい。`\`を行末に書くとその次の行は前の行に続いたものとして扱います。

あれば便利だが、括弧で一応解決できるし、ルールは少ないほうがよい。

### ドット対を`$`で囲むことはできますか？

A1 (採用)
: いいえ。ドット対は括弧の中だけで使えます。

A1 (没)
: はい。

どう考えても囲めて当然だが、実装が面倒なので見送り。

readは単独の.を読むとエラーになるので、使い回しができなくなる。もしくは先読みのためトークナイザの一部を作らねばならなくなる。本当に手抜きでよいならreadしてエラーを捕捉する手があるが、普通の文法エラーとの兼ね合いが不安。

### マーカーは`$`以外でもよかったのでは？

A1 (採用)
: いいえ。$式という名前が気に入ったので。

A2 (没)
: はい。`(`を自動で閉じてもよいですね。

A3 (没)
: はい。`#$`など拡張用のトークンを使うほうが安全ですね。

これは完全に好み。どんな記号を選んでも理由はつくが、`$`や`:`や`!`は通常のシンボルとして許されるので若干弱い。単独で許されないので決して使われない`.`がよかったかもしれない。

最初は`(`のまま「自動括弧閉じ器」を作ってみたこともあったが、あまりに目が落ち着かなかったので諦めた。

アイデアの由来はHaskellの`$`(関数適用)演算子だが、$式は中置でなく前置になっている。

[SRFI 110: Sweet-expressions](http://srfi.schemers.org/srfi-110/srfi-110.html)の`$`は中置のままのようだ。中置だけだと`let`が綺麗に並ばない。

~~~
 |,
(|   ドルマークは開き括弧と閉じ括弧をまとめたように見える。
 |)  
'|
~~~

### なぜ$を離すか ###

condでたまたま処理の部分を2字ぶん下げたとする。

~~~
 (cond
   ((foo?)
     bar))
~~~
上の開き括弧を単純に$で置き換えるとこうなる。

~~~
 $cond
   $$foo?
     bar
~~~
これを$式で復元するとこうなって、元と構造が変わってしまう。

~~~
 (cond
   ((foo?
     bar)))
~~~

condの処理を条件部分よりも下げるとハマるというのは落とし穴として少し大きすぎる。
しかも文法エラーにならない。

これは$を離して書き、インデントを2字で統一することで問題がなくなる。

~~~
 $ cond
   $ $ foo?
     bar
~~~
とはいえ、これも好み。くっつけたままインデントを1字に統一するという後ろ向きな方法もあるので、このこと単体では$を全て離す理由にはならない。対処療法的な感はあるが、そもそもインデント文法でインデントを間違えたらどうしようもないので解決方法は思いつかない。

### 細かい仕様まとめ ###

* 括弧の内部はS式としてパースされ、$式の構文糖は無効になる。
* `$`は空白に囲まれた単独の場合でのみ有効で、それ以外の場合は全て通常のシンボルになる。
  * マーカーでなくシンボルの`$`を使いたい時は`|$|`と書く。
  * `$`は単独のシンボルのみ開き括弧マーカーとして認識される。
  
    `$hoge$`はそのまま`$hoge$`というシンボルになる。
  * `$$`はそのまま`$$`というシンボルになる。
  * `'$`は`(quote $)`になる。
* 開き括弧マーカー`$`で開かれた括弧は、閉じ括弧で閉じることはできない。字下げの関係だけで閉じる。
* ベクタリテラルの開き括弧マーカー版は無い。
* 行を途中で折り返す方法は$式には無い。そういう場合は括弧で囲んでS式に戻る。
* ドット対はS式の括弧の中でしか使えない。

## 読み込み器の実装

どちらも適当。

### 処理系に挟む場合

処理系に組み入れる場合は、quoteの処理がすんだ直後のあたりに処理を挟むとしてこんな具合だろう。(適当)

  1. 通常のS式として字句解析する。ただし、次のような特別扱いを加える。
   * 各トークンに、その始めの文字の行頭からのカラム位置をひも付けておく。
   * 開き括弧マーカーはシンボル`$`と区別する。
    つまり、縦棒で囲まれた`|$|`はシンボル`$`として扱うが、
    囲まれない`$`は開き括弧マーカーとして記憶する。
  2. quoteなどを処理しておく。
  3. トークン列を走査して次のように処理する。
     括弧ネストのカウンタと数値のスタックが必要。
      1. 通常の開き括弧の類があったら無視フラグを立てる。
      2. 1に対応する閉じ括弧を見つけたら無視フラグを下ろす。
      3. スタックトップにあるより左か真下にあるトークン(マーカー含む)が現れたら、
          そのトークンがスタックトップより右になるまで、
          トークンの直前に閉じ括弧を挿入し、ポップすることを繰り返す。
      4. 開き括弧マーカーを見つけたら、
         無視フラグが立っている場合、`$`シンボルに置き換える。
         無視フラグが立っていない場合、そのカラム位置をスタックに記憶して、
         開き括弧に置き換える。
  4. スタックに開き括弧が残っていれば、その分閉じ括弧を挿入する。
  5. トークン列から通常のS式と同様に木構造を組み立てる。

### readを再利用して手抜きする場合

[Gaucheで書いてみた](https://gist.github.com/snipsnipsnip/7620314)。

次のような具合に手抜きして既存の`read`を再利用できる。

  1. カラムを記憶しながら`read`を繰り返し、カラム数とS式のペア(以下$トークン)を集める。
     処理系でする場合と違ってリストが読み出されることもあるが、
     Q&Aで決めた通りなら問題ない。
     `read`の前に先読みで少し前処理をする。
       1. 空白を読み飛ばす。
       2. 開き括弧マーカー`$`とシンボル`|$|`を区別するため、一文字先読みする。
       3. <strike>ドット`.`をそのままreadするとエラーになるので、回避のため先読みする。
           </strike> 未実装。
  2. $トークンの列を走査して次のように処理する。
     カラム数とS式のリストのペアを要素とするスタックを用意する。
     最初はスタックに`(-1)`を入れておく。スタックは決して空にならない。
      1. スタックトップの物よりも$トークンが右にある(カラム数が大きい)場合は、
          スタックトップのリストの末尾に付け足す。
      2. スタックにあるより左か真下にある$トークンが現れたら、
         その$トークンがスタックトップより右になるまで、以下の操作を繰り返す。
         * ※ ポップしたリストを新たなスタックトップにあるリストの末尾に付け足す
      3. 開き括弧マーカーを見つけたら、
        スタックにカラム数つきの新しいリストをプッシュする。
  3. スタックが1要素になるまで※を繰り返し、残った最終的なS式のリストを得る。

## その他

### 既存のインデント志向の文法

[SRFI 49: Indentation-sensitive syntax](http://srfi.schemers.org/srfi-49/srfi-49.html)や[SRFI 110: Sweet-expressions](http://srfi.schemers.org/srfi-110/srfi-110.html)がある。これらは開き括弧を暗黙にしようとしているのが好みでない。

~~~
define factorial(n)
  if {n <= 1}
     1
     {n * factorial{n - 1}}
~~~

### TODO

* 逆変換器
* 実行器
* 変換器をブラウザで動かす
* [fork me!]({{site.repos_url}}/{{page.path}})