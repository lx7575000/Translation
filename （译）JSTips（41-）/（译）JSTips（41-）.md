\#41 数值数组的平均数和中位数

```js
let values = [2, 56, 3, 41, 0, 4, 100, 23];
```

下面所有例子，都基于上述的数组。
为了求得数组平均值，我们需要加和，并且除以元素个数。
* 获得数组元素个数
* 加和所有元素值
* 获得平均值(总和 / 个数)

```js
let values = [2, 56, 3, 41, 0, 4, 100, 23];
let sum = values.reduce((pre vious, current) => current += previous);
let avg = sum / values.length;
// avg = 28

//OR

let values = [2, 56, 3, 41, 0, 4, 100, 23];
let count = values.length;
values = values.reduce((previous, current) => current += previous);
values /= count;
// avg = 28
```

获取中位数步骤：
* 重排序数组
* 获取中间值

```js
let values = [2, 56, 3, 41, 0, 4, 100, 23];
values.sort((a, b) => a - b);
let middle = Math.floor(values.length / 2);
let median = values[middle];
// median = 23

//或者使用右移操作符
let values = [2, 56, 3, 41, 0, 4, 100, 23];
values.sort((a, b) => a - b);
let median = values[values.length >> 1];
// median = 23
```

# 42 禁止原型链改写
通过重写原型链，攻击者可以重写代码公开和更改绑定参数。通过ES5的来开发会产生一个严重的安全漏洞。

```js
// example bind polyfill
function bind(fn) {
  var prev = Array.prototype.slice.call(arguments, 1);
  return function bound() {
    var curr = Array.prototype.slice.call(arguments, 0);
    var args = Array.prototype.concat.apply(prev, curr);
    return fn.apply(null, args);
  };
}

// unapply-attack
function unapplyAttack() {
  var concat = Array.prototype.concat;
  Array.prototype.concat = function replaceAll() {
    Array.prototype.concat = concat; // restore the correct version
    var curr = Array.prototype.slice.call(arguments, 0);
    var result = concat.apply([], curr);
    return result;
  };
}
```

上面的`unapplyAttack`函数丢弃了`prev`数组，意味着函数中任意一个`.concat`方法都会导致抛出错误。

使用[Object.freeze](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/freeze)令对象不可更改，防止原型链重写。

```js
(function freezePrototypes() {
  if (typeof Object.freeze !== 'function') {
    throw new Error('Missing Object.freeze');
  }
  Object.freeze(Object.prototype);
  Object.freeze(Array.prototype);
  Object.freeze(Function.prototype);
}());
```

# 43 使用解构提取函数参数
你可能已经很熟悉ES6的解构特性，但你是否知道可以用它来取得函数传递的参数值？

```js
var sayHello = function({ name, surname }) {
  console.log(`Hello ${name} ${surname}! How are you?`);
};

sayHello({
  name: 'John',
  surname: 'Smith'
});
```

这方法对于接收可选对象作为参数的函数非常有用。
> 切记，**解构赋值**并非在NodeJS和所有浏览器都可应用。你可以使用NodeJS的`--harmony-destructuring`标志来尝试在项目中使用。

#44 了解参数传值机制
严格来说，JavaScript是基于值传递的。按照真正的条款来看，**它既是值传递也是参数传递**。想要了解传参规则，你要认真看看下列两例代码段和解释。
##例一

```js
var me = {                  // 1
    'partOf' : 'A Team'
}; 

function myTeam(me) {       // 2

    me = {                  // 3
        'belongsTo' : 'A Group'
    }; 
}   

myTeam(me);     
console.log(me);            // 4  : {'partOf' : 'A Team'}
```

在上例中，当`myTeam`被调用。JavaScript传递`me`对象的引用值，因为它是一个对象和调用本身创造的**两个**对同一对象的独立引用（尽管名字相同都为`me`，误导我们认为是同一份引用）。但是引用变量本身是独立的。
当我们在**#3**为其赋值了一份新对象时，我们在`myTeam`方法中彻底更改了引用值（对象重新赋值）。但是它不会对外部的原生对象产生影响。因此，尽管作为参数被传递了，但是在函数外部，对象仍保持原值所以在**#4**处的输出仍为原值。

##例二

```js
var me = {                  // 1
    'partOf' : 'A Team'
}; 

function myGroup(me) {      // 2
    me.partOf = 'A Group';  // 3
} 

myGroup(me);
console.log(me);            // 4  : {'partOf' : 'A Group'}
```

我们传递对象`me`给`myTeam`函数作为参数。但是与例一不同，不是将`me`对象重新赋值为新对象。`myTeam`作用域中的对象仍与外部原生对象相关。因此，我们在函数内对对象内属性更改，会反映到外作用域的原生对象内。所以**#4**会输出这一结果。

所以，后例不能证明JavaScript参数是值传递？不，它的确是值传递。JavaScript中的对象会被以传值的形式来传引用（对象内属性存储的可以理解为是具体值的地址位置，因此以值传递到函数内，地址值不变。）。这很令人费解，我们不能很好理解什么是传引用。基于这一原因，人们更愿意称其为**共享传递**

#45 计算求取数组内最大/最小值
JavaScript中有两种内置方法可以计算出传入参数中的最大/小值，但是却不支持数组参数。

```js
Math.max(1, 2, 3, 4); // 4
Math.min(1, 2, 3, 4); // 1
```

`apply()`方法支持使用内建函数查找数组内的最值。

```js
var numbers = [1, 2, 3, 4];
Math.max.apply(null, numbers) // 4
Math.min.apply(null, numbers) // 1
```

传入第一个函数用于限定作用域，在这里，这点不重要。
可以通过使用**ES7**的分隔操作符**`...`**

```js
var numbers = [1, 2, 3, 4];
Math.max(...numbers) // 4
Math.min(...numbers) // 1
```