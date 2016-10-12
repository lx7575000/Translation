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