---
layout: getting-started
title: case, cond, and if
---

# {{ page.title }}

{% include toc.html %}

この章では、 case, cond, if といった制御構造について学んでいきます。

## `case`

`case` を使うと、いくつかあるパターンのいずれかにマッチするまで値を比較することができます。

```iex
iex> case {1, 2, 3} do
...>   {4, 5, 6} ->
...>     "This clause won't match"
...>   {1, x, 3} ->
...>     "This clause will match and bind x to 2 in this clause"
...>   _ ->
...>     "This clause would match any value"
...> end
"This clause will match and bind x to 2 in this clause"
```

すでに存在している変数に対してパターンマッチを行うには `^` 演算子を使います。

```iex
iex> x = 1
1
iex> case 10 do
...>   ^x -> "Won't match"
...>   _ -> "Will match"
...> end
"Will match"
```

節はガードを使って条件を指定することもできます。

```iex
iex> case {1, 2, 3} do
...>   {1, x, 3} when x > 0 ->
...>     "Will match"
...>   _ ->
...>     "Would match, if guard condition were not satisfied"
...> end
"Will match"
```

上記の例にある最初の節では `x` が整数の時にマッチします。

ガードにおけるエラーが処理を抜けることはなく、ただガードに失敗するだけであると覚えておいてください。

```iex
iex> hd(1)
** (ArgumentError) argument error
iex> case 1 do
...>   x when hd(x) -> "Won't match"
...>   x -> "Got #{x}"
...> end
"Got 1"
```

いずれの節にもマッチしない時はエラーになります。

```iex
iex> case :ok do
...>   :error -> "Won't match"
...> end
** (CaseClauseError) no case clause matching: :ok
```

ガードの使い方や利用できる式について知りたい時は [the full documentation for guards](https://hexdocs.pm/elixir/guards.html) で詳しい情報を得られます。

NOTE: 無名関数は複数の節とガードを含むこともできます。

```iex
iex> f = fn
...>   x, y when x > 0 -> x + y
...>   x, y -> x * y
...> end
#Function<12.71889879/2 in :erl_eval.expr/5>
iex> f.(1, 3)
4
iex> f.(-1, 3)
-3
```

無名関数の中で節は引数の数が同じでなくてはならず、その数が合わなければエラーになります。

```iex
iex> f2 = fn
...>   x, y when x > 0 -> x + y
...>   x, y, z -> x * y + z
...> end
** (CompileError) iex:1: cannot mix clauses with different arities in anonymous functions
```

## `cond`

`case` は様々な値に対してマッチさせたい時に有用です。それでも、様々な条件を調べて最初に true と評価されるものを見つけたいことが往々にしてあります。そういった場合に取る方法のひとつが `cond` です。

```iex
iex> cond do
...>   2 + 2 == 5 ->
...>     "This will not be true"
...>   2 * 2 == 3 ->
...>     "Nor this"
...>   1 + 1 == 2 ->
...>     "But this will"
...> end
"But this will"
```

これは多くの命令型言語における `else if` に相当するものです(ここではあまり使われません)。

true を返す条件が見つからないと (`CondClauseError`) になります。この理由は、そういった時に最後では必ず `true` に等しくなる条件を書かなくてはならない為です。

```iex
iex> cond do
...>   2 + 2 == 5 ->
...>     "This is never true"
...>   2 * 2 == 3 ->
...>     "Nor this"
...>   true ->
...>     "This is always true (equivalent to else)"
...> end
"This is always true (equivalent to else)"
```

最後に、 `cond` は `nil` と `false` 以外の如何なる値も `true` と見なすことを覚えておきましょう。

```iex
iex> cond do
...>   hd([1, 2, 3]) ->
...>     "1 is considered as true"
...> end
"1 is considered as true"
```

## `if` and `unless`

`case` と `cond` の他、ひとつの条件だけを検証したい時に有用なマクロ `if/2` と `unless/2` も使えます。

```iex
iex> if true do
...>   "This works!"
...> end
"This works!"
iex> unless true do
...>   "This will never be seen"
...> end
nil
```

`false` と `nil` を返す条件では `do/end` のブロック間にある内容が実行されず、代わりに `nil` を返します。 `unless` ではこの逆になります。

これらはブロックで `else` も使えます。

```iex
iex> if nil do
...>   "This won't be seen"
...> else
...>   "This will"
...> end
"This will"
```

`if/2` と `unless/2` がマクロとして実行されることに関して興味深いところですが、他の多くの言語でもそうであるように、取り立てるほど特別な構造ではありません。これらに関するドキュメントや `if/2` のソースは [the `Kernel` module docs](https://hexdocs.pm/elixir/Kernel.html) で確認できます。 `Kernel` モジュールには `+/2` のような演算子や `is_function/2` といった関数も定義されており、すべて自動的にインポートされるので、あなたのコードはデフォルトでそれらを利用することができます。

## `do/end` blocks

ここまで `case`, `cond`, `if`, `unless` という4つの制御構造を学びましたが、これらはすべて `do/end` ブロックに包含されていました。しかし、よくある書き方として `if` を以下のようにすることもできたのです。

```iex
iex> if true, do: 1 + 2
3
```

注目して欲しいのは、どうしてこの例で `true` と `do:` の間にコンマが置かれているのかです。この理由は、各引数がコンマによって区切られなければいけないという基本的な文法によります。つまり、この文法は *キーワードリスト* を使っているということなのです。キーワードに `else` を渡して使うこともできます。

```iex
iex> if false, do: :this, else: :that
:that
```

`do/end` ブロックは文法的な便宜です。では、なぜ `do/end` ブロックは前にある引数やブロックとの間にコンマを必要としないのでしょう。以下の二つは同等になります。

```iex
iex> if true do
...>   a = 1 + 2
...>   a + 10
...> end
13
iex> if true, do: (
...>   a = 1 + 2
...>   a + 10
...> )
13
```

`do/end` ブロックを使う場合にひとつ留意して欲しいのが、これらは一番外側で呼ばれた関数に対して働く点です。例えば以下の表現は、`if` のキーワードとしては認められません。

```iex
iex> is_number if true do
...>  1 + 2
...> end
** (CompileError) iex:1: undefined function is_number/2
```

その為、実際には以下のように解釈されることになってしまいます。

```iex
iex> is_number(if true) do
...>  1 + 2
...> end
** (CompileError) iex:1: undefined function is_number/2
```

無理なお願いをしたことで起こった undefined function のエラーです。 `is_number/2` は存在しないと言っています。`if true` という式はブロックが必要になる為、これ自体が無効な表現となります。 `is_number/2` のアリティとは一致しません。ゆえに Elixir はそれを評価するにさえ至らないのです。

`if` にブロックを括り付けるには明確に丸括弧で囲むだけで十分です。

```iex
iex> is_number(if true do
...>  1 + 2
...> end)
true
```

キーワードリストはとても重要な役割を果たしており、多くの関数やマクロにおいて普遍的なものです。それについてはまた別の章で取り上げる事として、そろそろ "バイナリ、文字列、文字リスト" の紹介してもいい頃でしょう。
