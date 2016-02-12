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

#8 NodeList 转换成 数组
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

#9 `template strings`
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

#10 判别某值是否在对象当中
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

#11 提升(hoisting)

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

#12  ES6函数中的默认参数
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

# 13 测量JS执行效率的小技巧
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

#14 胖箭头函数
作为ES6的新属性，使用胖箭头函数可以更方便的写出更简洁的代码。这个名字来源于其语法中的与`->`（瘦）相比的**`=>`（胖）**表达式。一些程序员可能已经在**Haskell**这类编程语言中了解了这种类型的**匿名函数**或**lambda表达式**。
##箭头函数好处在哪？
* 语法： 更少的代码行，不需要一遍遍的写`function`关键字
* 语义： 自动绑定上层作用域`this`关键字。避免函数内莫名绑定到`window`。

##简单小例子
看看下面执行功能相同的代码片段，相信你会很快理解箭头函数的用法的。

```js
// general syntax for fat arrow functions
// 正常箭头函数语法
param => expression

// may also be written with parentheses
// parentheses are required on multiple params
// 多函数的箭头函数
(param1 [, param2]) => expression


// using functions
// ES5
var arr = [5,3,2,9,1];
var arrFunc = arr.map(function(x) {
  return x * x;
});
console.log(arr)

// using fat arrow
// ES6
var arr = [5,3,2,9,1];
var arrFunc = arr.map((x) => x*x);
console.log(arr)
```

## `this`绑定
我们使用箭头函数的另一理由是其可以对`this`上下文关系的处理。在箭头函数中，我们不再需要担心`.bind(this)`或设定`that=this`。箭头函数会自动绑定上级词法作用域的`this`。

```js
// globally defined this.i
this.i = 100;

var counterA = new CounterA();
var counterB = new CounterB();
var counterC = new CounterC();
var counterD = new CounterD();

// bad example
function CounterA() {
  // CounterA's `this` instance (!! gets ignored here)
  this.i = 0;
  setInterval(function () {
    // `this` refers to global object, not to CounterA's `this`
    // therefore starts counting with 100, not with 0 (local this.i)
    this.i++;
    document.getElementById("counterA").innerHTML = this.i;
  }, 500);
}

// manually binding that = this
function CounterB() {
  this.i = 0;
  var that = this;
  setInterval(function() {
    that.i++;
    document.getElementById("counterB").innerHTML = that.i;
  }, 500);
}

// using .bind(this)
function CounterC() {
  this.i = 0;
  setInterval(function() {
    this.i++;
    document.getElementById("counterC").innerHTML = this.i;
  }.bind(this), 500);
}

// fat arrow function
function CounterD() {
  this.i = 0;
  setInterval(() => {
    this.i++;
    document.getElementById("counterD").innerHTML = this.i;
  }, 500);
}
```

