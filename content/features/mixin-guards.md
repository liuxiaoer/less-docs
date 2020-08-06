与简单的值和数量相反，当你想要在表达式上匹配时，Guards是很有用的。如果你很熟悉函数式编程，你可以已经遇到过它们。

试图保持与定义原生CSS相近，Less选择通过**guarded mixins**代替`if`/`else`,实现有条件的执行,就像在`@media`查询特性规范


以示例开始学习:

```less
.mixin(@a) when (lightness(@a) >= 50%) {
  background-color: black;
}
.mixin(@a) when (lightness(@a) < 50%) {
  background-color: white;
}
.mixin(@a) {
  color: @a;
}
```

key是`when`的关键字，它引入了一个guard序列（这是只有一个guard）.现在如果我们运行下面的代码：

```less
.class1 { .mixin(#ddd) }
.class2 { .mixin(#555) }
```

这是我们得到的结果：

```css
.class1 {
  background-color: black;
  color: #ddd;
}
.class2 {
  background-color: white;
  color: #555;
}
```

#### Guard比较运算符

这是guard可以使用的所有比较运算符：`>`, `>=`, `=`, `=<`, `<`。另外，true关键字是唯一个使用两个mixin等价的真值。

```less
.truth(@a) when (@a) { ... }
.truth(@a) when (@a = true) { ... }
```

除了`true`任何值都是fasle:

```less
.class {
  .truth(40); // Will not match any of the above definitions.
}
```

注意：你也可以相互比较参数，也可以与非参数比较：

```less
@media: mobile;

.mixin(@a) when (@media = mobile) { ... }
.mixin(@a) when (@media = desktop) { ... }

.max(@a; @b) when (@a > @b) { width: @a }
.max(@a; @b) when (@a < @b) { width: @b }
```

#### Guard逻辑运算符

你可以使用逻辑运算符。语法基于CSS的媒体查询。

结合guard使用`and`关键字：

```less
.mixin(@a) when (isnumber(@a)) and (@a > 0) { ... }
```

你可以使用逗号`,`分隔guards来模仿*or*运算符。如果任意一个guard结果为true，则视为匹配：

```less
.mixin(@a) when (@a > 10), (@a < -10) { ... }
```

使用`not`关键字来否定条件：

```less
.mixin(@b) when not (@b > 0) { ... }
```

#### 类型检查函数

最后，如果你想通过值的类型匹配mixin，你可以使用下面的is函数：

```less
.mixin(@a; @b: 0) when (isnumber(@b)) { ... }
.mixin(@a; @b: black) when (iscolor(@b)) { ... }
```

下面是基本的类型检查函数：

* `iscolor`
* `isnumber`
* `isstring`
* `iskeyword`
* `isurl`

如果你想检查一个值除了是一个数字之外，是否还使用了特定的单位，你可以使用下面的一个：

* `ispixel`
* `ispercentage`
* `isem`
* `isunit`
