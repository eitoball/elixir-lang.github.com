---
layout: getting-started
title: try、catch、rescue
---

# {{ page.title }}

{% include toc.html %}

Elixir は3つのエラー機構を持っています。エラー、スロー、イグジットです。この章では、それらがいつ使われるべきかをみていきましょう。

## エラー

エラー（または例外）はコード内で予期しない出来事が起きた時に使われます。例えば数値をアトムに加えようとした時に、エラーを発生させることが出来ます。

```iex
iex> :foo + 1
** (ArithmeticError) bad argument in arithmetic expression
     :erlang.+(:foo, 1)
```

ランタイムエラーは `raise/1` を使うことでいつでも発生させることが出来ます。

```iex
iex> raise "oops"
** (RuntimeError) oops
```

それ以外のエラーは `raise/2` にエラー名とキーワード引数のリストを渡すことで発生させることが出来ます。

```iex
iex> raise ArgumentError, message: "invalid argument foo"
** (ArgumentError) invalid argument foo
```

モジュールを作り、その中で `defexception/1` マクロを使うことで、独自のエラーを定義することもできます。この方法では、モジュールと同じ名前のエラーを作ることが出来ます。最も一般的な使用法は、独自のメッセージ付きエラーを定義することです。

```iex
iex> defmodule MyError do
iex>   defexception message: "default message"
iex> end
iex> raise MyError
** (MyError) default message
iex> raise MyError, message: "custom message"
** (MyError) custom message
```

エラーは `try/rescue` 構造を用いることで **捕捉** することが出来ます。

```iex
iex> try do
...>   raise "oops"
...> rescue
...>   e in RuntimeError -> e
...> end
%RuntimeError{message: "oops"}
```

上の例ではランタイムエラーを補足し、 `ies` セッション内でエラーが表示されるように、そのエラー自身を返しています。

もしエラーを使う必要がないのであれば、エラー自身を返す必要はありません。

```iex
iex> try do
...>   raise "oops"
...> rescue
...>   RuntimeError -> "Error!"
...> end
"Error!"
```

しかし実際のところ、Elixirで開発中に `try/rescue` 構造を使うことはほとんどないでしょう。例えば、多くの言語ではファイルを開けなかったときにエラーになるので、あなたがそれを捕捉しなければならないようになっています。一方Elixirが提供する `File.read/1` 関数は、ファイルを開くことに成功したかどうかの情報を含んだタプルを返します。

```iex
iex> File.read "hello"
{:error, :enoent}
iex> File.write "hello", "world"
:ok
iex> File.read "hello"
{:ok, "world"}
```

そこに `try/rescue` は存在しません。ファイルを開く時の複数の結果を取り扱いたい場合は、 `case` 構造と一緒にパターンマッチングを使うことが出来ます。

```iex
iex> case File.read "hello" do
...>   {:ok, body}      -> IO.puts "Success: #{body}"
...>   {:error, reason} -> IO.puts "Error: #{reason}"
...> end
```

要するに、ファイルを開くときのエラーが例外かそうでないかを決めるのはあなたのアプリケーション次第です。だからこそElixirは `File.read/1` やその他の関数で例外を強制しません。その代りに、最も良い例外の処理方法を開発者に委ねます。

ファイルが存在することを期待している時（そしてそのファイルが存在しないときは本当にエラーである時）、 `File.read!/1` を使うでしょう。

```iex
iex> File.read! "unknown"
** (File.Error) could not read file unknown: no such file or directory
    (elixir) lib/file.ex:272: File.read!/1
```

