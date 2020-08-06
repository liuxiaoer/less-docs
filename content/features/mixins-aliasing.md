发布版本 [v3.5.0]({{ less.master.url }}CHANGELOG.md)

> 将mixin调用分配给变量

Mixins can be assigned to a variable to be called as a variable call, or can be used for map lookup.
Mixin可以分配给变量作为变量调用，或者作为map循环。

```less
#theme.dark.navbar {
  .colors(light) {
    primary: purple;
  }
  .colors(dark) {
    primary: black;
    secondary: grey;
  }
}

.navbar {
  @colors: #theme.dark.navbar.colors(dark);
  background: @colors[primary];
  border: 1px solid @colors[secondary];
}
```

输出:

```css
.navbar {
  background: black;
  border: 1px solid grey;
}
```

#### 变量调用

整个mixin调用可以作为变量调用进行别名和调用。如下：

```less
#library() {
  .rules() {
    background: green;
  }
}
.box {
  @alias: #library.rules();
  @alias();
}
```
输出:
```css
.box {
  background: green;
}
```

与根目录使用mixin不同,分配给变量的mixin调用和无参数调用通常需要圆括号。下面不合法：

```less
#library() {
  .rules() {
    background: green;
  }
}
.box {
  @alias: #library.colors; //通常需要圆括号
  @alias();   // ERROR: Could not evaluate variable call @alias
}
```

This is because it's ambiguous if variable is assigned a list of selectors or a mixin call. For example, in Less 3.5+, this variable could be used this way.

这是因为如果变量被分配一个选择器列表或一个mixin调用，那么它是不明确的。例如，在Less3.5+,这个变量可以这么使用。

```less
.box {
  @alias: #library.colors;
  @{alias} {
    a: b;
  }
}
```
输出:
```css
.box #library.colors {
  a: b;
}
```
