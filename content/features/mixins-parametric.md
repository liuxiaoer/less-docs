> 如何将参数传递给mixins

Mixins也可以带有参数，参数是在混合时传递给选择器块的变量。

例如:

```less
.border-radius(@radius) {
  -webkit-border-radius: @radius;
     -moz-border-radius: @radius;
          border-radius: @radius;
}
```

这里是我们如何将它混合到各种规则集中：

```less
#header {
  .border-radius(4px);
}
.button {
  .border-radius(6px);
}
```

参数化的mixins的参数也可以有默认值:

```less
.border-radius(@radius: 5px) {
  -webkit-border-radius: @radius;
     -moz-border-radius: @radius;
          border-radius: @radius;
}
```

我们可以直接调用它:

```less
#header {
  .border-radius();
}
```

它包含了一个5px的圆角。

你也可以使用不带参数的参数化mixins。如果你想在CSS中隐藏规则集，但又想在其它规则集中使用它的属性，这是很有用的

```less
.wrap() {
  text-wrap: wrap;
  white-space: -moz-pre-wrap;
  white-space: pre-wrap;
  word-wrap: break-word;
}

pre { .wrap() }
```

输出:

```css
pre {
  text-wrap: wrap;
  white-space: -moz-pre-wrap;
  white-space: pre-wrap;
  word-wrap: break-word;
}
```

#### 多个参数的Mixins
多个参数用分号或逗号分隔。推荐使用分号。逗号有两层意思：它可以被理解为mixin的参数分隔符和css列表的分隔符。

使用逗号做为mixin的参数分隔符无法创建逗号分隔的列表做为参数。另一方面，如果编译器在mixin调用或声明中看到至少一个分号，它就认为参数是分号分隔的，所有逗号都是css列表的:

* 两个参数并且都包含逗号分隔的列表：`.name(1, 2, 3; something, else)`,
* 三个参数并且都包含一个数字：`.name(1, 2, 3)`,
* 使用伪分号创建mixin只一个包含逗号分割的css列表调用：`.name(1, 2, 3;)`,
* 逗号分割默认值： `.name(@param1: red, blue;)`.

定义多个相同参数名和参数个数的mixins是合法的。Less会使用所有它可以会用的参数。如果你使用一个参数的mixin，例如`.mixin(green);`，然后将使用只包含一个强制参数的所有mixin的属性。

```less
.mixin(@color) {
  color-1: @color;
}
.mixin(@color; @padding: 2) {
  color-2: @color;
  padding-2: @padding;
}
.mixin(@color; @padding; @margin: 2) {
  color-3: @color;
  padding-3: @padding;
  margin: @margin @margin @margin @margin;
}
.some .selector div {
  .mixin(#008000);
}
```

编译成:

```css
.some .selector div {
  color-1: #008000;
  color-2: #008000;
  padding-2: 2;
}
```

#### 命名的参数

mixin引用可以通过参数的名称而不仅仅是位置为提供参数值。所有参数都能被他们自己的俩字所引用，而不需要特定的顺序：

```less
.mixin(@color: black; @margin: 10px; @padding: 20px) {
  color: @color;
  margin: @margin;
  padding: @padding;
}
.class1 {
  .mixin(@margin: 20px; @color: #33acfe);
}
.class2 {
  .mixin(#efca44; @padding: 40px);
}
```
编译成:

```css
.class1 {
  color: #33acfe;
  margin: 20px;
  padding: 20px;
}
.class2 {
  color: #efca44;
  margin: 10px;
  padding: 40px;
}
```

#### `@arguments`变量

在mixins中`@arguments`有个特殊的含意。当mixin被调用时，它包含所有被传递的参数。如果你不想处理个别的参数这是很有用的。

```less
.box-shadow(@x: 0; @y: 0; @blur: 1px; @color: #000) {
  -webkit-box-shadow: @arguments;
     -moz-box-shadow: @arguments;
          box-shadow: @arguments;
}
.big-block {
  .box-shadow(2px; 5px);
}
```

结果:

```css
.big-block {
  -webkit-box-shadow: 2px 5px 1px #000;
     -moz-box-shadow: 2px 5px 1px #000;
          box-shadow: 2px 5px 1px #000;
}
```

#### 高级参数和变量`@rest`

你可以使用`...`，如果你希望你的mixin可以接收可变参数个数。把它放在一个变量名字后面，这个变量将接收这些参数。

```less
.mixin(...) {        // matches 0-N arguments
.mixin() {           // matches exactly 0 arguments
.mixin(@a: 1) {      // matches 0-1 arguments
.mixin(@a: 1; ...) { // matches 0-N arguments
.mixin(@a; ...) {    // matches 1-N arguments
```

此外:

```less
.mixin(@a; @rest...) {
   // @rest接收@a后面的所有参数
   // @arguments接收所有参数
}
```

## 模式匹配

有时，你可能想基于传递的参数改变mixin的行为。

```less
.mixin(@s; @color) { ... }

.class {
  .mixin(@switch; #888);
}
```

现在我们想让`.mixin`根据`@switch`的值具有不同的处理方式。我们定义如下`.mixin`:

```less
.mixin(dark; @color) {
  color: darken(@color, 10%);
}
.mixin(light; @color) {
  color: lighten(@color, 10%);
}
.mixin(@_; @color) {
  display: block;
}
```

我们来运行一下:

```less
@switch: light;

.class {
  .mixin(@switch; #888);
}
```

结果:

```css
.class {
  color: #a2a2a2;
  display: block;
}
```

传给`.mixin`的颜色是亮色。如果`@switch`的值是`dark`，结果会是一个深的颜色。

匹配过程:

* 第一个mixin定义未匹配，因为它的第一个参数是`dark`
* 第二个mixin定义匹配，因为它的参数期望值是`light`
* 第三个mixin定义匹配，因为它的参数期望值是任意值

所有匹配的mixin定义会被调用。变量匹配并且绑定到任意值。其它的值只会匹配与自己相等的值。

我们也可以在数量上匹配。示例如下：

```less
.mixin(@a) {
  color: @a;
}
.mixin(@a; @b) {
  color: fade(@a; @b);
}
```

如果我们调用`.mixin`传递一个参数，将会调用第一个定义，
如果我们调用`.mixin`传递两个参数，将会调用第二个定义。

