---
title: "Ecto Dynamic Internals"
date: 2023-04-19T13:31:29-04:00
draft: false
toc: false
images:
tags: 
  - elixir, ecto
---

I've run into the `dynamic/2` function a few times and wanted to glean a deeper understanding. 

If you're building a Phoenix application that requires complex queries or flexible search functionality, you may find that Ecto's query syntax isn't always sufficient to express your needs. In these situations, you can turn to the `dynamic/2` function, which allows you to generate SQL queries at runtime using Elixir code. There are many reasons why you might want to use `dynamic/2` in your application, including increased flexibility, improved performance, enhanced expressiveness, greater interoperability, and finer control over your SQL queries. In this post, we'll explore how Ecto generates the `dynamic/2` code.

Let's explore a simple example of `dynamic/2` from the [Ecto docs](https://hexdocs.pm/ecto/dynamic-queries.html). Imagine that you have a table of blog posts and you want to allow users to search for posts based on a combination of different filters, such as the author, the category and the published_at date. In this example, the `dynamic/2` function allows us to build a query with optional filters based on the user's input. The `where` clause of the query is constructed dynamically based on the values of the `params` map passed in as an argument to the function.

```elixir
def filter(params) do
  where(Post, ^filter_where(params))
end

defp filter_where(params) do
  Enum.reduce(params, dynamic(true), fn
    {"author", value}, dynamic ->
      dynamic([p], ^dynamic and p.author == ^value)

    {"category", value}, dynamic ->
      dynamic([p], ^dynamic and p.category == ^value)

    {"published_at", value}, dynamic ->
      dynamic([p], ^dynamic and p.published_at > ^value)

    {_, _}, dynamic ->
      # Not a where parameter
      dynamic
  end)
end
```

Let's take a deeper dive into the how a query is constructed with `dynamic/2`:

```elixir
query = Post.filter(%{"author" => "eric", "category" => "elixir", "published_at" => "03/12/2019"})
#Ecto.Query<from p0 in "post",
 where: true and p0.author == ^"eric" and p0.category == ^"elixir" and p0.published_at > ^"03/12/2019">

IO.inspect(query, structs: false)
%{
  __struct__: Ecto.Query,
  aliases: %{},
  assocs: [],
  combinations: [],
  distinct: nil,
  from: %{
    __struct__: Ecto.Query.FromExpr,
    as: nil,
    file: nil,
    hints: [],
    line: nil,
    params: [],
    prefix: nil,
    source: {"post", nil}
  },
  group_bys: [],
  havings: [],
  joins: [],
  limit: nil,
  lock: nil,
  offset: nil,
  order_bys: [],
  prefix: nil,
  preloads: [],
  select: nil,
  sources: nil,
  updates: [],
  wheres: [
    %{
      __struct__: Ecto.Query.BooleanExpr,
      expr: {:and, [],
       [
         {:and, [],
          [
            {:and, [],
             [
               true,
               {:==, [],
                [{{:., [], [{:&, [], [0]}, :author]}, [], []}, {:^, [], [0]}]}
             ]},
            {:==, [],
             [{{:., [], [{:&, [], [0]}, :category]}, [], []}, {:^, [], [1]}]}
          ]},
         {:>, [],
          [{{:., [], [{:&, [], [0]}, :published_at]}, [], []}, {:^, [], [2]}]}
       ]},
      op: :and,
      params: [
        {"eric", {0, :author}},
        {"elixir", {0, :category}},
        {"03/12/2019", {0, :published_at}}
      ],
      subqueries: []
    }
  ],
  windows: [],
  with_ctes: nil
}
```

Within the nested `wheres` -> `expr` map is an AST with the compiled dynamic code:

```elixir
expr = {:and, [],
 [
   {:and, [],
    [
      {:and, [],
       [
         true,
         {:==, [],
          [{{:., [], [{:&, [], [0]}, :author]}, [], []}, {:^, [], [0]}]}
       ]},
      {:==, [], [{{:., [], [{:&, [], [0]}, :category]}, [], []}, {:^, [], [1]}]}
    ]},
   {:>, [], [{{:., [], [{:&, [], [0]}, :published_at]}, [], []}, {:^, [], [2]}]}
 ]}
iex(2)> Macro.to_string(expr)
"true and &0.author() == ^0 and &0.category() == ^1 and &0.published_at() > ^2"
 ```

The `0`, `1`, `2` are placeholders for the params list that will be injected into the Postgres query at runtime. Now, let's explore how the AST is constructed in Ecto. The `dynamic/2` is a macro defined [here](https://github.com/elixir-ecto/ecto/blob/f619559dd105a2e86d5b7903a3695b156de9e2ea/lib/ecto/query.ex#L655) and looks like:

```elixir
defmacro dynamic(binding \\ [], expr) do
  Builder.Dynamic.build(binding, expr, __CALLER__)
end
```

The `Query` module is interesting because it encapsulates all query building logic–-all queryable functions (`from`, `where`, `join`) are located here. The next step in the call stack is the `Builder.Dynamic.build/3` function looks like (with added logging):

