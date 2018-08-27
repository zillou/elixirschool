---
version: 1.0.0
title: Mnesia 数据库
---

Mnesia 是一个强大的分布式实时数据库管理系统。

{% include toc.html %}

## 概要

Mnesia 是 Erlang 运行时中自带的一个数据库管理系统（DBMS），也可以在 Elixir 中很自然地使用。Mnesia 的数据库模型可以混合了关系型和对象型数据模型的特征，让它可以用来开发任何规模的分布式应用程序。

## 应用场景

何时该使用何种技术常常是一个令人困惑的事情。如果下面这些问题中任意一个的答案是 yes 的话，则是一个很好的迹象告诉我们在这个情况下用 Mnesia 比用 ETS 或者 DETS 要适合。
When to use a particular piece of technology is often a confusing pursuit. If you can answer 'yes' to any of the following questions, then this is a good indication to use Mnesia over ETS or DETS.

  - 我是否需要回滚事务？
  - 我是否需要用一个简单的语法来读写数据？
  - 我是否需要在多于一个以上的节点存储数据？
  - 我是否需要选择数据存储的位置（内存还是硬盘）？

## Schema

因为 Mnesia 是 Erlang 的内容，还没有被包含到 Elixir，我们要用 `:mnesia` 这种方式去引用 Mnesia （参考[和 Erlang 互操作](../../advanced/erlang/)）。

```elixir

iex> :mnesia.create_schema([node()])

# or if you prefer the Elixir feel...

iex> alias :mnesia, as: Mnesia
iex> Mnesia.create_schema([node()])
```

在本课中，我们会使用后一种方式来使用 Mnesia 的 API。`Mnesia.create_schema/1` 会初始化一个空的 Schema 并且传递给一个节点列表。 在本例中，我们传入的是当前 IEx 会话所在的节点。

## 节点（Node）

一旦我们在 IEx 中执行了 `Mnesia.create_schema([node()])` 命令后，我们就可以在当前目录下看到一个叫 **Mnesia.nonode@nohost** 或者类似名字的文件夹。你也许会好奇到底 **nonode@nohost** 代表着什么，因为我们还没有在之前的课程中见过它。我们接下来就来看看：

```shell
$ iex --help
Usage: iex [options] [.exs file] [data]

  -v                Prints version
  -e "command"      Evaluates the given command (*)
  -r "file"         Requires the given files/patterns (*)
  -S "script"       Finds and executes the given script
  -pr "file"        Requires the given files/patterns in parallel (*)
  -pa "path"        Prepends the given path to Erlang code path (*)
  -pz "path"        Appends the given path to Erlang code path (*)
  --app "app"       Start the given app and its dependencies (*)
  --erl "switches"  Switches to be passed down to Erlang (*)
  --name "name"     Makes and assigns a name to the distributed node
  --sname "name"    Makes and assigns a short name to the distributed node
  --cookie "cookie" Sets a cookie for this distributed node
  --hidden          Makes a hidden node
  --werl            Uses Erlang's Windows shell GUI (Windows only)
  --detached        Starts the Erlang VM detached from console
  --remsh "name"    Connects to a node using a remote shell
  --dot-iex "path"  Overrides default .iex.exs file and uses path instead;
                    path can be empty, then no file will be loaded

** Options marked with (*) can be given more than once
** Options given after the .exs file or -- are passed down to the executed code
** Options can be passed to the VM using ELIXIR_ERL_OPTIONS or --erl
```

当你给 IEx 传递 `--help` 选项的时候，IEx 会列出所有可用的选项。我们可以看到有 `--name` 和 `--sname` 两个选项可以给节点起名。
一个节点（Node）就是一个运行中的 Erlang 虚拟机，它独自管理着自己的通讯，垃圾回收，进程调度以及内存等等。如果你没有给节点起名，那么这个节点的名字就叫 **nonode@nohost** 。

```shell
$ iex --name learner@elixirschool.com

Erlang/OTP {{ site.erlang.OTP }} [erts-{{ site.erlang.erts }}] [source] [64-bit] [smp:4:4] [async-threads:10] [hipe] [kernel-poll:false] [dtrace]

Interactive Elixir ({{ site.elixir.version }}) - press Ctrl+C to exit (type h() ENTER for help)
iex(learner@elixirschool.com)> Node.self
:"learner@elixirschool.com"
```

我们可以看到，当我给节点起名后，我们当前的节点名字已经叫做 `:"learner@elixirschool.com"`。如果我们再运行 `Mnesia.create_schema([node()])` 的话，我们会看到另外一个叫做 **Mnesia.learner@elixirschool.com** 的文件夹。这样设计的目的很简单。Erlang 中的节点只是用来连接其他节点用以分享（分发）信息和资源，它们并不一定要在同一台机器上，也可以通过局域网或者互联网等方式通讯。

