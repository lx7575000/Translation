[原文地址](http://ryanchristiani.com/functions-as-first-class-citizens-in-javascript/)
使用JavaScript的我们是幸运的，因为我们可以将函数方法进行多种形式的传递。

#一级公民？！
在编程语言当中，当你能够传递、返回、赋值一个类型时，该类型通常被视为”一级公民“。这也是JavaScript为什么变得如此受函数式编程欢迎的一个原因：它可以创建一个函数方法，并且可以传递函数作为参数，令函数作为返回值。因此，我们可以直接创建其他语言需要复杂来实现的高级别函数方法。
我们可以令函数与其他变量一样作为变量进行传递、返回。

#赋值函数
在JS我们可以用**多种方法**将函数赋值给一个变量来进行使用。
##使用 var

```js
var myFunction = function(){
  // 具体方法
}
```

最常见的方法就是使用`var`来将一个函数方法赋值给一个变量。我们在此使用匿名函数的形式进行赋值，不过我们也可以通过明明函数的形式将函数方法的**引用**赋值给其他变量。**可以看到这么做的好处的是，方法可以递归调用自身。**

```js
var myFunction = function functionName(n){
  if(n < 10){
    n++;
    //具体方法。
    functionName(n);
  }
}
```

##方法形式
在JS中赋值一个方法非常简单，同样我们可以通过在**对象**当中使用关键字的形式，赋值一个方法。

```js
var warrior = {
    hp: 100,
    strength: 20,
    attack: function(target) {
        target.hp -= this.strength;
    }
}
```

有件事你需要记住，当我们创建如下函数时。存储在**add**变量中的是一个函数，因此我们可以不需要调用函数，直接传递值。

```js
var add = function(a,b) {
    return a + b;
}
var newAdd = add;
newAdd(3, 2) //5
```

#传递函数
函数作为JavaScript的一级公民，可以对他们进行传递。初学者通常会以如下形式运用。我们假设需要对不同事件进行分析排序。

```js
$('form').on('submit',function() {
    //perform some analytics.
});

$('a[href$=".pdf"]').on('click', function() {
    //perform same analytics.
});
```

上述做法有点过于重复，我们希望用其他简洁方式进行处理。通过传递函数的形式我们可以减少重复劳动，通过传递一个函数变量，来解决上述问题。

```js
function analyticsHandler(e) {
    ...//perform some analytics.
}

$('form').on('submit',analyticsHandler);

$('a[href$=".pdf"]').on('click',analyticsHandler);
```

通过这种方式，JQuery将函数视作一个变量值，当事件被触发时候回调函数进行调用。

```js
$('a').on('click', function() {
    //...
});
```

当`<a>`标签被点击，JQuery会进行函数回调，在上面`analyticsHandler`例子中，我们将函数存储在变量当中方便后面使用，并且帮助我们保持DRY。
##返回函数
有时我们会关注JavaScript函数返回值同为函数。这是进行函数式编程的重要概念。
当我们编写gulp配置文件时，我们使用`require`从**Nodejs**获取脚本到当前文件中。

```js
var cssNext = require('gulp-cssnext');
```

在**Node**中编写的模块，它可以是多种类型。但是，最常见的是返回函数。这也是为什么当我们`require`**gulp**插件时，你总要在后面进行函数调用。因为我们require到的模块经常是函数。
下面是一个`gulp-cssnext`插件例子：

```js
module.exports = function(options) {
  return through.obj(transform(options))
}
```

上述例子，应用`module.exports`传出一个返回函数自身结果的函数方法，通过上述方法，我们可以在我们的gulp文件中使用返回的函数。

```js
gulp.task('css',function() {
    gulp.src('*.css')
        .pipe(cssnext())
        .pipe(dist.);
});
```

在许多**Node**应用，尤其是`express`应用中，你会经常看到如下或类似的一行代码：

```
var app = require('express')();
```

或者如`postcss`中，你需要require相关插件：

```
postcss([ require('cssnext')(), require('cssnano')() ])
```

##Partial application（想不出好的翻译）
另一种常见返回函数的应用形式，是通过闭包方式来使用Partial application。Partial application 用于创建一个函数，并返回一个函数。要求存储第一个参数。

```js
function add(a) {
    return function(b) {
        return a + b;
    }   
}
```

这个函数通过闭包的形式存储参数在上级作用域当中，返回一个函数，并在返回的函数中应用上级作用域存储的参数。下例说明，该函数如何运用。

```js
var add5 = add(5);
```

此时，返回函数存储在**add5**变量中。在其内引用上级**add**函数作用域传入的**a**参数，并存储在自身作用域内。

```js
function(b) {
    return a + b;
}
```

接着，我们就可以应用`add5`函数了。

```js
add5(5); // 10
add5(8); // 13
```

JavaScript是一种学起来有趣且非常强大的语言，并且在其中函数作为一级公民。希望上面所述能够帮助你理解JS的函数方法。

（译者）写在最后：作为英语四级勉强过线选手，如果翻译有不到位地方，请多包涵。如若不能包涵就请跟着链接跳转看原文。当然，如果你感觉文章内容不错，就我给个star吧。



