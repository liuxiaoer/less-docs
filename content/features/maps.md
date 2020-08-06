发布版本 [v3.5.0]({{ less.master.url }}CHANGELOG.md)

> 用规则集和mixin做为值的映射

通过将命名空间与查找`[]`语法相结合，你可以装饰规则集/mixin转换为映射。

```less
@sizes: {
  mobile: 320px;
  tablet: 768px;
  desktop: 1024px;
}

.navbar {
  display: block;

  @media (min-width: @sizes[tablet]) {
    display: inline-block;
  }
}
```
输出:
```css
.navbar {
  display: block;
}
@media (min-width: 768px) {
  .navbar {
    display: inline-block;
  }
}
```

mixin由于名称空间和重载mixin的能力，所以作为映射，mixin的用途更广一些。

```less
#library() {
  .colors() {
    primary: green;
    secondary: blue;
  }
}

#library() {
  .colors() { primary: grey; }
}

.button {
  color: #library.colors[primary];
  border-color: #library.colors[secondary];
}
```
结果:
```css
.button {
  color: grey;
  border-color: blue;
}
```

你也可以通过使用[aliasing mixins](#mixins-feature-mixin-aliasing-feature)让它更简单。

```less
.button {
  @colors: #library.colors();
  color: @colors[primary];
  border-color: @colors[secondary];
}
```

如果查找值得到规则集，你可以在后面使用第二个`[]`：

```less
@config: {
  @options: {
    library-on: true
  }
}

& when (@config[@options][library-on] = true) {
  .produce-ruleset {
    prop: val;
  }
}
```

使用这方法，规则集和变量调用可以模拟一种与mixin相似的”命名空间“类型。

是否使用mixin或规则集分配给变量作为映射，由你决定。你可能想用重新声明规则集分配给变量替换整个map。或者你可能想“合并”个人的键值对。这各情况mixin作为映射可能更合适。


### 在查找中使用变量的变量

重要提示：在`[@lookup]`的值是`@lookup`的名称，不作为变量计算。如果你想键名成为变量，你可以使用`@@variable`语法。

E.g.
```less
.foods() {
  @dessert: ice cream;
}

@key-to-lookup: dessert;

.lunch {
  treat: .foods[@@key-to-lookup];
}
```
结果:
```css
.lunch {
  treat: ice cream;
}
```
