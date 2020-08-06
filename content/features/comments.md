> TODO

## CSS 注释

CSS样式中的注释被Less保留:

```less
.class {
  /* Hello, I'm a CSS-style comment */
  color: black
}
```

## Less 注释

单行注释在Less中是合法，但是它们是‘silent’，它们不会输出到CSS文件中：

```less
.class {
  // Hi, I'm a silent comment, I won't show up in your CSS
  color: white
}
```

{{#todo}}
* 文档 `/*! ... */`
* 文档 and/or 链接到 `--s0`/`--s1` 选项
{{/todo}}
