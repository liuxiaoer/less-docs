> 从其它样式表导入样式

在标准CSS中，`@import`在规则中必须在其它类型规则前面。但是Less不关心你把`@import`放哪。

示例:

```less
.foo {
  background: #900;
}
@import "this-is-valid.less";
```

## 文件扩展
`@import`语句可能会根据文件扩展名的不同而有所不同：


* 如果文件是`.css`被认为是CSS，`@import`语句如下所示.(见[内联选项](#import-options-inline))
* 如果有其它扩展名将认为是less并导入。
* 如果没有扩展名，将被加上`.less`文件扩展名。并且作为less文件导入。

示例:

```less
@import "foo";      // foo.less被导入
@import "foo.less"; // foo.less被导入
@import "foo.php";  // foo.php作为Less文件被导入
@import "foo.css";  // 声明留在原处
```

下面的选项常可以重写这个行为。

## 导入选项
> Less为CSS的`@import`提供了多个扩展，以便在处理外部文件时提供更大的灵活性。

语法: `@import (keyword) "filename";`

下面是已经实现的导入选项:

* `reference`: 使用一个Less文件，但是不输出
* `inline`: 输出源文件，但是不做处理
* `less`: 做为Less文件处理，不管文件扩展名是什么
* `css`: 做为css文件处理，不管文件扩展名是什么
* `once`: 只包含文件一次 (默认处理选项)
* `multiple`: 包含文件多次
* `optional`: 在编译文件时找不到继续

> 每个`@import`可以跟多个选项关键字，使用逗号分隔这些关键字：

示例: `@import (optional, reference) "foo.less";`

### 引用
使用 `@import (reference)` 导入其它文件，除了引用的样式，不添加导入的其它样式到输出文件中。

发布版本 [v1.5.0]({{ less.master.url }}CHANGELOG.md)

示例: `@import (reference) "foo.less";`

假设在导入的文件中，`reference`、给每一个规则和选择器标记一个_reference flag_，正常导入，但是生成CSS时，被标记引用的选择器(以及媒体查询只包含“引用”选择器)不会输出。被标记引用的样式不会出现在生成的CSS中，除非使用[mixins](#mixins-feature) 或 [extended](#extend-feature)引用样式

Additionally, **`reference`** produces different results depending on which method was used (mixin or extend):
另外，**`reference`**如何处理结果取决于使用哪个方法(mixin或扩展)：


* **[extend](#extend-feature)**: 当选择器被扩展，只有新的选择器被标记为_not referenced_，并被写入到引用`@import`的位置。
* **[mixins](#mixins-feature)**: 当`reference`样式作为[implicit mixin](#mixins-feature)，它的规则会被混入，标记为“not reference"，并原样出来在被引用的位置。


#### 引用的示例
这允许你通过以下操作从库，比如[Bootstrap](https://github.com/twbs/bootstrap)，中只引入特定的、有目标的样式。

```less
.navbar:extend(.navbar all) {}
```

你将只从Bootstrap中拉取`.navbar`相关的样式


### 行内(inline)

使用 `@import (inline)`导入外部文件，但不处理它们。

发布版本 [v1.5.0]({{ less.master.url }}CHANGELOG.md)

示例: `@import (inline) "not-less-compatible.css";`

当CSS文件不被Less兼容时使用这个；这是因为尽管Less支持大多数已知标准CSS,它在某些地方不支持注释，并且在不修改CSS的情况下也不支持所有已知的CSS攻击。

因此，你可以用它将文件包含在输出中，以便所有css文件都在一个文件中。


### less

使用`@import (less)`作为Less文件导入，不管文件扩展名。

发布版本 [v1.4.0]({{ less.master.url }}CHANGELOG.md)

示例:

```less
@import (less) "foo.css";
```

### css

使用`@import (css)`作为常规CSS文件导入，不管文件扩展名。意义就是被导入的将保持原样。

发布版本 [v1.4.0]({{ less.master.url }}CHANGELOG.md)

示例:

```less
@import (css) "foo.less";
```
outputs

```less
@import "foo.less";
```

### once

`@import`默认行为。意思是文件只被导入一次，后面导入这个文件将被忽略。

发布版本 [v1.4.0]({{ less.master.url }}CHANGELOG.md)

示例:

```less
@import (once) "foo.less";
@import (once) "foo.less"; // 这里面声明将被忽略
```


### multiple

使用`@import (multiple)`允许多次导入相同文件。它的行为与once相反。

发布版本 [v1.4.0]({{ less.master.url }}CHANGELOG.md)

示例:

```less
// file: foo.less
.a {
  color: green;
}
// file: main.less
@import (multiple) "foo.less";
@import (multiple) "foo.less";
```
结果

```less
.a {
  color: green;
}
.a {
  color: green;
}
```

### optional

使用`@import (optional)`只有文件存在时才导入。如果不使用`optional`关键字，当需要导入的文件找不到时Less抛出FileError并且停止编译。
发布版本 [v2.3.0]({{ less.master.url }}CHANGELOG.md)
