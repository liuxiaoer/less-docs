> "if"'s around selectors

发布版本 [v1.5.0]({{ less.master.url }}CHANGELOG.md)

就像Mixin Guadrds,guards也可以应用于css选择器，这是声明mixin并立即调用它的语法糖。

在1.5.0之前你不得不这么做：

```less
.my-optional-style() when (@my-option = true) {
  button {
    color: white;
  }
}
.my-optional-style();
```

现在你可以直接在样式里面应用guard。

```less
button when (@my-option = true) {
  color: white;
}
```

你也可以结合`&`功能实现一个`if`类型语句，来允许你来给多个guards分组.

```less
& when (@my-option = true) {
  button {
    color: white;
  }
  a {
    color: blue;
  }
}
```

你也可以使用`if()`函数和变量调用来实现一个相似的模式。如下：
```less
@dr: if(@my-option = true, {
  button {
    color: white;
  }
  a {
    color: blue;
  }
});
@dr();
```