```elixir
def build(binding, expr, env) do
  {query, vars} = Builder.escape_binding(quote(do: query), binding, env) 
  IO.inspect(query, label: "[1] escape_binding query")
  IO.inspect(vars, label: "[1] escape_binding vars")
  {expr, {params, acc}} = escape(expr, {[], %{subqueries: [], aliases: %{}}}, vars, env)
  IO.inspect(expr, label: "[2] escape expr")
  IO.inspect(params, label: "[2] escape params")
  IO.inspect(acc, label: "[2] escape acc")
  aliases = Builder.escape_select_aliases(acc.aliases) 
  IO.inspect(aliases, label: "[3] escape_select_aliases")
  params = Builder.escape_params(params) 
  IO.inspect(params, label: "[4] escape_params")

  quote do
    %Ecto.Query.DynamicExpr{fun: fn query ->
                              _ = unquote(query)
                              {unquote(expr), unquote(params), unquote(Enum.reverse(acc.subqueries)), unquote(aliases)}
                            end,
                            binding: unquote(Macro.escape(binding)),
                            file: unquote(env.file),
                            line: unquote(env.line)}
  end
end
```

With the logging added, compiling the app produces the following output:

```
[1] escape_binding query: {:query, [], Ecto.Query.Builder.Dynamic}
[1] escape_binding vars: []
[2] escape expr: true
[2] escape params: []
[2] escape acc: %{aliases: %{}, subqueries: []}
[3] escape_select_aliases: {:%{}, [], []}
[4] escape_params: []
[1] escape_binding query: {:query, [], Ecto.Query.Builder.Dynamic}
[1] escape_binding vars: [p: 0]
[2] escape expr: {:{}, [],
 [
   :and,
   [],
   [
     {:{}, [], [:^, [], [0]]},
     {:{}, [],
      [
        :==,
        [],
        [
          {:{}, [],
           [{:{}, [], [:., [], [{:{}, [], [:&, [], [0]]}, :author]]}, [], []]},
          {:{}, [], [:^, [], [1]]}
        ]
      ]}
   ]
 ]}
[2] escape params: [
  {{{:., [],
     [{:__aliases__, [alias: false], [:Ecto, :Query, :Builder]}, :not_nil!]},
    [], [{:value, [line: 20], nil}]}, {0, :author}},
  {{:dynamic, [line: 20], nil}, :boolean}
]
[2] escape acc: %{aliases: %{}, subqueries: []}
[3] escape_select_aliases: {:%{}, [], []}
[4] escape_params: [
  {{:dynamic, [line: 20], nil}, :boolean},
  {{{:., [],
     [{:__aliases__, [alias: false], [:Ecto, :Query, :Builder]}, :not_nil!]},
    [], [{:value, [line: 20], nil}]}, {0, :author}}
]
[1] escape_binding query: {:query, [], Ecto.Query.Builder.Dynamic}
[1] escape_binding vars: [p: 0]
[2] escape expr: {:{}, [],
 [
   :and,
   [],
   [
     {:{}, [], [:^, [], [0]]},
     {:{}, [],
      [
        :==,
        [],
        [
          {:{}, [],
           [{:{}, [], [:., [], [{:{}, [], [:&, [], [0]]}, :category]]}, [], []]},
          {:{}, [], [:^, [], [1]]}
        ]
      ]}
   ]
 ]}
[2] escape params: [
  {{{:., [],
     [{:__aliases__, [alias: false], [:Ecto, :Query, :Builder]}, :not_nil!]},
    [], [{:value, [line: 23], nil}]}, {0, :category}},
  {{:dynamic, [line: 23], nil}, :boolean}
]
[2] escape acc: %{aliases: %{}, subqueries: []}
[3] escape_select_aliases: {:%{}, [], []}
[4] escape_params: [
  {{:dynamic, [line: 23], nil}, :boolean},
  {{{:., [],
     [{:__aliases__, [alias: false], [:Ecto, :Query, :Builder]}, :not_nil!]},
    [], [{:value, [line: 23], nil}]}, {0, :category}}
]
[1] escape_binding query: {:query, [], Ecto.Query.Builder.Dynamic}
[1] escape_binding vars: [p: 0]
[2] escape expr: {:{}, [],
 [
   :and,
   [],
   [
     {:{}, [], [:^, [], [0]]},
     {:{}, [],
      [
        :>,
        [],
        [
          {:{}, [],
           [
             {:{}, [], [:., [], [{:{}, [], [:&, [], [0]]}, :published_at]]},
             [],
             []
           ]},
          {:{}, [], [:^, [], [1]]}
        ]
      ]}
   ]
 ]}
[2] escape params: [
  {{{:., [],
     [{:__aliases__, [alias: false], [:Ecto, :Query, :Builder]}, :not_nil!]},
    [], [{:value, [line: 26], nil}]}, {0, :published_at}}
  {{:dynamic, [line: 26], nil}, :boolean}
]
[2] escape acc: %{aliases: %{}, subqueries: []}
[3] escape_select_aliases: {:%{}, [], []}
[4] escape_params: [
  {{:dynamic, [line: 26], nil}, :boolean},
  {{{:., [],
     [{:__aliases__, [alias: false], [:Ecto, :Query, :Builder]}, :not_nil!]},
    [], [{:value, [line: 26], nil}]}, {0, :published_at}}
]
```

I did not expect Ecto to compile so many ASTs based on `dynamic/2` function calls. The compiler creates an AST for every dynamic function call as seen in the "escape expr" logging. Embedded within the escaped expression output is our dynamic query:

```elixir
expr = [
  {:{}, [],
   [{:{}, [], [:., [], [{:{}, [], [:&, [], [0]]}, :author]]}, [], []]},
  {:{}, [], [:^, [], [1]]}
]

iex(16)> Macro.to_string(expr)
"[{{:., [], [{:&, [], [0]}, :author]}, [], []}, {:^, [], [1]}]"
```


Overall, the `dynamic/2` function in Ecto is a powerful tool for building flexible and dynamic queries that can adapt to a wide variety of use cases.

### Sources
- [Under the Hood of Ecto](https://blog.appsignal.com/2023/02/14/under-the-hood-of-ecto.html)
