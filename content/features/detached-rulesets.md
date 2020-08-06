> 为变量分配规则集

发布版本 [v1.7.0]({{ less.master.url }}CHANGELOG.md)

（Detached ruleset）分离的规则集是一组css属性，嵌套规则集，媒体声明或者存储在变量中的其它任何内容。你可以将其包含到规则集或其它结构中，并且它的所有属性将被拷贝到那里。你还可以用它做为mixin参数，并将其作为任何其它变量传递。

简单示例:
````less
// declare detached ruleset
@detached-ruleset: { background: red; }; // 3.5.0+版本分号是可选的

// 使用 detached ruleset
.top {
    @detached-ruleset(); 
}
````

输出:
````css
.top {
  background: red;
}
````

detached ruleset调用必须在后面加圆括号（除非后面跟有[lookup value](#detached-rulesets-feature-property-variable-accessors)）。调用`@detached-ruleset;`不起作用。

如果你想定义一个可以抽象出在媒体查询中包含一段代码或者一个不受支持的浏览器类名的mixin时，它是很有用的。规则集可以被传递到mixin，所有mixin可以包装内容。

```less
.desktop-and-old-ie(@rules) {
  @media screen and (min-width: 1200px) { @rules(); }
  html.lt-ie9 &                         { @rules(); }
}

header {
  background-color: blue;

  .desktop-and-old-ie({
    background-color: red;
  });
}
```

这里`desktop-and-old-ie` mixin定义了一个媒体查询和根类，所以你可以用mixin包含一段代码。下面是输出

```css
header {
  background-color: blue;
}
@media screen and (min-width: 1200px) {
  header {
    background-color: red;
  }
}
html.lt-ie9 header {
  background-color: red;
}
```

现在规则集可以分配到变量上去或者传递到mixin，可以包含全套的Less功能。

```less
@my-ruleset: {
    .my-selector {
      background-color: black;
    }
  };
```

你甚至可以使用[media query bubbling](#features-overview-feature-media-query-bubbling-and-nested-media-queries)，如下

```less
@my-ruleset: {
    .my-selector {
      @media tv {
        background-color: black;
      }
    }
  };
@media (orientation:portrait) {
    @my-ruleset();
}
```

输出

```css
@media (orientation: portrait) and tv {
  .my-selector {
    background-color: black;
  }
}
```

分离的规则集调用与mixin调用一样，将其所有mixin解锁(返回)到调用方。但是，它不返回变量。

返回mixin:
````less
// 带有mixin的分离的规则集
@detached-ruleset: { 
    .mixin() {
        color: blue;
    }
};
// 调用分离的规则集
.caller {
    @detached-ruleset(); 
    .mixin();
}
````

结果:
````css
.caller {
  color: blue;
}
````

私有变量:
````less
@detached-ruleset: { 
    @color:blue; // 这是私有变量
};
.caller {
    color: @color; // 语法错误
}
````

## 作用域
分离的规则集可以在定义所有的变量和mixins的地方使用它们。换言之，它对定义者和调用者作用域都可见。如果两个作用域都包含相同变量和mixin，优先使用声明作用域中的值。

*声明作用域*是指分离规则集主体范围。从一个变量复制分离规则集到另一个变量中会修改它的作用域。规则集不能通过在那里引用而获得新作用域的访问权。

分离规则集可以解锁(导入)到其中获取作用域的访问权

_提示: 通过调用mixin解锁变量到作用域已弃用。使用工[属性/变量访问器](#detached-rulesets-feature-property-variable-accessors)。_

#### 定义作用域和调用作用域的可见性

分离规则集可以访问调用者作用域的变量和mixins:

````less
@detached-ruleset: {
  caller-variable: @caller-variable; // 变量还未定义
  .caller-mixin(); // mixin还未定义
};

selector {
  // 使用分离规则集
  @detached-ruleset(); 

  // 需要在分离规则集中定义变量和mixin
  @caller-variable: value;
  .caller-mixin() {
    variable: declaration;
  }
}
````

编译成:
````css
selector {
  caller-variable: value;
  variable: declaration;
}
````

从定义访问变量和mixin胜过调用程序中可用的变量和mixin：

````less
@variable: global;
@detached-ruleset: {
  // 将会使用全局的变量，因为分离规则集定义中是可以访问它的。
  variable: @variable; 
};

selector {
  @detached-ruleset();
  @variable: value; // 调用者作用域中定义 - 被忽略
}
````

编译成:
````css
selector {
  variable: global;
}
````

#### 引用 *不要* 修改分离规则集作用域

规则集无法仅仅通过被引用来获得对新的作用域的访问权：

````less
@detached-1: { scope-detached: @one @two; };
.one {
  @one: visible;
  .two {
    @detached-2: @detached-1; // 复制/重命名 规则集
    @two: visible; //规则集无法访问此变量
  }
}

.use-place {
  .one > .two(); 
  @detached-2();
}
````

抛异常:
````
ERROR 1:32 The variable "@one" was not declared.
````

#### 解锁 *将* 修改分离规则集的作用域
规则集通过解锁(导入)在作用域中获取访问权。
````less
#space {
  .importer-1() {
    @detached: { scope-detached: @variable; }; // 定义分离规则集
  }
}

.importer-2() {
  @variable: value; // 解锁的分享规则集可以访问此变量
  #space > .importer-1(); // 解锁/导入分离规则集
}

.use-place {
  .importer-2(); // 第二次解锁/导入分离规则集
   @detached();
}
````

编译成:
````css
.use-place {
  scope-detached: value;
}
````
