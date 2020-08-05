> 在mixin调用中选择属性和变量

#### 属性/值访问器

发布版本 [v3.5.0]({{ less.master.url }}CHANGELOG.md)_

从Less3.5开始，你可以使用属性/值访问器从带变量的mixin规则集中来选择值。类似函数一样的使用mixin。

示例:

```less
.average(@x, @y) {
  @result: ((@x + @y) / 2);
}

div {
  // 调用一个mixin并且查找"@result"值
  padding: .average(16px, 50px)[@result];
}
```

结果:

```css
div {
  padding: 33px;
}
```

#### 重写mixin值

如果有多个匹配的mixin，所有的规则集被执行和合并，并使用最后一个匹配mixin的值。这类似CSS的级联，它允许你“重写”mixin的值。

```less
// library.less
#library() {
  .mixin() {
    prop: foo;
  }
}

// customize.less
@import "library";
#library() {
  .mixin() {
    prop: bar;
  }
}

.box {
  my-value: #library.mixin[prop];
}
```
结果:
```css
.box {
  my-value: bar;
}
```

#### 未命名查找

如果没有指定查找值`[@lookup]`而是只写`[]`，所有的值都会级联并选择最后一个声明的值。

含义: 上面的平均值mixin示例写成：
```less
.average(@x, @y) {
  @result: ((@x + @y) / 2);
}

div {
  // 调用并查找值
  padding: .average(16px, 50px)[];
}
```
同样输出:
```css
div {
  padding: 33px;
}
```

mixin调用中规则集和变量别名有相同的级联行为。
```less
@dr: {
  value: foo;
}
.box {
  my-value: @dr[];
}
```
输出:
```css
.box {
  my-value: foo;
}
```


#### 将mixin&变量解锁到调用域

***DEPRECATED - 使用属性/值访问器***

在调用域中mixin中定义的变量和mixin都是可见的。只有一个例外：如果调用者包含一个相同的变量名，这个变量不会被拷贝（包含被另一个mixin调用定义的变量）。只有调用者局部作用域中存在的变量才受保护。从父作用域中继承的变量将被重写。

注意: 这个行为已弃用, 在将来, 这种方式将不会合并变量和mixins到调用者域中。

示例:

```less
.mixin() {
  @width:  100%;
  @height: 200px;
}

.caller {
  .mixin();
  width:  @width;
  height: @height;
}

```
结果:

```css
.caller {
  width:  100%;
  height: 200px;
}
```

在调用者域中直接定义的变量无法被重写。然而，调用者父作用域中定义的变量不受保护将会被重写。

````less
.mixin() {
  @size: in-mixin;
  @definedOnlyInMixin: in-mixin;
}

.class {
  margin: @size @definedOnlyInMixin;
  .mixin();
}

@size: globaly-defined-value; // 调用者的父作用域 - 不受保护
````

结果:
````css
.class {
  margin: in-mixin in-mixin;
}
````

最后，mixin中定义的mixin也可以作为返回值：
````less
.unlock(@value) { // 外层mixin
  .doSomething() { // 嵌套mixin
    declaration: @value;
  }
}

#namespace {
  .unlock(5); // 解锁doSomething mixin
  .doSomething(); //嵌套mixin拷贝到这并且可用
}
````

结果:
````css
#namespace {
  declaration: 5;
}
````
