发布版本 [v2.5.0]({{ less.master.url }}CHANGELOG.md)

> 导入JavaScript插件以添加Less.js的功能和特性

## 第一个插件

使用`@plugin`与在`.less`文件中使用`@import`相似。
```less
@plugin "my-plugin";  // 如果没有扩展名，自动添加.js
```
由于Less插件是在Less作用域中计算的，所以插件定义可以很简单。

```js
registerPlugin({
    install: function(less, pluginManager, functions) {
        functions.add('pi', function() {
            return Math.PI;
        });
    }
})
```
你可以使用`module.exports`（填充以在浏览器以及Node.js中运行）
```js
module.exports = {
    install: function(less, pluginManager, functions) {
        functions.add('pi', function() {
            return Math.PI;
        });
    }
};
```
注意其它Node.js CommonJS约定，就像`require()`不能在浏览器中使用。当写跨平台的件时记住这些。

插件可以干什么？很多，但是我们先看看基本的。我们将先把重点放在`install`函数中。假设你写了这个：

```js
// my-plugin.js
install: function(less, pluginManager, functions) {
    functions.add('pi', function() {
        return Math.PI;
    });
}
// etc
```

恭喜你！你已经写了一个Less插件！


如果在样式表中使用这个插件：
```less
@plugin "my-plugin";
.show-me-pi {
  value: pi();
}
```
结果:
```less
.show-me-pi {
  value: 3.141592653589793;
}
```
但是，你想将其与其它值相乘或其它Less操作，你需要返回一个适当的Less节点。否则输出到样式表中的是纯文本(对你的目的可能很好)。

意思是，这更正确:

```js
functions.add('pi', function() {
    return new tree.Dimension(Math.PI);
});
```

_提示：维度是一个带或者不带单位的数字，`less.Dimension(10, "px")`得到“10px"。查看[Less API](TODO)获取单位列表。_

现在你可以在操作中使用你的函数。
```less
@plugin "my-plugin";
.show-me-pi {
  value: pi() * 2;
}
```

你可能已经注意到你的插件有可用的全局参数，即函数注册表(`functions`对象)和`less`对象。这些是为了方便。

## 插件作用域

在函数前面加上`@plugin`规则遵循Less作用域规则。这对于那些希望添加功能面不引入命名冲突的库作者来说，这非常适合。

例如，来自两个第三方库的两个插件都有一个名称是“foo”的函数。
```js
// lib1.js
// ...
    functions.add('foo', function() {
        return "foo";
    });
// ...

// lib2.js
// ...
    functions.add('foo', function() {
        return "bar";
    });
// ...
```

没关系！你可以选择哪个库的函数创建哪个输出。

```less
.el-1 {
    @plugin "lib1";
    value: foo();
}
.el-2 {
    @plugin "lib2";
    value: foo();
}
```
输出:
```less
.el-1 {
    value: foo;
}
.el-2 {
    value: bar;
}
```

对于分享插件的作者来说，这意为着你也可以有效的在特定作用域创建私有函数代替他们。

```less
.el {
    @plugin "lib1";
}
@value: foo();
```

自Less 3.0起，函数可以返回任意Node类型，可以在任何级别调用。

也就是说下面的代码在2.x中将抛出错误，函数必须是属性值的一部分或变量分配。

```less
.block {
    color: blue;
    my-function-rules();
}
```
在3.x中，现在不是这样了，函数可以返回@规则，规则集，其它Less节点，字符串和数字(最后两个被转换成匿名节点)。

## 空函数

有时你可能想调用函数，但是你不想输出任何东西（比如为后面使用而存储一个值）。在那种情况，你仅仅需要在函数中返回`false`。

```js
var collection = [];

functions.add('store', function(val) {
    collection.push(val);  // imma store this for later
    return false;
});
```
```less
@plugin "collections";
@var: 32;
store(@var);
```

最后你可以像这样做：
```js
functions.add('retrieve', function(val) {
    return new tree.Value(collection);
});
```
```less
.get-my-values {
    @plugin "collections";
    values: retrieve();   
}
```

## Less.js插件对象

Less.js插件需要导出一个含有一个或多个这些属性的对象。
```js
{
    /* 插件第一次导入时立即被调用，只调用一次。 */
    install: function(less, pluginManager, functions) { },

    /* 每次使用@plugin都会被调用 */
    use: function(context) { },

    /* 每次使用@plugin都会被调用，当规则被计算。只是在计算的后期 */
    eval: function(context) { },

    /* 给插件传递任意字符串
     * 例如 @plugin (args) "file";
     * 这个字符串没有解析，所有它可以包含(几乎)任何东西
     */
    setOptions: function(argumentString) { },

    /* 设置一个Less最小兼容的字符串，你也可以使用数组，如 [3, 0] */
    minVersion: ['3.0'],

    /* 只能使用lessc在命令行解释选项 */
    printUsage: function() { },

}
```
`install()`函数的插件管理器实例提供添加访问者，文件管理器和后处理程序的方法。

这里是一些不同插件类型的示例： <!-- TODO: 更新示例 -->
 - post-processor: https://github.com/less/less-plugin-clean-css
 - visitor: https://github.com/less/less-plugin-inline-urls
 - file-manager: https://github.com/less/less-plugin-npm-import

## 预加载插件

While a `@plugin` call works well for most scenarios, there are times when you might want to load a plugin before parsing starts.
大部分行情使用`@plugin`就可以了，除了有时你想在解析之前加载插件。

见: [Pre-Loaded Plugins](../usage/#plugins) 在 "Using Less.js" 章节介绍如何使用.
