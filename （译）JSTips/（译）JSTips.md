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

#3 React keys属性在子组件当中很重要
在`React`当中，**key**属性是你一定要传入给所有相同类型动态创建并且以数组形式保存的子组件。`React`使用这些独一无二的常量*ID*来确认DOM中的各个子组件，并判断他们是否为相同类型组件。使用**key**属性可以用于判断子组件是否发生了变化，并防止奇怪的事情发生。

[ Paul O’Shannessy](https://github.com/facebook/react/issues/1342#issuecomment-39230939)

    key属性最重要的作用不是用于提升效率，它更关心的是确认子组件（这样反而会提高性能）。随机分配或者更改值不会产生一个identity。

* 使用存在对象独一无二的属性作为**key**值。
* 在父组件中就定义好所有子组件的**key**值。

```js
...
render() {
    <div key={{item.key}}>{{item.name}}</div>
}
...
   
//good
<MyComponent key={{item.key}}/>
```

* [不要通过数组的map方法传入的**index**设定**key**值](https://medium.com/@robinpokorny/index-as-a-key-is-an-anti-pattern-e0349aece318#.jadd313vz)
* `random()`在此不会起作用。

```js
//bad
{todos.map((todo, index) =>
  <Todo {...todo}
    key={index} />
)}
```

```js
//bad
<MyComponent key={{Math.random()}}/>
```

* 可以使用适合你自己项目，并且快速的方法创建独一无二的**id**值
* 当子组件很复杂或者数量较多的时候，使用**key**属性来提高你项目的效率。
* 我们应该给[ReactCSSTransitionGroup](http://reactjs.cn/react/docs/animation.html)的子组件设**key**值

#4 提高条件语句效率
[原文地址](https://github.com/loverajoel/jstips/blob/gh-pages/_posts/en/2016-01-03-improve-nested-conditionals.md)

我们如何提升JS中**if**语句的执行效率？

```js
if (color) {
  if (color === 'black') {
    printBlackBackground();
  } else if (color === 'red') {
    printRedBackground();
  } else if (color === 'blue') {
    printBlueBackground();
  } else if (color === 'green') {
    printGreenBackground();
  } else {
    printYellowBackground();
  }
}
```

首先，我们可以使用`switch`语句来替换提升`if`语句。尽管这么做会减少对条件顺序的要求以及多层嵌套的麻烦，但出于对**debug**困难程度的考虑我们仍不推荐这么做。

```js
switch(color) {
  case 'black':
    printBlackBackground();
    break;
  case 'red':
    printRedBackground();
    break;
  case 'blue':
    printBlueBackground();
    break;
  case 'green':
    printGreenBackground();
    break;
  default:
    printYellowBackground();
}
```

但是假如我们存在一种在同一环境下需要对多条件进行判断时，如果我们想要减少啰嗦的语句，并且更有序。我们可以条件性`switch`。我们传入`true`作为`switch`语句，这么做可以让我们在每个`case`中进行条件判断。

```js
switch(true) {
  case (typeof color === 'string' && color === 'black'):
    printBlackBackground();
    break;
  case (typeof color === 'string' && color === 'red'):
    printRedBackground();
    break;
  case (typeof color === 'string' && color === 'blue'):
    printBlueBackground();
    break;
  case (typeof color === 'string' && color === 'green'):
    printGreenBackground();
    break;
  case (typeof color === 'string' && color === 'yellow'):
    printYellowBackground();
    break;
}
```

尽管这么做很爽，但是我们必须避免多条件判断的使用，尽可能减少对`switch`的使用。我们可以考虑通过使用**对象**来进行更高效的判断。

```js
var colorObj = {
  'black': printBlackBackground,
  'red': printRedBackground,
  'blue': printBlueBackground,
  'green': printGreenBackground,
  'yellow': printYellowBackground
};

if (color in colorObj) {
  colorObj[color]();
}
```

[在此](http://www.nicoespeon.com/en/2015/01/oop-revisited-switch-in-js/)，你可以找到更多这方面的信息。

#5 通过重度字符排序字符串
JavaScript数组本身自带排序方法。当使用`array.sort()`时，会令数组内个元素按照字母表顺序进行排序。对于其他需求，你也可以定制自己的排序方法。

```js
['Shanghai', 'New York', 'Mumbai', 'Buenos Aires'].sort();
// ["Buenos Aires", "Mumbai", "New York", "Shanghai"]
```

但是当你想使用非**ASCII**字符的方式进行排序*['é', 'a', 'ú', 'c']*这些字母，你会发现得到*['c', 'e', 'á', 'ú']*这样非常奇怪的结果。这是由于排序是按照英语的语言方式进行的。
如下例子：

```js
// Spanish
['único','árbol', 'cosas', 'fútbol'].sort();
// ["cosas", "fútbol", "árbol", "único"] // bad order

// German
['Woche', 'wöchentlich', 'wäre', 'Wann'].sort();
// ["Wann", "Woche", "wäre", "wöchentlich"] // bad order
```

幸运的是，ECMASScript的国际化API接口提供了[localeCompare](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/localeCompare)和[Intl.Collator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Collator)两种方法克服这一问题。
##使用`localeCompare()`方法

```js
['único','árbol', 'cosas', 'fútbol'].sort(function (a, b) {
  return a.localeCompare(b);
});
// ["árbol", "cosas", "fútbol", "único"]

['Woche', 'wöchentlich', 'wäre', 'Wann'].sort(function (a, b) {
  return a.localeCompare(b);
});
// ["Wann", "wäre", "Woche", "wöchentlich"]
```

##使用`Intl.Collator()`方法

```js
['único','árbol', 'cosas', 'fútbol'].sort(Intl.Collator().compare);
// ["árbol", "cosas", "fútbol", "único"]

['Woche', 'wöchentlich', 'wäre', 'Wann'].sort(Intl.Collator().compare);
// ["Wann", "wäre", "Woche", "wöchentlich"]
```

* 上述两种方法，你都可以自定义位置。
* 根据*Firefox*浏览器的效率显示，比较大数目的字符串时**Intl.Collator**更快。

因此，当你需要对非英语字符串进行数组排序时，切记使用上述方法进行排序以避免意想不到的排序结果。