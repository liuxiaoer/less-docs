> 从已存在的样式中组合属性

你可以组合类选择器和id选择器。例如

```less
.a, #b {
  color: red;
}
.mixin-class {
  .a();
}
.mixin-id {
  #b();
}
```
结果:
```css
.a, #b {
  color: red;
}
.mixin-class {
  color: red;
}
.mixin-id {
  color: red;
}
```

再有版本和历史版本, mixin调用中的圆括号是可选的，但是可选圆括号是不推荐使用的，并且在将来的版本中必须使用。

```less
.a(); 
.a;  // 现在可以使用,但不推荐; 不要使用
```

## 不输出Mixin

如果你想创建一个mixin但是你不想mixin出现在CSS输出中，请在mixin定义后面加上圆括号。

```less
.my-mixin {
  color: black;
}
.my-other-mixin() {
  background: white;
}
.class {
  .my-mixin();
  .my-other-mixin();
}
```
结果

```css
.my-mixin {
  color: black;
}
.class {
  color: black;
  background: white;
}
```

## Mixins中的选择器

Mixins can contain more than just properties, they can contain selectors too.
Mixins不公可以包含属性，还可以包括选择器。

例如:

```less
.my-hover-mixin() {
  &:hover {
    border: 1px solid red;
  }
}
button {
  .my-hover-mixin();
}
```

结果

```css
button:hover {
  border: 1px solid red;
}
```

## 命名空间

如果想在一个很复杂的选择器使用mixin属性，可以将多个Id或类选择器叠加起来。

```less
#outer() {
  .inner {
    color: red;
  }
}

.c {
  #outer > .inner();
}
```

`>`和空格是可选的

```less
// 下面相同功能
#outer > .inner();
#outer .inner();
#outer.inner();
```

给你的mixins命名可以减少与其它的库或者用户mixins冲突，但是它也可以是一种“组织”mixins组的方法。

示例:

```less
#my-library {
  .my-mixin() {
    color: black;
  }
}
// 可以这么使用
.class {
  #my-library.my-mixin();
}
```

## Guarded Namespaces(带条件的命名)

如果mixins在带有条件的命名空间中定义，那么只有满足命名空间条件的情况才能正常使用。在命名空间上的条件和在mixins上的条件是等同的，所以下面两个mixins具有相同功能：

```less
#namespace when (@mode = huge) {
  .mixin() { /* */ }
}

#namespace {
  .mixin() when (@mode = huge) { /* */ }
}
```

`default`函数是假设所有嵌套命名空间和mixin有相同的值。下面的mixin不会执行，其中一个条件不满足。
```less
#sp_1 when (default()) {
  #sp_2 when (default()) {
    .mixin() when not(default()) { /* */ }
  }
}
```

## `!important`关键字

在mixin调用后面使用`!important`关键字，使所有属性做为`!important`继承它：

示例:

```less
.foo (@bg: #f5f5f5, @color: #900) {
  background: @bg;
  color: @color;
}
.unimportant {
  .foo();
}
.important {
  .foo() !important;
}
```

结果:

```css
.unimportant {
  background: #f5f5f5;
  color: #900;
}
.important {
  background: #f5f5f5 !important;
  color: #900 !important;
}
```