可以去[MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions)了解更多箭头函数的内容，不同的语法选择可以来[这里](http://jsrocks.org/2014/10/arrow-functions-and-their-scope/)

#15 `indexOf`用作包含条款最佳实践
JavaScript本身默认没有判断**字符串或数组**是否包含子串及元素的方法。通常我们会这么做：

```js
var someText = 'javascript rules';
if (someText.indexOf('javascript') !== -1) {
}

// or
if (someText.indexOf('javascript') >= 0) {
}
```

让我们看看[express.js](https://github.com/strongloop/express)的语法段。
[examples/mvc/lib/boot.js](https://github.com/strongloop/express/blob/2f8ac6726fa20ab5b4a05c112c886752868ac8ce/examples/mvc/lib/boot.js#L26)

```js
for (var key in obj) {
  // "reserved" exports
  if (~['name', 'prefix', 'engine', 'before'].indexOf(key)) continue;
```

[lib/utils.js](https://github.com/strongloop/express/blob/2f8ac6726fa20ab5b4a05c112c886752868ac8ce/lib/utils.js#L93)

```js
exports.normalizeType = function(type){
  return ~type.indexOf('/')
    ? acceptParams(type)
    : { value: mime.lookup(type), params: {} };
};
```

[examples/web-service/index.js](https://github.com/strongloop/express/blob/2f8ac6726fa20ab5b4a05c112c886752868ac8ce/examples/web-service/index.js#L35)

```js
// key is invalid
if (!~apiKeys.indexOf(key)) return next(error(401, 'invalid api key'));
```

相信你也注意到了，重点在于`~`操作符。按位操作符以二进制形式执行操作，但是返回标准JavaScript数据值。
它会将将**-1**转化成**0**，并且**0**在JavaScript中表示**`false`**.

```js
var someText = 'text';
!!~someText.indexOf('tex'); // someText contains "tex" - true
!~someText.indexOf('tex'); // someText NOT contains "tex" - false
~someText.indexOf('asd'); // someText doesn't contain "asd" - false
~someText.indexOf('ext'); // someText contains "ext" - true
```

##`String.prototype.includes()`
ES6引入[includes()方法](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/includes)你可以使用其判断字符串是否包含某个子串。

```js
'something'.includes('thing'); // true
```

在ES7中，你甚至可以将其应用到数组类型中

```js
//ES7
!!~[1, 2, 3].indexOf(1); // true
[1, 2, 3].includes(1); // true
```

很不幸，只用**Chrome, Firefox, Safari 9+及Edge**支持该语法，**IE11以下版本不支持**。推荐在可控环境下使用该方法。

#16 给回调函数传参
默认情况下你没有办法给回调函数传递参数，比如：

```js
function callback() {
  console.log('Hi human');
}

document.getElementById('someelem').addEventListener('click', callback);
```

可以利用JavaScript闭包作用域传递参数给回调函数。

```js
function callback(a, b) {
  return function() {
    console.log('sum = ', (a+b));
  }
}
var x = 1, y = 2;
document.getElementById('someelem').addEventListener('click', callback(x, y));
```

## 闭包是什么？
闭包是指可以包含独立(自由)变量的函数。换句话说，定义在闭包中的函数’记住‘创建了它的上层作用域。去[MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures)了解更多。
所以，通过闭包我们可以在回调函数被调用时将参数x和y传递给它。
另外也可以通过`bind`方法传递给它。

```js
var alertText = function(text) {
  alert(text);
};

document.getElementById('someelem').addEventListener('click', alertText.bind(this, 'hello'));
```

上述两种方法的效率仅仅有稍许区别，可以通过[jsperf](http://jsperf.com/bind-vs-closure-23)查看。

#17 NodeJS 执行未`require`的模块
根据你的代码是运行在`require('./something.js')`还是`node something.js`，你可以要求程序执行两种不同情况。如果你想与你的模块独立交互，这么做是非常有用的。

```js
if (!module.parent) {
    // ran with `node something.js`
    app.listen(8088, function() {
        console.log('app listening on port 8088');
    })
} else {
    // used with `require('/.something.js')`
    module.exports = app;
}
```

更多关于[模块](https://nodejs.org/api/modules.html#modules_module_parent)信息

#18 舍入的快速方法
今天的建议是关于效率的。[Ever came across the double tilde](http://stackoverflow.com/questions/5971645/what-is-the-double-tilde-operator-in-javascript)`~~` 操作符？同时也被称为双非操作符。你可以用其替换`Math.floor()`。
一位转变`~`会将32位二进制数转换为`-(input+1)`。因此双非转变会将原数据变为`-(-(input+1)+1)`，使用其进行向下取整非常方便。对输入的数值使用`~~`操作符，模仿使用`math.ceil()`对负数取整，`Math.floor()`对正数取整。当失败返回0，与之相比`Math.floor()`方法失败返回的`Nan`简直非人类。

```js
// single ~
console.log(~1337)    // -1338

// numeric input
console.log(~~47.11)  // -> 47
console.log(~~-12.88) // -> -12
console.log(~~1.9999) // -> 1
console.log(~~3)      // -> 3

// on failure
console.log(~~[]) // -> 0
console.log(~~NaN)  // -> 0
console.log(~~null) // -> 0

// greater than 32 bit integer fails
console.log(~~(2147483647 + 1) === (2147483647 + 1)) // -> 0
```

尽管，`~~`可能效率更高，但是为了可读性更好最好还是使用`Math.floor()`

#19 字符串安全连接
你可能希望将多种不同类型的变量连接成字符串。可以肯定的是，应用算术操作符不一定都适用，因此推荐使用`concat`方法。

```js
var one = 1;
var two = 2;
var three = '3';

var result = ''.concat(one, two, three); //"123"
```

应用`concat`可以准确达成你的需求，相比使用`+`操作符可能会产生预料之外的结果：

```js
var one = 1;
var two = 2;
var three = '3';

var result = one + two + three; //"33" instead of "123"
```

关于效率问题，`join`方法和`concat`效率基本一致。

#20 通过返回对象以实现函数链
在面向对象的JavaScript中创建函数，函数内返回对象使你可以类似**JQuery**的函数链。

```js
function Person(name) {
  this.name = name;

  this.sayName = function() {
    console.log("Hello my name is: ", this.name);
    return this;
  };

  this.changeName = function(name) {
    this.name = name;
    return this;
  };
}

var person = new Person("John");
person.sayName().changeName("Timmy").sayName();
```

#21 数组重组
这个代码段使用了[Fisher-Yates Shuffling](https://www.wikiwand.com/en/Fisher%E2%80%93Yates_shuffle)算法重组已有数组。

```js
function shuffle(arr) {
    var i,
        j,
        temp;
    for (i = arr.length - 1; i > 0; i--) {
        j = Math.floor(Math.random() * (i + 1));
        temp = arr[i];
        arr[i] = arr[j];
        arr[j] = temp;
    }
    return arr;    
};
//例子
var a = [1, 2, 3, 4, 5, 6, 7, 8];
var b = shuffle(a);
console.log(b);
// [2, 7, 8, 6, 5, 3, 1, 4]
```

#22 清空数组
通常你定义数组并且希望内容清空，你可能会这么做：

```js
// define Array
var list = [1, 2, 3, 4];
function empty() {
    //empty your array
    list = [];
}
empty();
```

但是，这里有更高效的方法：

```js
var list = [1, 2, 3, 4];
function empty() {
    //empty your array
    list.length = 0;
}
empty();
```

* `list = []`通过创建新数组进行赋值，然而这么做却不会对原数组内容产生影响。原数组内容仍存储在内存当中，可能会导致内存泄露。
* `list.length = 0`删除数组内内容，这样会对其引用同时产生影响。

换句话说，如果对同一数组进行了两次引用(`a=[1, 2, 3], a2 = a`)，并且你通过`list.length = 0`进行内容删除，两个引用(a 和 a2)都会为空。(因此，如果不希望产生影响，就别用上述技巧)。

想想，下例会输出什么：

```js
var foo = [1,2,3];
var bar = [1,2,3];
var foo2 = foo;
var bar2 = bar;
foo = [];
bar.length = 0;
console.log(foo, bar, foo2, bar2);

// [] [] [1, 2, 3] []
```

更多细节[difference-between-array-length-0-and-array](http://stackoverflow.com/questions/4804235/difference-between-array-length-0-and-array)

#23 字符串 => 数值
将**字符串类型**的数字变为**数值**很常见，最简单且迅速的方法建议使用**`+`**操作符。

```js
var one = '1';
var numberOne = +one // Number 1
```

当然，**`-`** 操作符也行，但是这么做会令数值为**负**

```js
var one = '1';
var negativeNumberOne = -one; // Number -1
```

#24 使用`===`代替`==`
**`==`(或`!=`)操作符**会在需要的情况下**自动进行类型转换**。
**`===`(或`!==`)操作符**不会进行类型转换。它只会比较值和类型，因此相对`==`操作符会执行更快。

```js
[10] ==  10      // is true
[10] === 10      // is false

'10' ==  10      // is true
'10' === 10      // is false

 []  ==  0       // is true
 []  === 0       // is false

 ''  ==  false   // is true but true == "a" is false
 ''  === false   // is false 
```

#25 使用立即执行函数表达式
立即执行函数表达式是缩写为IIFE(immediately invoked function expression)的匿名函数，它在JavaScript中有一些重要的作用。

```js
(function() {
 // Do something
 }
)()
```

包裹在圆括号中的匿名函数，将匿名函数编程一个函数表达式或变量表达式，而不是一个在全局（或任意）作用域的简单匿名函数。因此我们现在有一个未命名的函数表达式。
当然，我们也可以为其命名。

```js
(someNamedFunction = function(msg) {
    console.log(msg || "Nothing for today !!")
    }) (); // Output --> Nothing for today !!

someNamedFunction("Javascript rocks !!"); // Output --> Javascript rocks !!
someNamedFunction(); // Output --> Nothing for today !!
```

预知更多详细情节：URL‘s 1. [Link 1](https://blog.mariusschulz.com/2016/01/13/disassembling-javascripts-iife-syntax) 2. [Link 2](http://javascriptissexy.com/12-simple-yet-powerful-javascript-tips/)

#  26 过滤、排序字符串表
你可能有一个一大长串名单需要过滤重复元素，同时需要按照字母表排列。

在下面例子中给出了一个JavaScript**保留字**列表，但是你会发现其中有许多重复单词，并且未按序排列。因此，我们使用下列数组测试我们的小技巧。

```js
var keywords = ['do', 'if', 'in', 'for', 'new', 'try', 'var', 'case', 'else', 'enum', 'null', 'this', 'true', 'void', 'with', 'break', 'catch', 'class', 'const', 'false', 'super', 'throw', 'while', 'delete', 'export', 'import', 'return', 'switch', 'typeof', 'default', 'extends', 'finally', 'continue', 'debugger', 'function', 'do', 'if', 'in', 'for', 'int', 'new', 'try', 'var', 'byte', 'case', 'char', 'else', 'enum', 'goto', 'long', 'null', 'this', 'true', 'void', 'with', 'break', 'catch', 'class', 'const', 'false', 'final', 'float', 'short', 'super', 'throw', 'while', 'delete', 'double', 'export', 'import', 'native', 'public', 'return', 'static', 'switch', 'throws', 'typeof', 'boolean', 'default', 'extends', 'finally', 'package', 'private', 'abstract', 'continue', 'debugger', 'function', 'volatile', 'interface', 'protected', 'transient', 'implements', 'instanceof', 'synchronized', 'do', 'if', 'in', 'for', 'let', 'new', 'try', 'var', 'case', 'else', 'enum', 'eval', 'null', 'this', 'true', 'void', 'with', 'break', 'catch', 'class', 'const', 'false', 'super', 'throw', 'while', 'yield', 'delete', 'export', 'import', 'public', 'return', 'static', 'switch', 'typeof', 'default', 'extends', 'finally', 'package', 'private', 'continue', 'debugger', 'function', 'arguments', 'interface', 'protected', 'implements', 'instanceof', 'do', 'if', 'in', 'for', 'let', 'new', 'try', 'var', 'case', 'else', 'enum', 'eval', 'null', 'this', 'true', 'void', 'with', 'await', 'break', 'catch', 'class', 'const', 'false', 'super', 'throw', 'while', 'yield', 'delete', 'export', 'import', 'public', 'return', 'static', 'switch', 'typeof', 'default', 'extends', 'finally', 'package', 'private', 'continue', 'debugger', 'function', 'arguments', 'interface', 'protected', 'implements', 'instanceof'];
```

因为我们不希望更改原生列表，所以我们准备使用`filter`方法，它将会基于我们的传入的**过滤方法**返回过滤后数组。这个比较方法会比较原生数组当前位置的保留字与新生成数组索引，并且会在符合要求情况下添加进新数组中。

最后，我们会排序过滤后数组使用`sort`方法，我们会为其传入比较方法，要求其返回按字母序排列的数组。

```js
var filteredAndSortedKeywords = keywords
  .filter(function (keyword, index) {
      return keywords.lastIndexOf(keyword) === index;
    })
  .sort(function (a, b) {
      return a < b ? -1 : 1;
    });
```

ES6版本，我们使用**箭头函数**来传递比较方法：

```js
const filteredAndSortedKeywords = keywords
  .filter((keyword, index) => keywords.lastIndexOf(keyword) === index)
  .sort((a, b) => a < b ? -1 : 1);
```

最后，我们可以得到去重且重排序的JavaScript保留字数组。

# 27 短路求值
[短路求值](https://zh.wikipedia.org/wiki/%E7%9F%AD%E8%B7%AF%E6%B1%82%E5%80%BC)的第二个参数被执行或者返回值，必须要看第一个参数值是否满足表达式要求。

* 在AND(&&)方法中当第一个参数为假时，则整个返回值必为**假**。
* 在OR(||)方法中，当第一个参数为真时，则返回值必为**真**。

```js
var test = true;
var isTrue = function(){
  console.log('Test is true.');
};
var isFalse = function(){
  console.log('Test is false.');
};

//Using logical AND - &&.
// A normal if statement.
if(test){
  isTrue();    // Test is true
}

// Above can be done using '&&' as -

( test && isTrue() );  // Test is true


//Using logical OR - ||.

test = false;
if(!test){
  isFalse();    // Test is false.
}

( test || isFalse());  // Test is false.
```

逻辑或方法，也可以被用来为函数设默认值

```js
function theSameOldFoo(name){ 
    name = name || 'Bar' ;
    console.log("My best friend's name is " + name);
}
theSameOldFoo();  // My best friend's name is Bar
theSameOldFoo('Bhaskar');  // My best friend's name is Bhaskar
```

逻辑与方法，可以被用来避免属性值未定义的情况：

```js
var dog = { 
  bark: function(){
     console.log('Woof Woof');
   }
};

// Calling dog.bark();
dog.bark(); // Woof Woof.

//But if dog is not defined, dog.bark() will raise an error "Cannot read property 'bark' of undefined."
// To prevent this, we can you &&.

dog&&dog.bark();   // This will only call dog.bark(), if dog is defined.
```

# 28 科里化 VS 部分应用程序
##科里化
科里化会将一个函数传入另一个函数中。

```js
f  :  X * Y -> R
f' :  X -> ( Y -> R)
```

与第一个函数`f`传入两个参数不同，我们给`f'`传递一个参数。其返回结果是一个函数方法，我们会将第二个参数传递给它，然后求得结果。
所以，我们调用未科里化的`f`方法，以下例形式：

```js
f(2, 3);
```

应用科里化的`f'`函数以下述形式：

```js
f’(2)(3);
```

下面给出具体例子：

```js
//未科里化
function add(x, y) {
  return x + y;
}

add(3, 5);   // returns 8

//科里化
function addC(x) {
  return function (y) {
    return x + y;
  }
}

addC(3)(5);   // returns 8
```

###科里化算法
科里化是一个二元函数，其返回一个函数，然后该函数再返回一个函数。
`curry : (X * Y -> R) -> (X -> (Y -> R)) `

```js
function curry(f) {
  return function(x) {
    return function(y) {
      return f(x, y);
    }
  }
}
```

##局部应用
局部应用传入一个函数
`f: X * Y -> R`
并且设定固定值与第一个参数计算，产生新函数。
`f' Y -> R`
**f'**方法与**f**方法作用相同，只是将第二个参数放入返回的第二个方法中传入。
例如，下面例子将第一个参数传入新方法，并设定固定值5与其相加：

```js
function plus5(y) {
  return 5 + y;
}

plus5(3);  // returns 8
```

###部分应用算法
partApply方法传入一个二元函数方法和一个参数，返回一个一元函数。
`((X * Y -> R)) -> (X * (Y -> R))`

```js
function partApply(f, x) {
  return function(y) {
    return f(x, y);
  }
}
```

#29 通过记忆数据加快递归函数 
大家对斐波那契数列都很熟悉了，我们看以在20秒就写出下列函数

```js
var fibonacci = function(n){
    return n < 2 ? n : fibonacci(n-1) + fibonacci(n-2);
}
```

它很有用，但是因为做了过多重复计算工作所以没效率。我们可以通过存下其之前的计算结果来增速计算：

```js
var fibonacci = (function(){
    var cache = {
        0: 0,
        1: 1
    };
    return function self(n){
        return n in cache ? cache[n] : (cache[n] = self(n-1) + self(n-2));
    }
})()
```

我们也可以定义一个更高级接收函数作为参数的方法，并且返回之前存储下的该方法。

```js
//ES5

var memoize = function(func){
    var cache = {};
    return function(){
        var key = Array.prototype.slice.call(arguments).toString();
        return key in cache ? cache[key] : (cache[key] = func.apply(this,arguments));
    }
}
fibonacci = memoize(fibonacci);

//ES6

var memoize = function(func){
    const cache = {};
    return (...args) => {
        const key = [...args].toString();
        return key in cache ? cache[key] : (cache[key] = func(...args));
    }
}
fibonacci = memoize(fibonacci);
```

我们可以在多种情况下使用`memoize()`方法。
* GCD（最大公约数）
* 阶乘计算

```js
//Greatest Common Divisor
var gcd = memoize(function(a,b){
    var t;
    if (a < b) t=b, b=a, a=t;
    while(b != 0) t=b, b = a%b, a=t;
    return a;
})
gcd(27,183); //=> 3

//Factorial calculation
var factorial = memoize(function(n) {
    return (n <= 1) ? 1 : n * factorial(n-1);
})
factorial(5); //=> 120
```

# 30 转换真假值为布尔型
你可以转换真、假值为**真**布尔型，通过`!!`操作符。

```js
!!"" // false
!!0 // false
!!null // false
!!undefined // false
!!NaN // false

!!"hello" // true
!!1 // true
!!{} // true
!![] // true
```

# 31 避免更改或传递参数到`arguments`-非最优化
##背景
在JavaScript中，`arguments`变量可以获取传递给函数方法所有的参数。`arguments`是一个类数组对象，可以使用数组的操作符进行操作，同时它也有`length`属性。但是它没有数组内置方法，例如`filter`、`map`和`forEach`。因此，常见方法是将`arguments`转变为数组类型：}

```js
var args = Array.prototype.slice.call(arguments);
```

通过调用数组的`slice`方法传递`arguments`的值。但是，`slice`方法只是返回`arguments`的浅拷贝到新建数组。简写方法如下：

```js
var args = [].slice.call(arguments);
```

通过简单的数组符号调用`slice`方法即可。
## 优化
在函数内使用`arguments`都会导致Chrome的**V8**引擎和**Node**跳过函数优化部分，因此导致执行效率较差。可以阅读[优化杀手](https://github.com/petkaantonov/bluebird/wiki/Optimization-killers)。传递`arguments`给任意函数都会导致`arguments`泄露。
当然，如果想要`arguments`的数组，允许你是用这个方法：

```js
var args = new Array(arguments.length);
for(var i = 0; i < args.length; ++i) {
  args[i] = arguments[i];
}
```

恩，这样虽然很啰嗦，但是生产环境代码效率高于一切。

# 32 使用Map解决对象属性无序问题
## 对象属性顺序
> 一个对象是类型对象的一个成员。它是无序属性集合，每个属性都包含一个初始值、对象或方法。存储在对象内的函数被称为方法。

看下面例子，每个浏览器都有其自己的排序规则，因此对象的顺序没法固定。

```js
var myObject = {
    z: 1,
    '@': 2,
    b: 3,
    1: 4,
    5: 5
};
console.log(myObject) // Object {1: 4, 5: 5, z: 1, @: 2, b: 3}

for (item in myObject) {...
// 1
// 5
// z
// @
// b
```

## 如何解决？

### Map

我们可以通过使用ES6的新特性—`Map`。Map对象根据其添加顺序进行排序。使用`for ... of`循环方法返回包含每个元素的**键值对**数组

```js
var myObject = new Map();
myObject.set('z', 1);
myObject.set('@', 2);
myObject.set('b', 3);
for (var [key, value] of myObject) {
  console.log(key, value);
...
// z 1
// @ 2
// b 3
```

###老版本浏览器解决办法
**Mozilla 建议**
> 如果想要跨平台并且有序的将执行对象内的值或方法，可以通过使用两数组，分别按顺序存储关键字及相对应的值的方式来达成。

```js
// Using two separate arrays
var objectKeys = [z, @, b, 1, 5];
for (item in objectKeys) {
    myObject[item]
...

// Build an array of single-property objects
var myData = [{z: 1}, {'@': 2}, {b: 3}, {1: 4}, {5: 5}];
```

#33 产生从0到N-1位数的最简函数
通过下面一行代码，我们可以创建从0到N-1范围的数字组成的数组

```js
Array.apply(null, {length: N}).map(Number.call, Number);
```

让我们详细分析该代码，在JavaScript中，我们通过`call()`方法调用函数，传入的第一个参数是用于说明函数作用域，后续参数则为传入该函数作用域的参数值。

```js
function add(a, b){
    return (a+b);
}
add.call(null, 5, 6);
```

上例函数会返回5，6的和。
JavaScript的数组的`map()`方法，要求传入两个参数。第一个参数为回调函数，第二个参数则为作用域。**回调函数**要求传入三个参数,`值`、`索引值`和我们想要操作的`数组`。所以，下面为示例语法：

```js
[1, 2, 3].map(function(value, index, arr){
    //Code
}, this);
```

下面代码我们创建大小为N的数组：

```js
Array.apply(null, {length: N})
```

下面的方法将所有元素传入创建的新数组：

```js
Array.apply(null, {length: N}).map(Number.call, Number);
```

如果，你希望为从1开始到N的所组成的数组：

```js
Array.apply(null, {length: N}).map(function(value, index){
  return index+1;  
});
```



# 34 实现异步循环

让我们尝试实现一个每秒输出一个递增值的异步函数：

```js
for (var i=0; i<5; i++) {
    setTimeout(function(){
        console.log(i); 
    }, 1000);
}  
```

但是，结果很蛋疼，没按照要求工作

```js
> 5
> 5
> 5
> 5
> 5
```

**原因：**
每秒调用的不是i的拷贝，是i值的引用。因此 **`i`** 循环递增,函数执行获取到i值为5。
这个问题看起来解决很容易，通过将i每次期望执行到的值存储到一个临时变量中。

```js
for (var i=0; i<5; i++) {
    var temp = i;
    setTimeout(function(){
        console.log(temp); 
    }, 1000);
}  
```

**但是，结果依然不对**

```js
> 4
> 4
> 4
> 4
> 4
```

**解决办法**
换个方法复制i值。最常见方法是通过声明函数创建闭包并将i值作为参数传递，下面给出自调用方法

```js
for (var i=0; i<5; i++) {
    (function(num){
        setTimeout(function(){
            console.log(num); 
        }, 1000); 
    })(i);  
}  
```

在JavaScript中，函数参数仅仅是传值。因此，原生类型都是传递值拷贝。在函数内对参数进行更改不会影响到外部作用域。**对象类型特殊些，对其内容更改会在所有作用域产生影响。**

另一个解决办法是通过使用`let`，使用ES6的`let`关键字，因为它不同于`var`，它会限定作用域。

```js
for (let i=0; i<5; i++) {
    var temp = i;
    setTimeout(function(){
        console.log(temp); 
    }, 1000);
}  
```

# 35 赋值操作符
赋值操作非常常见，有时对于我们这些懒惰的程序员来说打字变得耗时了。因此我们会使用一些小技巧来使我们的代码简洁。

```js
x += 23; // x = x + 23;
y -= 15; // y = y - 15;
z *= 10; // z = z * 10;
k /= 7; // k = k / 7;
p %= 3; // p = p % 3;
d **= 2; // d = d ** 2;
m >>= 2; // m = m >> 2;
n <<= 2; // n = n << 2;
n ++; // n = n + 1;
n --; n = n - 1;
```

## `If-else`使用三元操作符
常见写法：

```js
var newValue;
if(value > 10) 
  newValue = 5;
else
  newValue = 2;
```

使用三元操作符：

```js
var newValue = (value > 10) ? 5 : 2;
```

## Null, Undefined, Empty 检测

```js
//常见写法
if (variable1 !== null || variable1 !== undefined || variable1 !== '') {
     var variable2 = variable1;
}

//缩写
var variable2 = variable1  || '';
```

## 对象数组表示法

```js
//Common 
var a = new Array();
a[0] = "myString1";
a[1] = "myString2";

//缩写
var a = ["myString1", "myString2"];
```

## 关联数组

```js
// Common 
var skillSet = new Array();
skillSet['Document language'] = 'HTML5';
skillSet['Styling language'] = 'CSS3';

//简写
var skillSet = {
    'Document language' : 'HTML5', 
    'Styling language' : 'CSS3'
};
```

## 36 观测DOM的扩展变化
**翻译的很差，后面会重新对其进行翻译**
[MutationObserver](https://developer.mozilla.org/en/docs/Web/API/MutationObserver)可以监听DOM变化，并且可以在他们状态变化时对其进行操作。在下例中有当第一个目标元素创建完成，准备进行创建后面一个元素时。定时器动态加载内容。扩展代码中，首先，根观察者工作直到目标元素产生，然后元素观察者开始工作。这样级联的观察帮助最后发现子目标元素的发现。这样的方式，对于扩展开发复杂网站动态加载内容很有用。

```js
const observeConfig = {
    attributes: true,
    childList: true,
    characterData: true,
    subtree: true
};

function initExtension(rootElement, targetSelector, subTargetSelector) {
    var rootObserver = new MutationObserver(function(mutations) {
        console.log("Inside root observer");
        targetElement = rootElement.querySelector(targetSelector);
        if (targetElement) {
            rootObserver.disconnect();
            var elementObserver = new MutationObserver(function(mutations) {
                console.log("Inside element observer")
                subTargetElement = targetElement.querySelector(subTargetSelector);
                if (subTargetElement) {
                    elementObserver.disconnect();
                    console.log("subTargetElement found!")
                }
            })
            elementObserver.observe(targetElement, observeConfig);
        }
    })
    rootObserver.observe(rootElement, observeConfig);
}

(function() {

    initExtension(document.body, "div.target", "div.subtarget")

    setTimeout(function() {
        del = document.createElement("div");
        del.innerHTML = "<div class='target'>target</div>"
        document.body.appendChild(del)
    }, 3000);


    setTimeout(function() {
        var el = document.body.querySelector('div.target')
        if (el) {
            del = document.createElement("div");
            del.innerHTML = "<div class='subtarget'>subtarget</div>"
            el.appendChild(del)
        }
    }, 5000);

})()
```

# 37数组的删除处理
##原生类型
如果数组内仅仅包括原生数据类型，我们可以通过`filter`和`indexOf`方法对其进行删除处理。

```js
var deduped = [ 1, 1, 'a', 'a' ].filter(function (el, i, arr) {
    return arr.indexOf(el) === i;
});

console.log(deduped); // [ 1, 'a' ]
```

## ES2015
可以通过使用箭头函数将该函数写的更加紧凑小巧。

```js
var deduped = [ 1, 1, 'a', 'a' ].filter( (el, i, arr) => arr.indexOf(el) === i);

console.log(deduped); // [ 1, 'a' ]
```

但是，假如引进使用`Sets`和`from`方法，我们实现该方法更加方便

```js
var deduped = Array.from( new Set([ 1, 1, 'a', 'a' ]) );

console.log(deduped); // [ 1, 'a' ]
```

## 对象
当元素是对象时，我们没法使用上述方法，这是因为对象存储是基于引用而其他元素则是基于值拷贝存储。

```js
1 === 1 // true

'a' === 'a' // true

{ a: 1 } === { a: 1 } // false
```

因此，我们通过使用哈希表来完成同样目的

```js
function dedup(arr) {
    var hashTable = {};

    return arr.filter(function (el) {
        var key = JSON.stringify(el);
        var match = Boolean(hashTable[key]);

        return (match ? false : hashTable[key] = true);
    });
}

var deduped = dedup([
    { a: 1 },
    { a: 1 },
    [ 1, 2 ],
    [ 1, 2 ]
]);

console.log(deduped); // [ {a: 1}, [1, 2] ]
```

在JavaScript中，哈希表是简单的对象。`key`关键字一直是`string`类型。这意味着我们无法分辨数值和字符串的相同值，比如1和‘1’.

```js
var hashTable = {};

hashTable[1] = true;
hashTable['1'] = true;

console.log(hashTable); // { '1': true }
```

但是，由于我们使用了`JSON.stringify`。关键字为`string`类型，因此存储形式为转移字符串值，在哈希表中转化为独一无二的键值。

```js
var hashTable = {};

hashTable[JSON.stringify(1)] = true;
hashTable[JSON.stringify('1')] = true;

console.log(hashTable); // { '1': true, '\'1\'': true }
```

这意味着重复的元素相同的值，但是类型不同。仍将被相同的方法删除处理。

```js
var deduped = dedup([
    { a: 1 },
    { a: 1 },
    [ 1, 2 ],
    [ 1, 2 ],
    1,
    1,
    '1',
    '1'
]);

console.log(deduped); // [ {a: 1}, [1, 2], 1, '1' ]
```

#38 多维数组 -> 单数组
有三种常用的办法将多维数组合并为单数组。
**样例数组：**

```js
var myArray = [[1, 2],[3, 4, 5], [6, 7, 8, 9]];

//转变为

[1, 2, 3, 4, 5, 6, 7, 8, 9]
```

##方法1 ： 使用`concat()` 和`apply()`

```js
var myNewArray = [].concat.apply([], myArray);
// [1, 2, 3, 4, 5, 6, 7, 8, 9]
```

##方法2： 使用`reduce()`方法

```js
var myNewArray = myArray.reduce(function(prev, curr) {
  return prev.concat(curr);
});
// [1, 2, 3, 4, 5, 6, 7, 8, 9]
```

##方法3： 

```js
var myNewArray3 = [];
for (var i = 0; i < myArray.length; ++i) {
  for (var j = 0; j < myArray[i].length; ++j)
    myNewArray3.push(myArray[i][j]);
}
console.log(myNewArray3);
// [1, 2, 3, 4, 5, 6, 7, 8, 9]
```

[这里](https://jsbin.com/qeqicu/edit?js,console)有上述三个方法的应用。
对无限嵌套的数组，可以尝试使用**Underscore**的[flatten](https://github.com/jashkenas/underscore/blob/master/underscore.js#L501)
[这里](http://jsperf.com/flatten-an-array-loop-vs-reduce/6)有对于三个方法表现的测试。

## 39 JavaScript高级属性
在JavaScript中，我们可以对对象`Object`中的属性进行设定。比如我们设定其为**私有**或**只读**。这一特性已经被ES5和新版本浏览器所支持。
因此，你可以通过`Object`的`defineProperty`方法使用。

```js
var a = {};
Object.defineProperty(a, 'readonly', {
  value: 15,
  writable: false
});

a.readonly = 20;
console.log(a.readonly); // 15
```

具体语法：

```js
//Single Line
Object.defineProperty(dest, propName, options)

//Multiple Line
Object.defineProperties(dest, {
  propA: optionsA,
  propB: optionsB, //...
})
```

其中`options`包括以下属性
* `value`值： 如果该属性没有`getter`方法，则`value`是强制设定属性。`{a: 12} === Object.defineProperty(obj, 'a', {value: 12})`
* `writable`属性： 设定属性为**只读**。注意，如果属性是一个嵌套对象，则其属性仍可写。
* `enumerable`：设定属性是否隐藏。这一设定用于说明`for ... of`循环和`stringify`方法将不会能遍历到这一属性，但是该属性仍存在且**可读可写**。
* `configurable`：设定属性不可修改。防止其被删除或重定义。当然，如果该属性为对象，则其内属性仍可以被更改。

因此，假如想要创建私有常量对象，你可以按照下例方式设定：

```js
Object.defineProperty(obj,
  'myPrivateProp', 
  { 
    value: val, 
    enumerable: false, 
    writable: false, 
    configurable: false
  });
```

除了设定属性，`defineProperty`允许我们设定**动态属性**，由于第二个参数是字符串。例如，我们希望根据一些额外设定创建属性。

```js
var obj = {
  getTypeFromExternal(): true // illegal in ES5.1
}

Object.defineProperty(obj, getTypeFromExternal(), {value: true}); // ok

// For the example sake, ES6 introduced a new syntax:
var obj = {
  [getTypeFromExternal()]: true
}
```

高级属性允许我们像面向对象语言那样创建`getter`和`setter`属性。因此，我们不能使用`writable`、`enumerable`和`configurable`属性，而改用：

```js
function Foobar () {
  var _foo; //  true private property

  Object.defineProperty(obj, 'foo', {
    get: function () { return _foo; }
    set: function (value) { _foo = value }
  });

}

var foobar = new Foobar();
foobar.foo; // 15
foobar.foo = 20; // _foo = 20
```

除了先进的访问器和明显封装的优势外，你会发现，我们直接通过**`.`**获得到了属性，没有调用`getter`方法。这么做非常简洁，例如我们有一个多层嵌套的对象。

```js
var obj = {a: {b: {c: [{d: 10}, {d: 20}] } } };
```

相比于`a.b.c[0].d`的语法，我们可以通过创建一个别名调用：

```js
Object.defineProperty(obj, 'firstD', {
  get: function () { return a && a.b && a.b.c && a.b.c[0] && a.b.c[0].d }
})

console.log(obj.firstD) // 10
```

##注意
如果设定了`getter`方法没有设定`setter`方法却想设定值，系统会返回错误。当使用辅助函数例如`$.extend`或`_.merge`时切记这点，注意使用。

# 40 使用`JSON.Stringfy`
假如有一对象存有`prop1`, `prop2`,`prop3`等属性。我们可以通过`JSON>stringify`方法传入**额外的参数**，选择性的更改对象属性值为字符串。

```js
var obj = {
    'prop1': 'value1',
    'prop2': 'value2',
    'prop3': 'value3'
};

var selectedProperties = ['prop1', 'prop2'];

var str = JSON.stringify(obj, selectedProperties);

// str
// {"prop1":"value1","prop2":"value2"}
```

**str**变量中只包含`selectedPropertyies`**数组**中传入选择要的几个属性值。
当然，除了数组我们也可以**传入方法**。

```js
function selectedProperties(key, val) {
    // the first val will be the entire object, key is empty string
    if (!key) {
        return val;
    }

    if (key === 'prop1' || key === 'prop2') {
        return val;
    }

    return;
}
```

`JSON.stringify`方法最后一个可选参数是用于更改写入对象属性的字符串值。

```js
var str = JSON.stringify(obj, selectedProperties, '\t\t');

/* str output with double tabs in every line.
{
        "prop1": "value1",
        "prop2": "value2"
}
*/
```