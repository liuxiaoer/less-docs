> Extend是一个Less伪类，它将放入的选择器与匹配它引用的选择器合并。

发布版本 [v1.4.0]({{ less.master.url }}CHANGELOG.md)

```less
nav ul {
  &:extend(.inline);
  background: blue;
}
```

在上面的规则集合中，`:extend`选择器将"扩展选择器"(`nav ul`)应用到`.inline`类出现的任何地方。声明块将保持原样，但没有任何对textend的引用(因为extend不是css)。

示例如下:

```less
nav ul { //声明块
  &:extend(.inline);
  background: blue;
}
.inline {
  color: red;
}
```

结果

```css
nav ul {
  background: blue;
}
.inline,
nav ul {
  color: red;
}
```

注意：`nav ul:extend(.inline)`选择器是怎样输出`nav ul`- extend在输出之前被移除并保持选择器块不变。如果选择器块中没有属性，选择器会被删除(extend仍可以影响其它选择器)

### Extend 语法
extend两种用法：1.附加在某个选择器 2.放在某个规则集合中。
它像一个选择器的伪类，选择器参数后面可选的关键字`all`:

示例:

```less
.a:extend(.b) {}

// 上面的选择器块和下面的选择器块功能相同
.a {
  &:extend(.b);
}
```

```less
.c:extend(.d all) {
  // 扩展所有匹配（".d" e.g. ".x.d" or ".d.x"）的规则 
}
.c:extend(.d) {
  // 仅仅扩展匹配".d"的规则
}
```

它能包含一个或多个类要扩展的类，用逗号分隔。

示例:

```less
.e:extend(.f) {}
.e:extend(.g) {}

// 上面代码和下面代码功能相同
.e:extend(.f, .g) {}
```

### Extend Attached to Selector

延伸到选择器看起来像一个普通的伪类，选择器做为参数。一个选择器可以包含多个extend子句，但是所有extend必须放在选择器的末尾。

* Extend 在选择器末尾: `pre:hover:extend(div pre)`.
* 允许选择器与extend中间有空格: `pre:hover :extend(div pre)`.
* 允许多个extend: `pre:hover:extend(div pre):extend(.bucket tr)` - 与后面功能相同 `pre:hover:extend(div pre, .bucket tr)`
* 不允许这样写法: `pre:hover:extend(div pre).nth-child(odd)`. Extend必须放在选择器末尾.


如果一个规则集合包含多个选择器，每一个选择器都可以跟extend关键字。示例：

```less
.big-division,
.big-bag:extend(.bag),
.big-bucket:extend(.bucket) {
  // body
}
```

### 在规则集中使用extend
使用`&:extend(selector)`语法extend可以放在规则集中使用。把extend放到规则集是将其放置到该规则集中的每个选择器中的快捷方式。

规则集中使用Extend:

```less
pre:hover,
.some-class {
  &:extend(div pre);
}
```

与在每个选择器后面加上extend相同的功能：

```less
pre:hover:extend(div pre),
.some-class:extend(div pre) {}
```

### 扩展嵌套选择器

Extend可以扩展嵌套选择器。

示例:

```less
.bucket {
  tr { // 带有目标选择器的嵌套规则集
    color: blue;
  }
}
.some-class:extend(.bucket tr) {} // 嵌套规则集被识别
```
Outputs

```css
.bucket tr,
.some-class {
  color: blue;
}
```

实际上extend查找的是编译后的css，而不是原来的less

示例:

```less
.bucket {
  tr & { // 带有目标选择器的嵌套规则集
    color: blue;
  }
}
.some-class:extend(tr .bucket) {} // 嵌套规则集被识别
```

结果

```css
tr .bucket,
.some-class {
  color: blue;
}
```


### Extend精确匹配
Extend默认在选择器中间查找精确的匹配。选择器是否使用头部星号(leading star)很重要。两个nth表达式有相同含义很重要，他们必须且有相同的形式才能匹配。唯一例外的时属性选择器中的引号，less知道他们有相同的含义并匹配他们。