標準ライブラリ内の多くの関数は、マッチするタプルを返す代わりに例外を発生させる、という反対のパターンに従います。規約では、 `foo` 関数を作るときは `{:ok, result}` か `{:error, reason}` タプルを返し、 `foo!` という、 `foo` と同じ名前に `!` がつき `foo` と同じ引数を取る関数は、エラーが生じたときは例外を発生させます。 `foo!` はすべてが上手く行った時は、タプルで囲わない結果を返すべきです。 [`File` モジュール](https://hexdocs.pm/elixir/File.html)  はこの規約についての良い例です。

Elixirでは、 `try/rescue` の使用を避けています。なぜなら、 **フローを制御するためにエラーを使わないから** です。私達はエラーを文字通りに受け取ります。エラーは予期しないまたは予期した例外的状況のために用意されています。フローを制御する構造が必要な場合は、 *スロー* が使われるべきです。それでは次にスローを見ていきましょう。

## スロー

Elixirでは、ある値を投げ、後で捕まえることができます。`trhow` と `catch` を使わなければ値を取ることが出来ない状況のために、 `throw` と `catch` は予約されています。

それらの状況は、適切なAPIが提供されていないライブラリに直面しなければ、実際にはかなり珍しいことです。例えば、 `Enum` モジュールが値を探すためのAPIを何一つ用意しておらず、数字のリストから最初の13の倍数を探す必要がある場合を想像してみてください。

```iex
iex> try do
...>   Enum.each -50..50, fn(x) ->
...>     if rem(x, 13) == 0, do: throw(x)
...>   end
...>   "Got nothing"
...> catch
...>   x -> "Got #{x}"
...> end
"Got -39"
```

`Enum` が適切なAPIを *提供している* のであれば、実際には `Enum.find/2` を使うのがベストです。

```iex
iex> Enum.find -50..50, &(rem(&1, 13) == 0)
-39
```

## イグジット

すべてのElixirのコードは、お互いに通信をし合うプロセス群の中で動作します。プロセスが"自然な原因"（未処理の例外など）で死んだ時、プロセスは `exit` 信号を送ります。プロセスは明示的に `exit` 信号を送ることで殺すことも出来ます。

```iex
iex> spawn_link fn -> exit(1) end
** (EXIT from #PID<0.56.0>) evaluator process exited with reason: 1
```

上の例ではリンクしていたプロセスが、1という値と一緒に `exit` 信号が送られ、死にました。Elixirシェルは自動的にこれらのメッセージを処理し、ターミナル上に表示します。

`exit` は `try/catch` を使うことでも "捕まえる" ことができます。

```iex
iex> try do
...>   exit "I am exiting"
...> catch
...>   :exit, _ -> "not really"
...> end
"not really"
```

`try/catch` を使うことは珍しく、それを使ってイグジットを捕まえることはさらに稀です。

`exit` 信号はErlang <abbr title="Virtual Machine">VM</abbr> によって提供される耐障害性の重要な要素です。プロセス群は通常、監視プロセス群からの `exit` 信号を待ち受けている、監視ツリーの元で動作します。ひとたび `exit` 信号を受け取ると、監視ストラテジーが実行に移され、監視されているプロセスは再起動されます。

Elixirで `try/catch` と `try/rescue` の構造を珍しくするのは、この監視の仕組のおかげです。エラーの後に既知の初期状態にアプリケーションを戻してくれることを監視ツリーが保証してくれるので、エラーを救う代わりに、むしろ "エラー時はただちにシステムを停止" させるのです。

## アフター

潜在的にエラーを起こす可能性のあるアクションの後に、リソースが片付けられていることを確実にすることが、時々必要になります。 `try/after` 構造はそれをできるようにします。例えば、ファイルを開いた時に `after` 句を使って、失敗したとしてもファイルを閉じることができます。

```iex
iex> {:ok, file} = File.open "sample", [:utf8, :write]
iex> try do
...>   IO.write file, "olá"
...>   raise "oops, something went wrong"
...> after
...>   File.close(file)
...> end
** (RuntimeError) oops, something went wrong
```

`after` 句は思考されたブロックが成功したかどうかに関わらず実行されます。しかしリンクされたプロセスが終了した場合は、
このプロセスが終了し `after` 句が実行されることはない、ということに注意してください。したがって、 `after` は簡易な保証しか提供しません。運良くElixir内のファイルが現在のプロセス群にリンクされており、そのために現在のプロセスがクラッシュした時に常にファイルが閉じられたとしても、 
`after` 句とは別の話です。ETSテーブルやソケット、ポートなどその他のリソースについても同じことが言えます。

時々、最後にいくつかのコードが実行されることを保証するために、関数内のすべてのコードを `try` 構造で囲みたいと思うかもしれません。そのような場合は、Elixirでは `try` 行を省略することができます。

```iex
iex> defmodule RunAfter do
...>   def without_even_trying do
...>     raise "oops"
...>   after
...>     IO.puts "cleaning up!"
...>   end
...> end
iex> RunAfter.without_even_trying
cleaning up!
** (RuntimeError) oops
```

Elixirは `after`、`rescue`、`catch` のいずれかが指定されている時は、自動的に関数の本体を `try` で囲みます。

## エルス

`else` ブロックが存在する場合は、`try` ブロックがエラーもスローもなく終了する度に、`try` ブロックの結果とマッチします。

```iex
iex> x = 2
2
iex> try do
...>   1 / x
...> rescue
...>   ArithmeticError ->
...>     :infinity
...> else
...>   y when y < 1 and y > -1 ->
...>     :small
...>   _ ->
...>     :large
...> end
:small
```

`else` ブロック内の例外は捕捉されません。`else` ブロック内のどのパターンともマッチしない場合、例外が起こされます。この例外は現在の `try/catch/rescue/after` ブロックでは捕捉されません。

## 変数のスコープ

`try/catch/rescue/after` ブロックの中で定義された変数は、その外側のコンテキストに漏れることがないということを心に留めておいてください。`try` ブロックは失敗するかもしれず、ブロック内の変数がそもそも見つかることが無いかもしれないからです。この不正なコードの例で見てみましょう。

```iex
iex> try do
...>   raise "fail"
...>   what_happened = :did_not_raise
...> rescue
...>   _ -> what_happened = :rescued
...> end
iex> what_happened
** (RuntimeError) undefined function: what_happened/0
```

その代りに、`try` 式の値を保存することが出来ます。

```iex
iex> what_happened =
...>   try do
...>     raise "fail"
...>     :did_not_raise
...>   rescue
...>     _ -> :rescued
...>   end
iex> what_happened
:rescued
```

`try`、`catch`、`rescue` についての紹介はこれで終わりです。他の言語と比べてElixirでは、それらがそれほど頻繁に使われないことにあなたは気づくでしょう。しかし、ライブラリや "ルールに従っていない" 一部のコードの中においては、役に立つかもしれません。
