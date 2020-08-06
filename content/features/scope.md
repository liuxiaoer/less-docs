---
published: false
---

> 如何计算变量

## 提示与技巧

Credit: [less/less.js/issues/1472]({{ site.coderepo }}/issues/1472#issuecomment-22213697)

这是一个技巧，可以定义变量并将它们保持在某个私有作用域中，防止它们泄漏到全局空间。

```less
& {
  // Vars
  @height: 100px;
  @width: 20px;
  // 不要在这个作用域定义属性:值 (这样做将生成(错误)输出).

  .test {
    height: @height;
   width: @width;
  }
}

.rest {
  height: @height; // Name error: @height变量未定义
}
```

`@height` 和 `@width`只对`& { ... }`的作用域有定义。你也可以在规则中嵌套作用域：


```less
.some-module {
  @height: 200px;
  @width: 200px;
  text-align: left;
  line-height: @height; // 200px

  & {
    // 重写原来的值
    @height: 100px;
    @width: auto;

    .some-module__element {
      height: @height; // 100px
      width: @width; // 200px
    }

    .some-module__element .text {
      line-height: (@height / 2); // 50px
    }
  }

  & {
    // 重写原来的值
    @height: 50px;

    .some-module__another-element {
      height: @height; // 50px
      width: @width; // 200px
    }

    .some-module__another-element .text {
      line-height: (@height / 2); // 25px
    }
  }
}
```
