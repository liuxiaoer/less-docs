> 合并属性

`merge` 功能允许将多个属性中的值聚合到单属性下以逗号或空格分隔的列表中。`merge`对诸如background、transform等属性非常有用。

### 逗号

> 以逗号拼接属性值

发布版本 [v1.5.0]({{ less.master.url }}CHANGELOG.md)

示例:

```less
.mixin() {
  box-shadow+: inset 0 0 10px #555;
}
.myclass {
  .mixin();
  box-shadow+: 0 0 20px black;
}
```
结果

```css
.myclass {
  box-shadow: inset 0 0 10px #555, 0 0 20px black;
}
```

### 空格

> 以空格拼接属性值

发布版本 [v1.7.0]({{ less.master.url }}CHANGELOG.md)

示例:

```less
.mixin() {
  transform+_: scale(2);
}
.myclass {
  .mixin();
  transform+_: rotate(15deg);
}
```
结果

```css
.myclass {
  transform: scale(2) rotate(15deg);
}
```
为了避免无意的拼接，`merge`在每个拼接定义时需要明确的指定 `+` 或 `+_` 标志。
