---
layout: getting-started
title: キーワードリストとマップ
redirect_from: /getting-started/maps-and-dicts.html
---

# {{ page.title }}

{% include toc.html %}

今までのところ、とある値をキーと関連づけすることが出来るようなデータ構造についてまだ触れていません。言語によってはそういうデータ構造のことを辞書、ハッシュ、あるいは連想配列という風に異なる名前で呼んでいます。

Elixir では主にキーワードリストとマップという二つの連想データ構造があります。これらについて学んでいきましょう。

## キーワードリスト

大抵の関数型プログラミング言語は、キーと値を表現する為にタプルのリストを使うのが一般的です。Elixir では、タプルの最初にキーとしてアトムを使っているリストのことを、キーワードリストと呼んでいます。

```iex
iex> list = [{:a, 1}, {:b, 2}]
[a: 1, b: 2]
iex> list == [a: 1, b: 2]
true
```

上のコードで分かる通り、Elixir は同じことを `[key: value]` という風に書ける特別な構文をサポートしています。キーワードリストはリストですから、リストに対して行える全ての操作がキーワードリストにも利用できます。例えば、新しく値を追加する為に `++` を使うことができます。

```iex
iex> list ++ [c: 3]
[a: 1, b: 2, c: 3]
iex> [a: 0] ++ list
[a: 0, a: 1, b: 2]
```

探索する際に引き出される値が先頭に追加された値であることに注意してください。

```iex
iex> new_list = [a: 0] ++ list
[a: 0, a: 1, b: 2]
iex> new_list[:a]
0
```

キーワードリストは以下に挙げる 3 つに特徴づけられた重要なものです。

  * キーはアトムである
  * キーは開発者が指定した通りに並ぶ
  * キーはいくつでも与えられる

