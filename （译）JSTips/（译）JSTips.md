# （译）JSTips

#1 数组元素插入 
[源地址](https://github.com/loverajoel/jstips/blob/gh-pages/_posts/en/2015-12-29-insert-item-inside-an-array.md)
向已存在的数组中插入元素已经成为日常了，你可以使用`push`方法想数组尾插入新元素，也可以`unshift`新元素到组头，或者应用`splice`方法把元素加入到数组中间。
以上都是已知的方法，但是并不意味着没有更高效的方法。
使用`push()`向数组添加新元素虽然简单，但并不是最高效的方法：

```js
var arr = [1,2,3,4,5];

arr.push(6);
arr[arr.length] = 6; // 43% faster in Chrome 47.0.2526.106 on Mac OS X 10.11.1
```

以上两种方法都改变了原数组，你不相信？可以去[jsperf](http://jsperf.com/push-item-inside-an-array)查看。
现在，如果我们尝试在数组首部添加元素：

```js
var arr = [1,2,3,4,5];

arr.unshift(0);
[0].concat(arr); // 98% faster in Chrome 47.0.2526.106 on Mac OS X 10.11.1
```

**细节：** `unshift`修改原数组，`concat`通过复制参数数组内元素，返回一个新数组。
在数组中间添加新元素，通常使用数组的`splice`方法，并且他是效率最高的方法。

```js
var items = ['one', 'two', 'three', 'four'];
items.splice(items.length / 2, 0, 'hello');
```

以上几种我在多种浏览器和OS进行了测试，结果均相近。我希望这些小知识点能够帮助到你提升你的效率。

#2 AngularJS digest VS apply

[源地址]([https://github.com/loverajoel/jstips/blob/gh-pages/\_posts/en/2016-01-01-angularjs-digest-vs-apply.md](https://github.com/loverajoel/jstips/blob/gh-pages/_posts/en/2016-01-01-angularjs-digest-vs-apply.md)
Angular最值得称道的就是它的双向绑定思想，为了它能实现，AngularJS通过循环(`$digest`)对**model**层和**view**层做判断。若想理解这个框架的关键，你需要去理解这一思想。

##`$apply`
`$apply`这一方法能让你明确地开始理解这一循环比较过程。此意味着所有观察者都被检查；整个应用均开始`$digest`循环。其中，当结束一个作为参数传入的可选处理函数，Angular都会调用`$rootScope.$degest()`方法。
##`digest`
`$digest`方法在这种情况下开始为当前作用域以及其子作用域进行`$digest`循环。可以发现其父组件的作用域并没有被检查和影响。
##建议
* 当浏览器DOM节点在AngularJS外触发事件时，使用`$apply`或`$digest`方法
* 向`$apply`方法传入回调函数会导致错误的处理机制，并且会使变化同时发生在`$digest`循环当中。

```js
$scope.$apply(() => {
    $scope.tip = 'Javascript Tip';
});
```

* 最好的效率是自身作用域被更新，如果仅仅是想更新当前作用域或子作用域，使用`$digest`方法，同时禁止全局进行digest循环。
* `$apply()`方法是一个硬处理方法，并且当绑定过多时，会导致效率问题。
* 如果你使用AngularJS 1.2.X版本，使用`$evalAsync`，它会在当前或下次循环时判断表达式值变化。可以有效提高应用整体效率。