示例:

```less
.a.class,
.class.a,
.class > .a {
  color: blue;
}
.test:extend(.class) {} // 无法匹配上面的选择
```

头部星号(leading star)很重要. 选择器 `*.class` 和 `.class` 是相等的, 但是extend不会匹配他们:

```less
*.class {
  color: blue;
}
.noStar:extend(.class) {} // 无法匹配上面的选择
```

结果

```css
*.class {
  color: blue;
}
```

伪类的顺序很重要。选择器`link:hover:visited`和`link:visited:hover`匹配相同的元素，但extend认为他们是不同的：

```less
link:hover:visited {
  color: blue;
}
.selector:extend(link:visited:hover) {}
```
结果

```css
link:hover:visited {
  color: blue;
}
```

### nth表达式

Nth表达式形式很重要。表达式`1n+3`和`n+3`是相同的，但是extend不会匹配他们:

```less
:nth-child(1n+3) {
  color: blue;
}
.child:extend(:nth-child(n+3)) {}
```
结果

```css
:nth-child(1n+3) {
  color: blue;
}
```

属性选择器中的引号类型不重要。下面所有的写法都是相同的。

```less
[title=identifier] {
  color: blue;
}
[title='identifier'] {
  color: blue;
}
[title="identifier"] {
  color: blue;
}

.noQuote:extend([title=identifier]) {}
.singleQuote:extend([title='identifier']) {}
.doubleQuote:extend([title="identifier"]) {}
```
结果

```css
[title=identifier],
.noQuote,
.singleQuote,
.doubleQuote {
  color: blue;
}

[title='identifier'],
.noQuote,
.singleQuote,
.doubleQuote {
  color: blue;
}

[title="identifier"],
.noQuote,
.singleQuote,
.doubleQuote {
  color: blue;
}
```

### Extend "all"
当时在extend最后参数指定all关键字，就是告诉Less匹配选择器做为另外选择器的一部分。选择器将被复制，选择器的匹配部分将被extend替换，从而生成一个新的选择器。

示例:

```less
.a.b.test,
.test.c {
  color: orange;
}
.test {
  &:hover {
    color: green;
  }
}

.replacement:extend(.test all) {}
```

结果

```css
.a.b.test,
.test.c,
.a.b.replacement,
.replacement.c {
  color: orange;
}
.test:hover,
.replacement:hover {
  color: green;
}
```

_You can think of this mode of operation as essentially doing a non-destructive search and replace._
_你可以将这种操作模式视为是进行非破坏性的查找和替换_


### Selector Interpolation with Extend
> Extend**不能**匹配带有变量的选择器。extend将忽视带有变量的选择器。

However, extend can be attached to interpolated selector.
然而，extend可以附加到带有变量的选择器。

带有变量的选择器将不会匹配:

```less
@variable: .bucket;
@{variable} { // 带有变量的选择器
  color: blue;
}
.some-class:extend(.bucket) {} //不能匹配
```

在目标选择器extend变量无法匹配。

```less
.bucket {
  color: blue;
}
.some-class:extend(@{variable}) {} // 带有变量的选择器无法匹配
@variable: .bucket;
```

上面两个示例结果都如下:

```less
.bucket {
  color: blue;
}
```

然而, `:extend`附加在带有变量的选择器可以正常运行。

```less
.bucket {
  color: blue;
}
@{variable}:extend(.bucket) {}
@variable: .selector;
```

结果:

```less
.bucket, .selector {
  color: blue;
}
```

### 域 /在@media使用Extend

现在，`:extend`在`@media`中定义只能匹配相同媒体定义中的选择器。

```less
@media print {
  .screenClass:extend(.selector) {} // 在@media使用Extend
  .selector { // 可以匹配 - 他们在相同的媒体中
    color: black;
  }
}
.selector { // 规则集在顶级样式表中 - extend忽略
  color: red;
}
@media screen {
  .selector {  // 规则集在其它媒体中 - extend忽略
    color: blue;
  }
}
```

