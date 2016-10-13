##Rx.Observable
###create(fn)
通过Rx.Observable.create创建返回一个**Observable**对象。create中传入参数为函数，Observable对象会在subscribe时候执行该函数，如果该函数有返回值，会返回到subscribe方法当中。

```JS
let foo = Rx.Observable.create(() => next('Hello Rx'));
foo.subscribe(v => console.log(v)) //=> 'Hello Rx'
```
除了上述方法，还可以使用**new**来创建`Observable`对象
```Js
let foo = new Rx.Observable(fn);
const handler = {
  next: function(x) { console.log('next ' + x);},
  error: function(err) { console.log('error ' + err)},
  complete: function() { console.log('done');}
}
foo.subscribe(handler);
```
###of(arg1, arg2, arg3, ...)
Rx.Observable.of()方法中会传入若干参数，当执行subscribe方法时会将这若干个参数返回，最后会传出`complete(完成)`状态信息，表示该Observable对象执行结束。
```JS
let foo = Rx.Observable.of(1, 2, 3, 4, 5);
foo.subscribe( v => console.log(v), 
               err => console.log('error' + err),
               () => console.log('complete done !') )
               // => 1
               // => 2
               // => 3
               // => 4
               // => 5
               // => 'complete done !'
```
###fromArray
在`Rx.Observable.of()`中连传参数是不是很蛋疼？`fromArray`方法来解决你的难言之隐，现在只需要在`Rx.Observable.fromArray()`中传入一个数组可以达到和`of`方法一样的效果了。
```JS
const arr = [1, 2, 3, 4, 5]
let foo = Rx.Observable.of(arr);
foo.subscribe( v => console.log(v), 
               err => console.log('error' + err),
               () => console.log('complete done !') )
               // => 1
               // => 2
               // => 3
               // => 4
               // => 5
               // => 'complete done !'
```
###from
既然`fromArray`方法都这么好用，`from`方法看起来会更厉害的样子。没错，`Rx.Observable.from`方法中可以传入数组、类数组对象、Promise，iterable对象(generator方法返回值)或Observable对象。
同理，在`subscribe`方法中会依次执行并根据处理函数进行处理。

```JS
function *generator() {
  yield 10;
  yield 20;
  yield 30;
}
let iterator = generator();
let foo = Rx.Observable.from(iterator);
foo.subscribe( v => console.log(v), 
               err => console.log('error' + err),
               () => console.log('complete done !') )
               // => 10
               // => 20
               // => 30
               // => 'complete done !'
```
###fromEventPattern & fromEvent
`fromEvenetPattern`方法中传入事件的添加、移除操作方法，具体的处理方法，同样在`subscribe`方法中完成。
下面为`fromEventPattern`方法的原理简单实现。
```Js
function fromEventPattern(add, remove) {
  return Rx.Observable.create(function (observer) {
    add(function(ev) {
      observer.next(ev);
    });
  });
}
```
相对该方法，更推荐使用封装版本的`fromEvent`方法。
`Rx.Observable.fromEvent(DOM, Event)`该方法分别传入需要监听的DOM节点，和需要监听的事件类型。
```Js
let foo = Rx.Observable.fromEvent(document.querySelector('#submit'), 'click');
foo.subscribe(() => console.log('click '));
            // => 'click '
```

###timer & interval
`timer`和`interval`方法都可以传入时间间隔参数，在固定时间内触发事件。

`timer(startTime, intervalTime)`中可以设定事件触发的具体时间**startTime**后才开始执行。

##in deepth
###mapTo
`mapTo`方法和`map`相似，在每次执行时候返回一个值。不过与它不同的点在于，`mapTo`每次返回的是传递给它的常量值。
```Js
let foo = Rx.Observable.interval(1000); //每1秒执行一次
let mapTo = foo.mapTo('Hello');
mapTo.subscribe( x => console.log(x)); 
//每一秒输出一个'Hello'
```
###do
`do`方法通常用于Debug。
它会拦截源Observable对象传递的值，然后执行传递给`do`方法的函数，传递的方法建议不要有返回值，因为会被忽略掉。`Observable`对象返回的仍然是之前传入的值。
```Js
let foo = Rx.Observable.interval(1000).take(4)
                       .do( x => console.log(x));
foo.subscribe({
  next: (x) => console.log('next' + x),
  error: (err) => console.log('error'),
  complete: (x) => console.log('done')
});
  // => 0  
  // => next 0
  // => 1
  // => next 1
  // => 2
  // => next 2
  // => 3
  // => next 3
  // => done
```
###take
`take`方法传入整数参数，会返回该数量的数据。
```Js
let interval = Rx.Observable.interval(1000).take(5);
interval.subscribe(x => console.log(x)); // 0 - 1 - 2 - 3 - 4
```
###takeLast || last
与`take`类似，不过返回的是倒数的几个数据。**并且，这些数据是在最后结束的时候同时返回。**`last`是其缩写。
```Js
let many = Rx.Observable.range(1, 100);
let lastThree = many.takeLast(3);
lastThree.subscribe( x => console.log(x)); // 98|99|100
```
###concat || startWith
与数组的`concat`方法类似，不过此处在后面添加的是`Observable`对象的事件流。
`startWith`方法与`concat`相似，不过就是将事件（数据）流添加在了前面。

```Js
let two = Rx.Observable.interval(2000).take(4);
// --0--1--2--3
let four = Rx.Observable.interval(4000).take(4);
// ----0----1----2----3
let TwoFour = two.concat(four);
TwoFour.subscribe(x => console.log(x)); 
//--0--1--2--3----0----1----2----3
let FourTwo = four.concat(two);
//----0----1----2----3--0--1--2--3
let sTwoFour = four.startWith(two);
//--0--1--2--3----0----1----2----3
let sFourTwo = two.startWith(four);
//----0----1----2----3--0--1--2--3
```
###merge
`merge`方法会将多个`Observable`对象的事件（数据）流合并到同一条流中。
```JS
let two = Rx.Observable.interval(2000).take(4);
// --0--1--2--3
let four = Rx.Observable.interval(4000).take(4);
// ----0----1----2----3
let merge = two.merge(four);
// --0-01--21--3-2----3
```
###skip
`skip(n)`方法传入整数，跳过`Observable`对象的前n个事件（数据）。
```Js
let foo = Rx.Observable.interval(1000).take(10).skip(5);
foo.subscribe(x => console.log(x)); // ----------5-6-7-8-9
```