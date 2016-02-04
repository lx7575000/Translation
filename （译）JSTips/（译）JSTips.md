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

#6 `undefined` 和 `null`的区别
* `undefined`表示一个变量没有被声明或此时该变量已经被声明但是仍未赋值
* `null`表示一个该变量已经被赋值，但表示**没有值**
* JavaScript默认将没有进行赋值操作的变量赋值为`undefined`
* JavaScript不会**主动**给变量赋值为`null`。程序员用该值赋值`var`变量**表示该变量没有值**
* 在JSON中`undefined`是无效的，然而`null`却可以。
* `undefined`的类型就是**undefined**
* `null`的类型是**object对象**
* 两值都是原始的
* 两值均是[falsy](https://developer.mozilla.org/en-US/docs/Glossary/Falsy)，（即`Bollean(undefined) //false`, `Bollean(null) //false` ）。

**但是，如何判别是否是`undefined`或者`null`呢？下列方法可以帮助你：**

```js
//undefined
typeof variable === 'undefined' 

//null
variable === null
```

**判等判别式认为他们相同，但是当附加类型判别后就不同了**

```js
null == undefined //true
null === undefined // false
```

#7 懒人使用严格模式
JavaScript的严格模式让开发者能够更容易的写出“安全”的代码。
默认情况下，JavaScript允许程序员很粗心。比如，我们可以不使用`var`声明变量就进行使用。尽管对于一些无经验的初学者这么做很方便，**但是**当变量拼写错误或者在作用域外引用变量也会因此导致错误。
开发者通常喜欢将枯燥的工作交给计算机来做，自动检查是否有错误。JavaScript的严格模式命令就是被用来检查JS错误的。

我们可以将该命令添加在各个JS文件首部：

```js
// Whole-script strict mode syntax
"use strict";
var v = "Hi!  I'm a strict mode script!";
```

或者添加到函数当中：

```js
function f()
{
  // Function-level strict mode syntax
  'use strict';
  function nested() { return "And so am I!"; }
  return "Hi!  I'm a strict mode function!  " + nested();
}
function f2() { return "I'm not strict."; }
```

通过包含上述命令在JS文件或函数中，我们就可以直接命令JavaScript引擎执行严格模式（通常在大多数大型JavaScript项目中是关闭的）。其中，严格模式通常会改变以下行为：
  * 变量必须通过`var`声明后才能引用
  * 尝试写入**只读**属性生成产生易被发现的错误
  * 你必须通过`new`关键字执行构造函数
  * **this**关键字不可以隐式绑定到全局对象
  * 只允许有限的使用**eval()**
  * 不允许使用保留字和未来保留字作为变量名

在老项目中引入严格模式或许有些困难，因此新项目中推荐使用严格模式。如果你将所有JS文件引入一个大文件，此时会使所有文件都执行严格模式。
严格模式的命令不是声明，只是字面表达式。通常会被早期JavaScript所忽略（个人感觉，目前没人会用ES5以前的版本了）。以下版本以上的浏览器开始支持严格模式：
* Internet Explorer from version 10.
* Firefox from version 4.
* Chrome from version 13.
* Safari from version 5.1.
* Opera from version 12.

#7 NodeList 转换成 数组
[原文地址](https://github.com/loverajoel/jstips/blob/gh-pages/_posts/en/2016-01-08-converting-a-node-list-to-an-array.md)

当我们使用`querySelectorAll`方法选择DOM节点时，通常会返回一个类似数组的对象**NodeList**。这种数据类型与数组类型很相似，但是却不具备数组自身的一些方法，比如`map`,`forEach`方法。现在我会介绍给你们一种快速、安全且可复用的方法将**NodeList**转回成为**Array**类型。

```js
const nodelist = document.querySelectorAll('div');
const nodelistToArray = Array.apply(null, nodelist);

//later on ..

nodelistToArray.forEach(...);
nodelistToArray.map(...);
nodelistToArray.slice(...);

//etc...
```

通过`apply`方法将以数组形式传入的参数传给函数，添加到其作用域`this`。MDN表示，`apply`会得到`querySelectorAll`返回的类数组对象。因为我们在此函数方法中并不需要特定的作用域`this`指向，因此我们传递`null`或者0作为参数。**最后返回的结果是包含所有`NodeList`中DOM元素的真实数组，这下我们可以使用数组方法了。**

另，如果你使用ES6语法的话，可以直接使用[spread operator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_operator)

```js
const nodelist = [...document.querySelectorAll('div')]; // returns a real array

//later on ..

nodelist.forEach(...);
nodelist.map(...);
nodelist.slice(...);

//etc...
```

#8 `template strings`
自从ES6发布后，JS现在可以使用**模板字符串**替换传统的**引用字符串**了。

```js
//栗子： 普通字符串
var firstName = 'Jake';
var lastName = 'Rawr';
console.log('My name is ' + firstName + ' ' + lastName);
// My name is Jake Rawr

//Template String
var firstName = 'Jake';
var lastName = 'Rawr';
console.log(`My name is ${firstName} ${lastName}`);
// My name is Jake Rawr
```

通过模板字符串，你可以不使用"\n"创建多行字符串，在`${}`中写入简单的逻辑表达式(ie 2+3)。
你也可以通过函数方法更改输出模板字符串：[标签模板字符串](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/template_strings#Tagged_template_strings)

```js
var a = 5;
var b = 10;

function tag(strings, ...values) {
  console.log(strings[0]); // "Hello "
  console.log(strings[1]); // " world "
  console.log(values[0]);  // 15
  console.log(values[1]);  // 50

  return "Bazinga!";
}

tag`Hello ${ a + b } world ${ a * b}`;
// "Bazinga!"
```

[更多相关资料，请点击here](https://hacks.mozilla.org/2015/05/es6-in-depth-template-strings-2)

#9 判别某值是否在对象当中
当你需要去判断某一对象中是否有某一值时，你很可能是如下做法：

```js
var myObject = {
  name: '@tips_js'
};

if (myObject.name) { ... }
```

这样做虽然没错，但是你需要知道两种更原生的方法可以令你做这件事更容易:`in`操作符和`Object.hasOwnProperty`。每个继承自`Object`的对象都可以使用。
## 不同之处

```js
var myObject = {
  name: '@tips_js'
};

myObject.hasOwnProperty('name'); // true
'name' in myObject; // true

myObject.hasOwnProperty('valueOf'); // false, valueOf is inherited from the prototype chain
'valueOf' in myObject; // true
```

上述两种方法在检查属性时各有不同。换句话说，`hasOwnProperty`会在当前对象**本身**具有该属性时返回true。然而，`in`操作符不会区别属性是属于当前对象本身还是继承自其原型链。

**下面有个栗子**

```js
var myFunc = function() {
  this.name = '@tips_js';
};
myFunc.prototype.age = '10 days';

var user = new myFunc();

user.hasOwnProperty('name'); // true
user.hasOwnProperty('age'); // false, because age is from the prototype chain
```

[实时演示例子](https://jsbin.com/tecoqa/edit?js,console)
推荐你阅读这个关于判别对象属性常见错误的[讨论](https://github.com/loverajoel/jstips/issues/62)

#9 提升(hoisting)

[原文位置](https://github.com/loverajoel/jstips/blob/gh-pages/_posts/en/2016-01-11-hoisting.md)

理解[提升](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/var#var_hoisting)能够更好的帮助你组织函数的作用域。切记，在最开始声明变量和定义函数。变量定义和声明并不是一回事，尽管你能将它们同时执行在同一行中。声明会令系统知道变量的存在，而定义仅仅只是赋值操作而已。

```js
function doTheThing() {
  // ReferenceError: notDeclared is not defined
  console.log(notDeclared);

  // Outputs: undefined
  console.log(definedLater);
  var definedLater;

  definedLater = 'I am defined!'
  // Outputs: 'I am defined!'
  console.log(definedLater)

  // Outputs: undefined
  console.log(definedSimulateneously);
  var definedSimulateneously = 'I am defined!'
  // Outputs: 'I am defined!'
  console.log(definedSimulateneously)

  // Outputs: 'I did it!'
  doSomethingElse();

  function doSomethingElse(){
    console.log('I did it!');
  }

  // TypeError: undefined is not a function
  functionVar();

  var functionVar = function(){
    console.log('I did it!');
  }
}
```

为了令代码阅读清楚些，在函数方法的最开始声明所有需要用到的变量方法，这样可以更清楚的了解各个变量的作用域。声明变量在应用之前，定义函数方法在最后，令其更清晰。

#10  ES6函数中的默认参数
[原文地址](https://github.com/loverajoel/jstips/blob/gh-pages/_posts/en/2016-01-12-pseudomandatory-parameters-in-es6-functions.md)

许多编程语言中函数方法的参数在默认情况下是强制设定的，程序员必须**显示定义**参数为可选。在JS中函数参数是可选的，但是我们可以利用**ES6的默认参数值**特性，不用在函数体内做一系列复杂操作。

```js
const _err = function( message ){
  throw new Error( message );
}

const getSum = (a = _err('a is not defined'), b = _err('b is not defined')) => a + b

getSum( 10 ) // throws Error, b is not defined
getSum( undefined, 10 ) // throws Error, a is not defined
```

`_err`函数会在`getSum`函数方法没有参数值传入时立即抛出异常。当然，你可以在[MDN](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Functions/default_parameters)中看到关于默认参数值特性的例子。

# 11 测量JS执行效率的小技巧
[原文地址](https://github.com/loverajoel/jstips/blob/gh-pages/_posts/en/2016-01-13-tip-to-measure-performance-of-a-javascript-block.md)

为了快速衡量JavaScript的性能，我们可以使用`console`方法比如`console.time(label)`和`console.timeEnd(label)`

```js
console.time("Array initialize");
var arr = new Array(100),
    len = arr.length,
    i;

for (i = 0; i < len; i++) {
    arr[i] = new Object();
};
console.timeEnd("Array initialize"); // Outputs: Array initialize: 0.711ms
```

更多信息： [Console object](https://github.com/DeveloperToolsWG/console-object), [JavaScript benchmarking](https://mathiasbynens.be/notes/javascript-benchmarking)
Demo: [jsfiddle](https://jsfiddle.net/meottb62/)-[codepen](http://codepen.io/anon/pen/JGJPoa)(在浏览器中console)

