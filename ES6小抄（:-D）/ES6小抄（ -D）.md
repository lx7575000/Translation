# ES6小抄（:-D）

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