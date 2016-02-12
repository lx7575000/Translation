# （译）JSTips（41-）

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