## Starting Mnesia

Now we have the background basics out of the way and set up the database, we are now in a position to start the Mnesia DBMS with the `Mnesia.start/0` command.

```elixir
iex> alias :mnesia, as: Mnesia
iex> Mnesia.create_schema([node()])
:ok
iex> Mnesia.start()
:ok
```

It is worth keeping in mind when running a distributed system with two or more participating nodes, the function `Mnesia.start/1` must be executed on all participating nodes.

## Creating Tables

The function `Mnesia.create_table/2` is used to create tables within our database. Below we create a table called `Person` and then pass a keyword list defining the table schema.

```elixir
iex> Mnesia.create_table(Person, [attributes: [:id, :name, :job]])
{:atomic, :ok}
```

We define the columns using the atoms `:id`, `:name`, and `:job`. When we execute `Mnesia.create_table/2`, it will return either one of the following responses:

 - `{:atomic, :ok}` if the function executes successfully
 - `{:aborted, Reason}` if the function failed

In particular, if the table already exists, the reason will be of the form `{:already_exists, table}` so if we try to create this table a second time, we will get:

```elixir
iex> Mnesia.create_table(Person, [attributes: [:id, :name, :job]])
{:aborted, {:already_exists, Person}}
```

## 脏操作

First of all we will look at the dirty way of reading and writing to a Mnesia table. This should generally be avoided as success is not guaranteed, but it should help us learn and become comfortable working with Mnesia. Let's add some entries to our **Person** table.
首先我们来学习对 Mnesia 表的脏炒作方式。一般情况下，我们都不会使用脏操作，因为脏操作并不一定保证成功，但是它可以帮助我们学习和适应 Mnesia 的使用方式。下面让我们往 **Person** 表中添加一些记录。

```elixir
iex> Mnesia.dirty_write({Person, 1, "Seymour Skinner", "Principal"})
:ok

iex> Mnesia.dirty_write({Person, 2, "Homer Simpson", "Safety Inspector"})
:ok

iex> Mnesia.dirty_write({Person, 3, "Moe Szyslak", "Bartender"})
:ok
```

...然后我们可以通过 `Mnesia.dirty_read/1` 来读取数据：

```elixir
iex> Mnesia.dirty_read({Person, 1})
[{Person, 1, "Seymour Skinner", "Principal"}]

iex> Mnesia.dirty_read({Person, 2})
[{Person, 2, "Homer Simpson", "Safety Inspector"}]

iex> Mnesia.dirty_read({Person, 3})
[{Person, 3, "Moe Szyslak", "Bartender"}]

iex> Mnesia.dirty_read({Person, 4})
[]
```

如果我们查询的记录不存在时，Mnesia 会返回一个空的列表。

## 事务（Transaction）

我们一般会把我们对数据库的读写包在一个数据库事务里面。对事务的支持对设计容错系统和分布式系统非常重要。Mnesia 的事务是通过对数据库的多个操作包含到一个函数体中来实现。首先我们创建一个匿名函数，如此例中的 `data_to_write`，然后把这个函数传给 `Mnesia.transaction`。

```elixir
iex> data_to_write = fn ->
...>   Mnesia.write({Person, 4, "Marge Simpson", "home maker"})
...>   Mnesia.write({Person, 5, "Hans Moleman", "unknown"})
...>   Mnesia.write({Person, 6, "Monty Burns", "Businessman"})
...>   Mnesia.write({Person, 7, "Waylon Smithers", "Executive assistant"})
...> end
#Function<20.54118792/0 in :erl_eval.expr/5>

iex> Mnesia.transaction(data_to_write)
{:atomic, :ok}
```

从 IEx 中打印的消息看来，我们可以安全地假设数据已经被成功地写进了 `Person` 表。我们来验证一下使用事务从数据库里面读出刚刚写入的数据。我们可以用 `Mnesia.read/1` 来从数据库里面读取数据，同样的，我们也需要使用一个匿名函数。

```elixir
iex> data_to_read = fn ->
...>   Mnesia.read({Person, 6})
...> end
#Function<20.54118792/0 in :erl_eval.expr/5>

iex> Mnesia.transaction(data_to_read)
{:atomic, [{Person, 6, "Monty Burns", "Businessman"}]}
```

Note that if you want to update data, you just need to call `Mnesia.write/1` with the same key as an existing record. Therefore, to update the record for Hans, you can do:

```elixir
iex> Mnesia.transaction(
...>   fn ->
...>     Mnesia.write({Person, 5, "Hans Moleman", "Ex-Mayor"})
...>   end
...> )
```

## 使用索引

Mnesia 支持在非主键字段上添加索引
所以我们可以在 `Person` 表的 `:job`
Mnesia support indices on non-key columns and data can then be queried against those indices. So we can add an index against the `:job` column of the `Person` table:

