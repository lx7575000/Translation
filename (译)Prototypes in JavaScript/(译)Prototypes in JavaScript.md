[原文地址](http://code.tutsplus.com/tutorials/prototypes-in-javascript--net-24949)

当你在JavaScript中定义方法时，它会预先定义一些属性，例如其中的`prototype`。在本文中，我会详细介绍它和它的使用方法。

#什么是Prototype？
`prototype`属性被初始化为空对象，不过可以被添加各种属性在其中。

```js
var myObject = function(name){
    this.name = name;
    return this;
};
 
console.log(typeof myObject.prototype); // object
 
myObject.prototype.getName = function(){
    return this.name;
};
```

在上文的代码段中创建了一个方法，但是我们调用`myObject()`，它会返回`window`对象。这是因为它被定义在全局作用域中，`this`因此会返回全局对象，表示其未被实例化。

```js
console.log(myObject() === window); // true
```

#隐式链接
> JavaScript中的每个对象都有一个隐式链接
在我们继续这一话题之前，我们会先讨论下使原型生效的隐式链接。

JavaScript中的每个对象在被定义或实例化时都有一个“秘密”对象被添加进来，关键字为:`__proto__`；这就是为什么原型链会起作用了。然而，在你的应用中使用`__proto__`并不是一个好想法，因为它还没有被所有浏览器中支持(目前应该都支持了)。
`__proto__`属性和对象的`prototype`属性不同，它们是两种不同的属性，这点一定要明确！现在我来解释，当我创建`myObject`方法时，我定义了一个`Function`类型的对象。

```js
console.log(typeof myObject) //function
```

`Function`类型是在JavaScript中是一种会被预先定义的类型，并且它含有自己的属性(例如：`length`和`arguments`)和方法(例如:`call`和`apply`)还有**对象**属性，例如`__proto__`链接。这说明JavaScript引擎中的某个地方，有类似如下代码。

```js
Function.prototype = {
    arguments: null,
    length: 0,
    call: function(){
        // secret code
    },
    apply: function(){
        // secret code
    }
    ...
}
```

事实上，上述代码过于简单，仅仅是为了展示原型链如何工作的。
因此，我们定义`myObject`方法，给其设定了参数`name`，但是我们没有设定其他属性，例如`length`或方法例如`call`。但是它是如何如下运行情况的呢？

```js
console.log(myObject.length); // 1 (being the amount of available arguments)
```

这是因为，当我们定义了`myObject`，它同时生成一个`__proto__`属性，并给其赋值`Function.prototype`。因此，当我们想要调用`myObject.length`时，它会通过`myObject`的原型链寻找`length`属性，但是发现`myObject`并没有定义该属性。因此它通过原型链经过`__proto__`链接向上查找，并返回属性。
你或许会好奇，为什么`length`属性值为**1**，不是其他属性。这是由于事实上`myObject`是`Function`的一个实例。

```js
console.log(myObject instanceof Function); // true
console.log(myObject === Function); // false
```

当一个对象实例被创建，`__proto__`属性会更新指向构造器的原型。在本例中，它是`Function`。

```js
console.log(myObject.__proto__ === Function.prototype) // true
```

除此之外，当创建一个新的`Function`对象时，`Function`构造器会查找参数数量，并且相应更新`this.length`属性，因此此处为**1**。
当我们使用`new`关键字创建一个新的`myObject`实例时，`__proto__`会指向`myObject.prototype`。认为`myObject`是新实例的构造器。

```js
var myInstance = new myObject(“foo”);
console.log(myInstance.__proto__ === myObject.prototype); // true
```

如上面访问`Function.prototype`中的原生代码，例如`call`和`apply`。我们现在在新实例中可以访问`myObject`的方法`getName`。

```js
console.log(myInstance.getName()); // foo
 
var mySecondInstance = new myObject(“bar”);
 
console.log(mySecondInstance.getName()); // bar
console.log(myInstance.getName()); // foo
```

#为什么使用`prototype`更好？
当我们创建一个画图游戏时，同时需要许多对象。每个对象都具有其自己的属性，比如坐标**x**,**y**，长宽**width**,**height**以及其他类型属性。
我们或许会下面这么做：

```js
var GameObject1 = {
    x: Math.floor((Math.random() * myCanvasWidth) + 1),
    y: Math.floor((Math.random() * myCanvasHeight) + 1),
    width: 10,
    height: 10,
    draw: function(){
        myCanvasContext.fillRect(this.x, this.y, this.width, this.height);
    }
   ...
};
 
var GameObject2 = {
    x: Math.floor((Math.random() * myCanvasWidth) + 1),
    y: Math.floor((Math.random() * myCanvasHeight) + 1),
    width: 10,
    height: 10,
    draw: function(){
        myCanvasContext.fillRect(this.x, this.y, this.width, this.height);
    }
    ...
};
```

100遍啊，100遍！！
这么做会令内存中存在100个对象，分别拥有各自的方法（虽然他们都相同，但是仍然会占用100份内存）。这么做一定不符合我们所期望的，毕竟这会导致内存占用过大，程序运行缓慢。
这么做很苦逼，毕竟上百个各不相同的对象需要向上搜索方法。因此，我推荐使用`prototype`,在同一对象上定义统一方法。

#如何使用`prototype`？
为使应用尽可能运行的快（做到最佳实践），我们可以（重新）定义`GameObject`的原型属性，每个`GameObject`的实例都会通过`GameObject.prototype`索引其原型方法进行使用。

```js
// define the GameObject constructor function
var GameObject = function(width, height) {
    this.x = Math.floor((Math.random() * myCanvasWidth) + 1);
    this.y = Math.floor((Math.random() * myCanvasHeight) + 1);
    this.width = width;
    this.height = height;
    return this;
};
 
// (re)define the GameObject prototype object
GameObject.prototype = {
    x: 0,
    y: 0,
    width: 5,
    width: 5,
    draw: function() {
        myCanvasContext.fillRect(this.x, this.y, this.width, this.height);
    }
};
```

下面我们可以实例化对象100遍了。

```js
var x = 100,
arrayOfGameObjects = [];
 
do {
    arrayOfGameObjects.push(new GameObject(10, 10));
} while(x--);
```

现在，我们有一份具有100个`GameObject`实例对象的数组了，它们原型链上分享同一份`draw`方法（真心够节省内存了）。
当我们调用`draw`方法时，它们会索引同一个方法。

```js
var GameLoop = function() {
    for(gameObject in arrayOfGameObjects) {
        gameObject.draw();
    }
};
```

#原型是一个实时对象
对象的原型可以说是动态的，当我们创建了所有实例后，我们希望除了画矩形，我们还想要画圆，因此我们可以相应更新`GameObject.prototype.draw`方法。

```js
GameObject.prototype.draw = function() {
    myCanvasContext.arc(this.x, this.y, this.width, 0, Math.PI*2, true);
}
```

现在，之前和之后创建的`GameObject`实例都可以画圆了。
#更新原生对象
我们可以通过`prototype`更新原生对象的方法。

```js
if(!String.prototype.trim) {
    String.prototype.trim = function() {
        return this.replace(/^\s+|\s+$/g, ‘’);
    };
}
```

（译者）写在最后：作为英语四级勉强过线选手，如果翻译有不到位地方，请多包涵。如若不能包涵就请跟着链接跳转看原文。当然，如果你感觉文章内容不错，就我给个star吧。