例えば、[Ecto ライブラリ](https://github.com/elixir-lang/ecto) ではデータベースのクエリを書く為に使われる洗練された DSL を提供する為にそれら機能を利用しています。

```elixir
query = from w in Weather,
      where: w.prcp > 0,
      where: w.temp < 20,
     select: w
```

Elixir で関数にオプションを渡す手法がキーワードリストとなっているのは、これらの特性によるきっかけです。5 章でマクロの `if/2` について触れた時に、以下のような構文がサポートされていることを言及しました。

```iex
iex> if false, do: :this, else: :that
:that
```

`do:` と `else:` という二つのペアはキーワードリストなのです！実際に上記の呼び出しは以下と同等です。

```iex
iex> if(false, [do: :this, else: :that])
:that
```

以下もまた上記と同じように、どちらも等しいものです。

```iex
iex> if(false, [{:do, :this}, {:else, :that}])
:that
```

一般的にキーワードリストが関数の最後の引数である時は、角括弧を任意で省略できます。

キーワードリストでパターンマッチを利用できるのですが、要素の数とその順序が一致している必要がある為に殆ど利用されません。

```iex
iex> [a: a] = [a: 1]
[a: 1]
iex> a
1
iex> [a: a] = [a: 1, b: 2]
** (MatchError) no match of right hand side value: [a: 1, b: 2]
iex> [b: b, a: a] = [a: 1, b: 2]
** (MatchError) no match of right hand side value: [a: 1, b: 2]
```

Elixir はキーワードリストを操作する為に [`Keyword` モジュール](https://hexdocs.pm/elixir/Keyword.html) を提供しています。ただし、キーワードリストは単なるリストであり、それ自体はリストとして同じ線形的な運用特性があることは覚えておいてください。リストが長いほどキーの検索や要素数のカウントに時間がかかります。その理由は、Elixir のキーワードリストが主にオプション値を渡す為に使われる為です。要素をたくさん保持したり、一つのキーが一つの値とだけ関連づけられることの保証が必要な場合は、キーワードリストの代わりにマップを使うべきです。

## マップ

キーと値のペアで保持したいものがある時にはマップが打ってつけのデータ構造です。マップは `%{}` という構文を使って作成します。

```iex
iex> map = %{:a => 1, 2 => :b}
%{2 => :b, :a => 1}
iex> map[:a]
1
iex> map[2]
:b
iex> map[:c]
nil
```

キーワードリストと比較すると二つの違いが見て取れます。

  * マップはどんな値もキーにできる
  * マップのキーは順序がない

キーワードリストと違ってマップのパターンマッチは非常に有用です。パターンにマップを使うと、与えられた値のサブセットとマッチします。

```iex
iex> %{} = %{:a => 1, 2 => :b}
%{2 => :b, :a => 1}
iex> %{:a => a} = %{:a => 1, 2 => :b}
%{2 => :b, :a => 1}
iex> a
1
iex> %{:c => c} = %{:a => 1, 2 => :b}
** (MatchError) no match of right hand side value: %{2 => :b, :a => 1}
```

見ての通り、パターン内のキーが与えられたマップ内に存在する間はマッチします。よって、空のマップは完全にどのマップとも一致します。

変数を使って値にアクセスしたり、マッチングやキーを追加することが出来ます。

```iex
iex> n = 1
1
iex> map = %{n => :one}
%{1 => :one}
iex> map[n]
:one
iex> %{^n => :one} = %{1 => :one, 2 => :two, 3 => :three}
%{1 => :one, 2 => :two, 3 => :three}
```

[`Map` モジュール](https://hexdocs.pm/elixir/Map.html) は `Keyword` モジュールが備えている便利な関数とよく似た API をマップの操作の為に提供しています。

```iex
iex> Map.get(%{:a => 1, 2 => :b}, :a)
1
iex> Map.put(%{:a => 1, 2 => :b}, :c, 3)
%{2 => :b, :a => 1, :c => 3}
iex> Map.to_list(%{:a => 1, 2 => :b})
[{2, :b}, {:a, 1}]
```

マップはキーが持つ値を更新する為に以下の構文をサポートします。

```iex
iex> map = %{:a => 1, 2 => :b}
%{2 => :b, :a => 1}

iex> %{map | 2 => "two"}
%{2 => "two", :a => 1}
iex> %{map | :c => 3}
** (KeyError) key :c not found in: %{2 => :b, :a => 1}
```

上記の構文は与えられたキーが存在する必要があります。これを新しいキーの追加には使えません。`:c` というキーを使った例では、マップ内に `:c` が存在しないので失敗しています。

マップ内のすべてのキーがアトムの時は便利なキーワード構文を使って簡潔に書けます。

```iex
iex> map = %{a: 1, b: 2}
%{a: 1, b: 2}
```

マップのもう一つ興味深い特質として、アトムのキーにアクセスする独自の構文が用意されていることが挙げられます。

```iex
iex> map = %{:a => 1, 2 => :b}
%{2 => :b, :a => 1}

iex> map.a
1
iex> map.c
** (KeyError) key :c not found in: %{2 => :b, :a => 1}
```

一般的に Elixir を使う開発者は、マップを用いた操作で `Map` モジュールの関数を使うよりも、パターンマッチと `map.field` という構文を好んで使います。その方が確定的なコードを書けるようになるからです。[このブログ記事](http://blog.plataformatec.com.br/2014/09/writing-assertive-code-with-elixir/) では、Elixir の確定的なコードで如何に簡潔かつ高速なソフトウェアを手に入れるかの例と洞察を紹介しています。

Note: マップは最近になって Erlang <abbr title="Virtual Machine">VM</abbr> に導入されたばかりで、何百万という沢山のキーを効率的に保持できるのは Elixir v1.2 以降だけです。もしも今、あなたが以前のバージョンの Elixir (v1.0 or v1.1) を使っていて、少なくとも数百のキーをサポートする必要があるなら、[`HashDict` モジュール](https://hexdocs.pm/elixir/HashDict.html) の使用を検討してもよいかも知れません。

## ネストされたデータ構造

マップの中でさらにマップやキーワードリスト等が必要なケースが往々にしてあります。Elixir は `put_in/2` や `update_in/2` 及び、命令型言語に見られるものと同様に便利なその他のマクロよって、属性をイミュータブルに保ったまま多次元のデータ構造を操作するという利便性を提供しています。

以下のような構造があるとします。

```iex
iex> users = [
  john: %{name: "John", age: 27, languages: ["Erlang", "Ruby", "Elixir"]},
  mary: %{name: "Mary", age: 29, languages: ["Elixir", "F#", "Clojure"]}
]
[john: %{age: 27, languages: ["Erlang", "Ruby", "Elixir"], name: "John"},
 mary: %{age: 29, languages: ["Elixir", "F#", "Clojure"], name: "Mary"}]
```

名前と年齢、そして各ユーザーが好きな言語リストで構成されるマップを含んだキーワードリストがあります。John の年齢にアクセスしたい時はこう書きます。

```iex
iex> users[:john].age
27
```

これと同じ構文を値の更新に使うこともできます。

```iex
iex> users = put_in users[:john].age, 31
[john: %{age: 31, languages: ["Erlang", "Ruby", "Elixir"], name: "John"},
 mary: %{age: 29, languages: ["Elixir", "F#", "Clojure"], name: "Mary"}]
```

マクロ `update_in/2` もこれと似ていますが、値をどのように変更するかを制御する為に関数を渡すことができます。例えば、Mary の言語リストから "Clojure" を削除してみましょう。

```iex
iex> users = update_in users[:mary].languages, fn languages -> List.delete(languages, "Clojure") end
[john: %{age: 31, languages: ["Erlang", "Ruby", "Elixir"], name: "John"},
 mary: %{age: 29, languages: ["Elixir", "F#"], name: "Mary"}]
```

値を取り除いたり、データ構造を一度に更新するのことができる `get_and_update_in/2` も含めて `put_in/2` と `update_in/2` をもっと知る必要があるでしょう。動的にデータ構造へアクセスできる `put_in/3` と `update_in/3` 、 `get_and_update_in/3` もあります。それぞれの詳しいドキュメントは [`Kernel` モジュール](https://hexdocs.pm/elixir/Kernel.html) を参照してください。

ここで Elixir における連想データ構造の紹介を終わりとします。キーワードリストとマップがある時に、あなたはいつでも連想データ構造を必要とする問題に立ち向かう為の適切なツールがあることが分かると思います。
