---
标题: 变量
---

> 控制单个位置中常用的值。

### Overview(概述)
同一个值在样式表中重复几十次甚至几百次并不少见

```css
a,
.link {
  color: #428bca;
}
.widget {
  color: #fff;
  background: #428bca;
}
```

变量提供一种从单个位置控制这些常量方法，使你的代码更容易维护:

```less
// Variables
@link-color:        #428bca; // sea blue
@link-color-hover:  darken(@link-color, 10%);

// Usage
a,
.link {
  color: @link-color;
}
a:hover {
  color: @link-color-hover;
}
.widget {
  color: #fff;
  background: @link-color;
}
```

### Variable Interpolation(可变插值)

上面的例子使用变量控制css规则中的值，但是他们也可以在其它位置很好的使用，例如选择器名、属性名、URL和`@import`声明。


#### 选择器

_v1.4.0_

```less
// Variables
@my-selector: banner;

// Usage
.@{my-selector} {
  font-weight: bold;
  line-height: 40px;
  margin: 0 auto;
}
```
编译成:

```css
.banner {
  font-weight: bold;
  line-height: 40px;
  margin: 0 auto;
}
```

#### URLs

```less
// Variables
@images: "../img";

// Usage
body {
  color: #444;
  background: url("@{images}/white-sand.png");
}
```

#### Import 声明

_v1.4.0_

语法: `@import "@{themes}/tidal-wave.less";`

注意：在v2.0.0之前，只有考虑在根或者当前域声明的变量，并且在查找变量时，只考虑当前文件和被调用的文件

示例:

```less
// 变量
@themes: "../../src/themes";

// 使用
@import "@{themes}/tidal-wave.less";
```

#### 属性名

_v1.6.0_

```less
@property: color;

.widget {
  @{property}: #0ee;
  background-@{property}: #999;
}
```

编译成:

```css
.widget {
  color: #0ee;
  background-color: #999;
}
```

### 变量的变量

在Less中你可以使用变量去定义另外一个变量的名字


```less
@primary:  green;
@secondary: blue;

.section {
  @color: primary;
  
  .element {
    color: @@color;
  }
}
```

编译成:

```less
.section .element {
  color: green;
}
```

<span class="anchor-target" id="variables-feature-lazy-loading"></span>
<!-- ^ please keep old anchor to not break zillion outer links -->
### 差评/缺点

> 变量必须在使用前定义

有效Less代码:

```less
.lazy-eval {
  width: @var;
}

@var: @a;
@a: 9%;
```
有效Less代码:

```less
.lazy-eval {
  width: @var;
  @a: 9%;
}

@var: @a;
@a: 100%;
```
上面两种写法都会编译成:

```css
.lazy-eval {
  width: 9%;
}
```

When defining a variable twice, the last definition of the variable is used, searching from the current scope upwards. This is similar to css itself where the last property inside a definition is used to determine the value.
当变量重复被定义，会使用从当前域向上查找到的最后一次定义的值。与css自有特性一样，使用定义中的最后一个属性值

例如:

```less
@var: 0;
.class {
  @var: 1;
  .brass {
    @var: 2;
    three: @var;
    @var: 3;
  }
  one: @var;
}
```
编译成:

```css
.class {
  one: 1;
}
.class .brass {
  three: 3;
}
```

实际上每个域都有一个final值，与类似于浏览器中的属性。

```css
.header {
  --color: white;
  color: var(--color);  // the color is black
  --color: black;
}
```

也就是说，Less变量的行为很像CSS，不像其它预处理的CSS。


### 属性做为变量 **(新!)**

_v3.0.0_

你可以很容易的使用`$prop`语法像变量一样来使用属性。有时这种写法会你写代码更轻松一点。

```less
.widget {
  color: #efefef;
  background-color: $color;
}
```

编译成:

```css
.widget {
  color: #efefef;
  background-color: #efefef;
}
```

注意：像变量一样，Less会选择在当前/父域中找到的最后一次属值做为最终值。

```less
.block {
  color: red; 
  .inner {
    background-color: $color; 
  }
  color: blue;  
} 
```

编译成:
```css
.block {
  color: red; 
  color: blue;  
} 
.block .inner {
  background-color: blue; 
}
```

### 默认变量

我们时常有默认变量的请求-在没有设置值的情况下。默认亦是不是必需的，因为很容易事后定义变量重写一个变量。

例如:

```less
// library
@base-color: green;
@dark-color: darken(@base-color, 10%);

// use of library
@import "library.less";
@base-color: red;
```

This works fine because of [Lazy Loading](#variables-feature-lazy-loading) - `@base-color` is overridden and `@dark-color` is a dark red.
这样会正常运行，因为[Lazy Loading](#variables-feature-lazy-loading)- `@base-color` 被重写，并且 `@dark-color` 被重写成darken(red).
