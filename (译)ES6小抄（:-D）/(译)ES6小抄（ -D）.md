# (译)ES6小抄（:-D）

[原文地址](https://github.com/DrkSephy/es6-cheatsheet)

#1 `var` VS `let`/`const`

在JavaScript中除了**var类型**外，我们现在还可以使用`let`和`const`两种新类型进行变量声明了。与`var`不同，`let`和`const`类型会按照定义的顺序产生变量，而`var`在作用域任意位置定义都可。

```js
//var 的例子
var snack = 'Meow Mix';

function getFood(food) {
    if (food) {
        var snack = 'Friskies';
        return snack;
    }
    return snack;
}

getFood(false); // undefined

//我们改为使用let 代替 var
let snack = 'Meow Mix';

function getFood(food) {
    if (food) {
        let snack = 'Friskies';
        return snack;
    }
    return snack;
}

getFood(false); // 'Meow Mix'
```

当我在使用`let`替换`var`重构代码时，我们需要更加小心了。因为类型作用域的问题会导致很多意想不到的麻烦。
  
  **Note:** `let`和`const`类型会限定自身的有效作用域，因此，当在定义变量**之前**引用所定义的变量时会产生**引用错误**

```js
console.log(x);

let x = 'hi'; // ReferenceError: x is not defined
```

**最佳实践**
使用`var`声明变量当对代码进行重构时，需要仔细判断其作用范围。因此，对于新的项目，建议使用`let`声明变量，使用`const`声明不允许改变的常量。

#2 使用块作用域替换**IIFES**
使用**立即执行函数表达式**的一种方式是将其包含在小括号当中。在ES6中，我们可以创建一个非基于函数作用域形式的块级作用域。

```js
(function () {
    var food = 'Meow Mix';
}());

console.log(food); // Reference Error
//ES6
{
    let food = 'Meow Mix';
}

console.log(food); // Reference Error
```

#3 箭头函数
通常我们使用嵌套函数是想要在词法作用域中保留`this`。

```js
function Person(name) {
    this.name = name;
}

Person.prototype.prefixName = function (arr) {
    return arr.map(function (character) {
        return this.name + character; // Cannot read property 'name' of undefined
    });
};
```

最常见的解决方式是通过赋值变量保留上级作用域的`this`

```js
function Person(name) {
    this.name = name;
}

Person.prototype.prefixName = function (arr) {
    var that = this; // Store the context of this
    return arr.map(function (character) {
        return that.name + character;
    });
};
```

我们也可以通过合适的方式传递`this`

```js
function Person(name) {
    this.name = name;
}

Person.prototype.prefixName = function (arr) {
    return arr.map(function (character) {
        return this.name + character;
    }, this);
};
```

`bind`也是一个好办法

```js
function Person(name) {
    this.name = name;
}

Person.prototype.prefixName = function (arr) {
    return arr.map(function (character) {
        return this.name + character;
    }.bind(this));
};
```

最后使用箭头函数，`this`的词法值不再隐藏而会自动绑定。我们重写上述例子

```js
function Person(name) {
    this.name = name;
}

Person.prototype.prefixName = function (arr) {
    return arr.map(character => this.name + character);
};
```

**最佳实践** 无论何时你需要保持`this`的值，使用箭头函数。

当仅仅需要通过函数方法返回一个值时，使用箭头函数也更加简洁。

```js
var squares = arr.map(function (x) { return x * x }); // Function Expression
const arr = [1, 2, 3, 4, 5];
const squares = arr.map(x => x * x); // Arrow Function for terser implementation
```

**最佳实践** 尽可能的使用箭头函数作为函数表达式。

#4 字符串
在ES6中，标准库变得更加丰富。其中有更多可以应用在字符串方面的方法。比如：`.includes()`和`.repeat()`。

## .includes()

```js
var string = 'food';
var substring = 'foo';

console.log(string.indexOf(substring) > -1);
```

与通过是否返回-1判断是否包含字符串相比我们可以使用`.includes()`返回boolean类型更简单的进行判断。

```js
const string = 'food';
const substring = 'foo';

console.log(string.includes(substring)); // true
```

## .repeat()

```js
//ES5
function repeat(string, count) {
    var strings = [];
    while(strings.length < count) {
        strings.push(string);
    }
    return strings.join('');
}
//ES6
// String.repeat(numberOfRepetitions)
'meow'.repeat(3); // 'meowmeowmeow'
```

##模板字符串
使用模板字符串，我们可以不需要特殊说明直接构建含有特殊字符的字符串。

```js
var text = "This string contains \"double quotes\" which are escaped.";
let text = `This string contains "double quotes" which are escaped.`;
```

模板字符串也支持**字符串+变量值**这一类型的变量插入操作。

```js
//ES5
var name = 'Tiger';
var age = 13;

console.log('My cat is named ' + name + ' and is ' + age + ' years old.');

//ES6
//Much simpler:
const name = 'Tiger';
const age = 13;

console.log(`My cat is named ${name} and is ${age} years old.`);
```

在ES5中，创建多行字符串，我们需要：

```js
var text = (
    'cat\n' +
    'dog\n' +
    'nickelodeon'
);

//Or:

var text = [
    'cat',
    'dog',
    'nickelodeon'
].join('\n');
```

在ES6当中，模板字符串不需要特殊说明，默认保存多行字符串。

```js
let text = ( `cat
  dog
  nickelodeon`
);
```

同时，在模板字符串中我们可以添加表达式：

```js
let today = new Date();
let text = `The time and date is ${today.toLocaleString()}`;
```

#5 重构
重构方法允许我们使用更方便的方法，从数组或者对象中获取特定的属性或元素，并将它们存储在变量中。
##重构数组

```js
var arr = [1, 2, 3, 4];
var a = arr[0];
var b = arr[1];
var c = arr[2];
var d = arr[3];
let [a, b, c, d] = [1, 2, 3, 4];

console.log(a); // 1
console.log(b); // 2
```

##重构对象

```js
var luke = { occupation: 'jedi', father: 'anakin' };
var occupation = luke.occupation; // 'jedi'
var father = luke.father; // 'anakin'
let luke = { occupation: 'jedi', father: 'anakin' };
let {occupation, father} = luke;

console.log(occupation); // 'jedi'
console.log(father); // 'anakin'
```

#模块
ES6之前，我们使用[Browserify](http://browserify.org/)等库在客户端创建模块，并且使用**Node.js**的[require](https://nodejs.org/api/modules.html#modules_module_require_id)库引用。在ES6中我们可以直接使用所有类型的模块(AMD 和 CommenJS)。

## 使用CommonJS方式Export
在ES6中，我们有多种方式传递出模块。通常使用**命名传递的形式**

```js
export let name = 'David';
export let age  = 25;
```

**也可以使用基于对象列表的形式传出**

```js
function sumTwo(a, b) {
    return a + b;
}

function sumThree(a, b, c) {
    return a + b + c;
}

export { sumTwo, sumThree };
```

当然，简单的使用**`export`**关键字传出方法、对象和值。

```js
export function sumTwo(a, b) {
    return a + b;
}

export function sumThree(a, b, c) {
    return a + b + c;
}
```

最后，还有`export default`**绑定**的方式

```js
function sumTwo(a, b) {
    return a + b;
}

function sumThree(a, b, c) {
    return a + b + c;
}

let api = {
    sumTwo,
    sumThree
};

export default api;
```

**最佳实践:** 在模块文件的最后使用`export default`方法。这么做可以让人很清楚明白传递的是什么方法、对象，并且可以避免费时判断各值的命名。CommonJS最常见做法为每次只传递出一个值或对象。坚持这种模式，我们可以使代码更易读，并且允许我们插入到CommonJS或ES6模块中

# ES6的`import`
ES6为我们提供了许多导入模块的方法，我们可以使用其导入整文件。

```js
import 'underscore';
```

  > 需要提醒简单的整文件导入，会执行整个文件内的所有代码

与python相似我们可以基于命名导入模块：

```js
import { sumTwo, sumThree } from 'math/addition';
```

我们也可以重命名导入的模块

```js
import {
    sumTwo as addTwoNumbers,
    sumThree as sumThreeNumbers
} from 'math/addition';
```

除此之外，可以使用`*`引入所有模块（命名空间导入）

```js
import * as util from 'math/addition';
```

最后，我们可以通过像从模块中引入一系列值

```js
import * as additionUtil from 'math/addition';
const { sumTwo, sumThree } = additionUtil;
```

当引入默认对象时，我们可以指定特定方法进行引用。

```js
import React from 'react';
const { Component, PropTypes } = React;

//简单方法
import React, { Component, PropTypes } from 'react';
```

  > 注意： 导出的值是绑定不是引用。因此，改变模块内被绑定的变量将影响导出模块内的值。因此，避免更改导出这些值的公共接口。

