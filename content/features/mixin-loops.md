> 创建loops

在Less中mixin可以调用自身。这样的递归mixin，当结合[Guard Expressions](#mixin-guards-feature) 和 [Pattern Matching](#mixins-parametric-feature-pattern-matching)，可以用来创建各种迭代/循环结构。

示例:

```less
.loop(@counter) when (@counter > 0) {
  .loop((@counter - 1));    // 下一次迭代
  width: (10px * @counter); // 每次迭代
}

div {
  .loop(5); // 执行循环
}
```

输出:

```css
div {
  width: 10px;
  width: 20px;
  width: 30px;
  width: 40px;
  width: 50px;
}
```

一个使用迭代循环生成CSS表格类的通用的示例：

```less
.generate-columns(4);

.generate-columns(@n, @i: 1) when (@i =< @n) {
  .column-@{i} {
    width: (@i * 100% / @n);
  }
  .generate-columns(@n, (@i + 1));
}
```

输出:

```css
.column-1 {
  width: 25%;
}
.column-2 {
  width: 50%;
}
.column-3 {
  width: 75%;
}
.column-4 {
  width: 100%;
}
```
