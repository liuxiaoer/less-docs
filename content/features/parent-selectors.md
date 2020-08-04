> 使用`&`引用父类选择器

`&`相当于父类选择器[nested rule](#features-overview-feature-nested-rules) ，并且常用的用于将修改类或伪类应用到现有选择器时。


```less
a {
  color: blue;
  &:hover {
    color: green;
  }
}
```

结果:

```css
a {
  color: blue;
}

a:hover {
  color: green;
}
```

注意：如果不使用`&`，上面的例子将得到一个`a :hover`规则(`<a>`的子标签选择器的hover)。这不是我们通常希望的嵌套`:hover`

“父选择器"有多样化的使用方式。基本上，任何时候你需要的嵌套选择器都不是默认的。例如另一种典型的用法就是使用`&`做为重复类的名字。

```less
.button {
  &-ok {
    background-image: url("ok.png");
  }
  &-cancel {
    background-image: url("cancel.png");
  }

  &-custom {
    background-image: url("custom.png");
  }
}
```

输出:

```css
.button-ok {
  background-image: url("ok.png");
}
.button-cancel {
  background-image: url("cancel.png");
}
.button-custom {
  background-image: url("custom.png");
}
```

### 多个`&`一起使用

可能在一个选择器中出现多次。使不使用重复的名字而引用父选择器成为可能。

```less
.link {
  & + & {
    color: red;
  }

  & & {
    color: green;
  }

  && {
    color: blue;
  }

  &, &ish {
    color: cyan;
  }
}
```

输出:

```css
.link + .link {
  color: red;
}
.link .link {
  color: green;
}
.link.link {
  color: blue;
}
.link, .linkish {
  color: cyan;
}
```

注意：`&` 相当于所有的父选择器(不仅仅只是最近的父选择器)。示例如下：

```less
.grand {
  .parent {
    & > & {
      color: red;
    }

    & & {
      color: green;
    }

    && {
      color: blue;
    }

    &, &ish {
      color: cyan;
    }
  }
}
```

结果:

```css
.grand .parent > .grand .parent {
  color: red;
}
.grand .parent .grand .parent {
  color: green;
}
.grand .parent.grand .parent {
  color: blue;
}
.grand .parent,
.grand .parentish {
  color: cyan;
}
```


### 改变选择器顺序

在继承的(父)选择器前面插入一个选择器是很有用的。可以在当前选择器后面加上`&`来完成。
例如，当使用Modernizr，你可能需要根据基于支持的功能指定不同的规则：

```less
.header {
  .menu {
    border-radius: 5px;
    .no-borderradius & {
      background-image: url('images/button-background.png');
    }
  }
}
```

选择器 `.no-borderradius &` 将会把 `.no-borderradius` 插入到 `.header .menu` 前面，即 `.no-borderradius .header .menu`:

```css
.header .menu {
  border-radius: 5px;
}
.no-borderradius .header .menu {
  background-image: url('images/button-background.png');
}
```


### 组合使用

`&`也可用于逗号分隔列表中生成选择器的所有可能排列：

```less
p, a, ul, li {
  border-top: 2px dotted #366;
  & + & {
    border-top: 0;
  }
}
```

这将扩展到指定元素的所有可能(16)组合。
```css
p,
a,
ul,
li {
  border-top: 2px dotted #366;
}
p + p,
p + a,
p + ul,
p + li,
a + p,
a + a,
a + ul,
a + li,
ul + p,
ul + a,
ul + ul,
ul + li,
li + p,
li + a,
li + ul,
li + li {
  border-top: 0;
}
```