```elixir
iex> Mnesia.add_table_index(Person, :job)
{:atomic, :ok}
```

The result is similar to the one returned by `Mnesia.create_table/2`:

 - `{:atomic, :ok}` if the function executes successfully
 - `{:aborted, Reason}` if the function failed

In particular, if the index already exists, the reason will be of the form `{:already_exists, table, attribute_index}` so if we try to add this index a second time, we will get:

```elixir
iex> Mnesia.add_table_index(Person, :job)
{:aborted, {:already_exists, Person, 4}}
```

Once the index is successfully created, we can read against it and retrieve a list of all principals:

```elixir
iex> Mnesia.transaction(
...>   fn ->
...>     Mnesia.index_read(Person, "Principal", :job)
...>   end
...> )
{:atomic, [{Person, 1, "Seymour Skinner", "Principal"}]}
```

## Match and select

Mnesia supports complex queries to retrieve data from a table in the form of matching and ad-hoc select functions.

The `Mnesia.match_object/1` function returns all records that match the given pattern. If any of the columns in the table have indices, it can make use of them to make the query more efficient. Use the special atom `:_` to identify columns that don't participate in the match.

```elixir
iex> Mnesia.transaction(
...>   fn ->
...>     Mnesia.match_object({Person, :_, "Marge Simpson", :_})
...>   end
...> )
{:atomic, [{Person, 4, "Marge Simpson", "home maker"}]}
```

The `Mnesia.select/2` function allows you to specify a custom query using any operator or function in the Elixir language (or Erlang for that matter). Let's look at an example to select all records that have a key that is greater than 3:

```elixir
iex> Mnesia.transaction(
...>   fn ->
...>     {% raw %}Mnesia.select(Person, [{{Person, :"$1", :"$2", :"$3"}, [{:>, :"$1", 3}], [:"$$"]}]){% endraw %}
...>   end
...> )
{:atomic, [[7, "Waylon Smithers", "Executive assistant"], [4, "Marge Simpson", "home maker"], [6, "Monty Burns", "Businessman"], [5, "Hans Moleman", "unknown"]]}
```

Let's unpack this. The first attribute is the table, `Person`, the second attribute is a triple of the form `{match, [guard], [result]}`:

- `match` is the same as what you'd pass to the `Mnesia.match_object/1` function; however, note the special atoms `:"$n"` that specify positional parameters that are used by the remainder of the query
- the `guard` list is a list of tuples that specifies what guard functions to apply, in this case the `:>` (greater than) built in function with the first positional parameter `:"$1"` and the constant `3` as attributes
- the `result` list is the list of fields that are returned by the query, in the form of positional parameters of the special atom `:"$$"` to reference all fields so you could use `[:"$1", :"$2"]` to return the first two fields or `[:"$$"]` to return all fields

For more details, see [the Erlang Mnesia documentation for select/2](http://erlang.org/doc/man/mnesia.html#select-2).

## Data initialization and migration

With every software solution, there will come a time when you need to upgrade the software and migrate the data stored in your database. For example, we may want to add an `:age` column to our `Person` table in v2 of our app. We can't create the `Person` table once it's been created but we can transform it. For this we need to know when to transform, which we can do when creating the table. To do this, we can use the `Mnesia.table_info/2` function to retrieve the current structure of the table and the `Mnesia.transform_table/3` function to transform it to the new structure.

The code below does this by implementing the following logic:

* Create the table with the v2 attributes: `[:id, :name, :job, :age]`
* Handle the creation result:
    * `{:atomic, :ok}`: initialize the table by creating indices on `:job` and `:age`
    * `{:aborted, {:already_exists, Person}}`: check what the attributes are in the current table and act accordingly:
        * if it's the v1 list (`[:id, :name, :job]`), transform the table giving everybody an age of 21 and add a new index on `:age`
        * if it's the v2 list, do nothing, we're good
        * if it's something else, bail out

The `Mnesia.transform_table/3` function takes as attributes the name of the table, a function that transforms a record from the old to the new format and the list of new attributes.

```elixir
iex> case Mnesia.create_table(Person, [attributes: [:id, :name, :job, :age]]) do
...>   {:atomic, :ok} ->
...>     Mnesia.add_table_index(Person, :job)
...>     Mnesia.add_table_index(Person, :age)
...>   {:aborted, {:already_exists, Person}} ->
...>     case Mnesia.table_info(Person, :attributes) do
...>       [:id, :name, :job] ->
...>         Mnesia.transform_table(
...>           Person,
...>           fn ({Person, id, name, job}) ->
...>             {Person, id, name, job, 21}
...>           end,
...>           [:id, :name, :job, :age]
...>           )
...>         Mnesia.add_table_index(Person, :age)
...>       [:id, :name, :job, :age] ->
...>         :ok
...>       other ->
...>         {:error, other}
...>     end
...> end
```
