> Less允许像正常级联CSS一样使用嵌套

...

### 嵌套媒体查询

媒体查询可以使用选择器相同方法被嵌套，输出将在媒体查询中创建相应的规则。

如何:

```less
.one {
  @media (width: 400px) {
    font-size: 1.2em;
    @media print and color {
      color: blue;
    }
  }
}
```

结果:

```css
@media (width: 400px) {
  .one {
    font-size: 1.2em;
  }
}
@media (width: 400px) and print and color {
  .one {
    color: blue;
  }
}
```

...

### `&` 和 mixins

`&`也可以和mixin一起使用。这种情况，`&`是指另一类中mixin作用域的选择器。

例如:

```less
.mixin {
  &:hover { // & 指下面调用mixin的作用域的选择器,即.blue
    color:cyan;
  }
}
.blue {
  .mixin;//作用域的选择为.blue
  color:blue;
}

```

结果：

```css
.mixin:hover {
  color: cyan;
}

.blue {
  color: blue;
}
.blue:hover {
  color: cyan;
}
```