编译成:

```css
@media print {
  .selector,
  .screenClass { /*  规则集在相同媒体中，被扩展 */
    color: black;
  }
}
.selector { /* 规则集在顶级样式表中被忽略 */
  color: red;
}
@media screen {
  .selector { /* 规则集在其它媒体中被忽略 */
    color: blue;
  }
}
```

Note: extending does not match selectors inside a nested `@media` declaration:
注意：extend不会匹配嵌套`@media`中的定义：

```less
@media screen {
  .screenClass:extend(.selector) {} // extend 在 media中
  @media (min-width: 1023px) {
    .selector {  // 规则集在嵌套media中 - extend忽略
      color: blue;
    }
  }
}
```

编译成:

```css
@media screen and (min-width: 1023px) {
  .selector { /* 规则集在另外一个嵌套media中被忽略 */
    color: blue;
  }
}
```

顶级extend匹配所有内容，包括嵌套media中的所有选择器：

```less
@media screen {
  .selector {  /* 规则集在嵌套media中 - 顶级extend正常运行 */
    color: blue;
  }
  @media (min-width: 1023px) {
    .selector {  /* 规则集在嵌套media中 - 顶级extend正常运行 */
      color: blue;
    }
  }
}

.topLevel:extend(.selector) {} /* 顶级entend匹配所有内容 */
```

编译成:

```css
@media screen {
  .selector,
  .topLevel { /* 规则集在media中被扩展 */
    color: blue;
  }
}
@media screen and (min-width: 1023px) {
  .selector,
  .topLevel { /* 规则集在嵌套media中被扩展 */
    color: blue;
  }
}
```

### 重复检测

目前没有重复检测.

Example:

```less
.alert-info,
.widget {
  /* declarations */
}

.alert:extend(.alert-info, .widget) {}
```
结果

```css
.alert-info,
.widget,
.alert,
.alert {
  /* declarations */
}
```

### Extend用例

#### 经典用例

The classic use case is to avoid adding a base class. For example, if you have
经典用例就是避免添加一个基类。例如，如果你有以下代码

```css
.animal {
  background-color: black;
  color: white;
}
```

你想定义一个覆盖背景颜色的animal的子类型，然而你有两个选择，第一个选择修改HTML

```html
<a class="animal bear">Bear</a>
```
```css
.animal {
  background-color: black;
  color: white;
}
.bear {
  background-color: brown;
}
```

或者简化html并且使用在Less中使用extend。例如

```html
<a class="bear">Bear</a>
```

```less
.animal {
  background-color: black;
  color: white;
}
.bear {
  &:extend(.animal);
  background-color: brown;
}
```

#### 减少CSS大小
Mixins复制所有的属性到一个选择器中，这样会导致不必要的重复。因此你可以使用extends而不是mixins将选择器上移到你希望使用的属性上，这样会导致生成更少的CSS。

示例 - 使用mixin:

```less
.my-inline-block() {
  display: inline-block;
  font-size: 0;
}
.thing1 {
  .my-inline-block;
}
.thing2 {
  .my-inline-block;
}
```

结果

```css
.thing1 {
  display: inline-block;
  font-size: 0;
}
.thing2 {
  display: inline-block;
  font-size: 0;
}
```

示例 (使用 extends):

```less
.my-inline-block {
  display: inline-block;
  font-size: 0;
}
.thing1 {
  &:extend(.my-inline-block);
}
.thing2 {
  &:extend(.my-inline-block);
}
```

结果

```css
.my-inline-block,
.thing1,
.thing2 {
  display: inline-block;
  font-size: 0;
}
```

#### 组合样式 / 更高级的Mixin

另一个用例是mixin的替代方案 - 因为mixins只能用于简单的选择器，如果你有两个不同的html块,但是需要使用相同的样式，可以使用extends来关联两个区域。

示例:

```less
li.list > a {
  // 样式表
}
button.list-style {
  &:extend(li.list > a); // 使用相同的样式表
}
```
