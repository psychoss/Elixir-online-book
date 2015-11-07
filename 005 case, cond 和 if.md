# case, cond 和 if

在这一章里，我们学习` case`，` cond` 和 `if` 这些流控制结构。

## `case`

`case` 允许我们使用不同的匹配模式来匹配一个值：

    iex> case {1, 2, 3} do
    ...>   {4, 5, 6} ->
    ...>     "This clause won't match"
    ...>   {1, x, 3} ->
    ...>     "This clause will match and bind x to 2 in this clause"
    ...>   _ ->
    ...>     "This clause would match any value"
    ...> end
    "This clause will match and bind x to 2 in this clause"

如果我们想在匹配模式中使用存在的变量，你需要使用 `^` 操作符：

    iex> x = 1
    1
    iex> case 10 do
    ...>   ^x -> "Won't match"
    ...>   _  -> "Will match"
    ...> end
    "Will match"
语句也允许使用额外的指定条件：

    iex> case {1, 2, 3} do
    ...>   {1, x, 3} when x > 0 ->
    ...>     "Will match"
    ...>   _ ->
    ...>     "Would match, if guard condition were not satisfied"
    ...> end
    "Will match"
上面的第一条语句将仅在 x 是正值时匹配。

## 卫语句中表达式

Elixir 导入和允许下面这些表达默认在安全器中执行：

- 比较操作符 `(==, !=, ===, !==, >, >=, <, <=)`
- 布尔操作符 `(and, or, not)`
- 数学操作符 `(+, -, *, /)`
- 数学计数符 `(+, -)`
- 二进制连接符 `<>`
- 当右边是是一个范围和列表时的 `in` 操作符
- 所有下面这些类型检测函数：

    - `is_atom/1`
    - `is_binary/1`
    - `is_bitstring/1`
    - `is_boolean/1`
    - `is_float/1`
    - `is_function/1`
    - `is_function/2`
    - `is_integer/1`
    - `is_list/1`
    - `is_map/1`
    - `is_nil/1`
    - `is_number/1`
    - `is_pid/1`
    - `is_port/1`
    - `is_reference/1`
    - `is_tuple/1`
- 加上这些函数：

    - `abs(number)`
    - `binary_part(binary, start, length)`
    - `bit_size(bitstring)`
    - `byte_size(bitstring)`
    - `div(integer, integer)`
    - `elem(tuple, n)`
    - `hd(list)`
    - `length(list)`
    - `map_size(map)`
    - `node()`
    - `node(pid | ref | port)`
    - `rem(integer, integer)`
    - `round(number)`
    - `self()`
    - `tl(list)`
    - `trunc(number)`
    - `tuple_size(tuple)`

再者，用户或许会定义自己的安全器。例如移位模块定义了一些当做函数和操作的安全器：`bnot`，` ~~~`，` band`，` &&&`，` bor`，` |||`，` bxor`，<code>&#94;&#94;&#94;</code>，` bsl`，` <<<`，` bsr`，` >>>`。

注意在安全器中，不允许 `and`，`or`，`not`这些布尔操作符，但这种限制不包括更普通的短路操作符`&&`。`, || `，`!`。

请记住，在安全器里的错误不会泄露，只是这个安全器会失败：

    iex> hd(1)
    ** (ArgumentError) argument error
        :erlang.hd(1)
    iex> case 1 do
    ...>   x when hd(x) -> "Won't match"
    ...>   x -> "Got: #{x}"
    ...> end
    "Got 1"
如果没有语句匹配，会抛出一个错误：

    iex> case :ok do
    ...>   :error -> "Won't match"
    ...> end
    ** (CaseClauseError) no case clause matching: :ok
注意匿名函数同样可以拥有多个语句和安全器：

    iex> f = fn
    ...>   x, y when x > 0 -> x + y
    ...>   x, y -> x * y
    ...> end
    #Function<12.71889879/2 in :erl_eval.expr/5>
    iex> f.(1, 3)
    4
    iex> f.(-1, 3)
    -3
在匿名函数中的语句的参数必须相等，不然会抛出错误。
## cond

`case` 在需要使用不同的匹配时很实用。但是，在很多状况里，我们想测试不同的状况，然后找到第一个测试为 true 的匹配。在这样的例子里，我们可以使用 `cond`：

    iex> cond do
    ...>   2 + 2 == 5 ->
    ...>     "This will not be true"
    ...>   2 * 2 == 3 ->
    ...>     "Nor this"
    ...>   1 + 1 == 2 ->
    ...>     "But this will"
    ...> end
    "But this will"
它等同于在大多数语言中的 `else if` 语句 （虽然在这里使用的比较少）。

如果没有 true 返回，将抛出一个错误。因为这个理由，这可能需要加上最后一个条件，等于 true,它将总是被匹配：

    iex> cond do
    ...>   2 + 2 == 5 ->
    ...>     "This is never true"
    ...>   2 * 2 == 3 ->
    ...>     "Nor this"
    ...>   true ->
    ...>     "This is always true (equivalent to else)"
    ...> end
    "This is always true (equivalent to else)"
最后，注意 `cond`把除 nil 和 fasle 的所有值都看做 true：

    iex> cond do
    ...>   hd([1,2,3]) ->
    ...>     "1 is considered as true"
    ...> end
    "1 is considered as true"
## if 和 unless

除了 case 和 cond， Elixir 同时也提供在测试一个状况时很有用的两个宏` if/2` 和 `unless/2`：

    iex> if true do
    ...>   "This works!"
    ...> end
    "This works!"
    iex> unless true do
    ...>   "This will never be seen"
    ...> end
    nil
如果 `if/2` 返回 fasle 或者 nil，在 `do/end` 之间的代码不会执行，它指示简单的返回一个 nil。与此相对的是 `unless/2`。

它们都提供 `else` 区块：

    iex> if nil do
    ...>   "This won't be seen"
    ...> else
    ...>   "This will"
    ...> end
    "This will"

现在，我们已经学习了 4 种控制结构 `case`，` cond`，` if `和` unless`，它们都封装在 `do/end` 块里。我们也可以像下面一样编写 `if`：

    iex> if true, do: 1 + 2
    3
在  Elixir 里，`do/end` 区块是向 `do:` 传入一组表达式的便捷方式。它等同于：    

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
我们把第二种方法叫做使用了关键字列表。我们可以用下面的语法插入 `else`：

    iex> if false, do: :this, else: :that
    :that
一个需要记住的事情是，`do/end` 区块总是绑定最远可达的函数调用。比如说下面的例子

    iex> is_number if true do
    ...>  1 + 2
    ...> end
    ** (RuntimeError) undefined function: if/1
会被解释成：

    iex> is_number(if true) do
    ...>  1 + 2
    ...> end
    ** (RuntimeError) undefined function: if/1
因为  Elixir 尝试调用 `if/1`，这会导致一个会定义的函数错误。添加明确的括号能解决这个不明确的问题：

    iex> is_number(if true do
    ...>  1 + 2
    ...> end)
    true
关键字列表在这门语言中扮演了很重要的角色。在大多数函数和宏里都非常常见。我们将在后面的章节里更多的解释它们。