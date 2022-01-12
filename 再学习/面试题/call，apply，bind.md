# call apply bind

## 用法

**call**

function.call(thisArg, arg1, arg2, ...)

参数

1. thisArg
2. arg1, arg2...

返回值

使用调用这提供的 this 值和参数调用该函数的返回值。若该方法没有返回值，则返回 undefined。

**apply**

function.apply(thisArg, [argsArray])

参数

1. thisArg
2. argsArray

返回值

调用有指定 this 值和参数的函数的结果。如果没有返回undefined

**bind**

function.bind(thisArg, arg1, arg2...)

参数

1. thisArg
2. arg1, arg2

返回值

返回一个原函数的拷贝，并拥有指定的 this 值和初始参数

bind 是返回对应函数，便于稍后调用

apply，call 则是立即调用

## apply、 call

第三方库的许多函数，以及 JavaScript 语言和宿主环境中许多新的内置函数，都提供了可选的参数，通常被称为“上下文”，在 javascript中，call 和 apply 都是为了改变某个函数运行时的上下文而存在的，换句话说，就是为了改变函数体内部 this 的指向。

apply、call 的区别

从 this 绑定的角度来说，call 和 apply 是一样的，它们的区别体现在接收参数的方式不太一样。

```js
func.call(this, arg1, arg2);
func.apply(this, [arg1, arg2]);
```

## bind

```js
// 最简单的
function bind (fn, obj) {
    return function() {
        fn.apply(obj, arguments)
    }
}
bind(func, this)(arg1, arg2)
// MDN
if (!Function.prototype.bind) {
    Function.prototype.bind = function(oThis) {
        if (typeof this !== 'function') {
            throw new TypeError(
                "Function.prototype.bind - what is trying" +
                "to be bound is not callable"
            )
        }

        var aArgs = Array.prototype.slice.call(arguments, 1), 
        fToBind = this, 
        fNOP = function() {}, 
        fBound = function() {
            return  fTobind.apply(
                (
                    this instanceof fNOP &&
                    oThis ? this : oThis
                ),
                aArgs.concat(
                    Array.prototype.slice.call(arguments)
                )
            )
        };

        fNOP.prototype = this.prototype;
        fBound.prototype = new fNOP();

        return fBound;
    }
}
```

回想一下bind是怎么调用的

```js
var a = function(name, age) {
    this.name = name;
    this.age = age;
}
var obj = {};
a.bind(obj, 'tom', '18')();

console.log(obj.name); // 'tom'
console.log(obj.age); // '18'
```

现在回过头来看看，MDN实现

```js
if (!Function.prototype.bind) { // 判断
    Function.prototype.bind = function(oThis) {
        // 将更改的 thisArg 赋值给 oThis;
        if (typeof this !== 'function') {
            // 调用 bind 的必须是函数
            throw new TypeError(
                "Function.prototype.bind - what is trying" +
                "to be bound is not callable"
            )
        }

        
        var aArgs = Array.prototype.slice.call(arguments, 1), // 提取参数，并且变成数组格式
        fToBind = this, // 原this，函数
        fNOP = function() {},
        fBound = function() {
            return  fToBind.apply(
                (
                    // this 是否在 fNOP的原型链上 并且 oThis不为空，这里是针对call是否是new
                    this instanceof fNOP &&
                    oThis ? this : oThis
                ),
                aArgs.concat(
                    Array.prototype.slice.call(arguments)
                )
            )
        };

        fNOP.prototype = this.prototype;
        fBound.prototype = new fNOP();
h
        return fBound;
    }
}

var b = function(){}.call(this);
var cfn = new b();
// 回顾一下 new
// var context =  Object.assign(null, b.prototype)
// b 是 fBound, b.prototype = new fNOP();
// b.apply(context, args)
// this instanceof fNOP 是true,
// 所以 如果 用了 new， 那么就会用 context 而不是用 oThis 当上下文
```

```js
/*
fn.prototype = fNOP.prototype ---> fNOP
                     |             /                         
                     |           /                           
                __proto__      /  new                       
                     |       /                               
                     |     /                                 
            fBound.prototype ---> fBound
                     |             /                         
                     |           /                           
                __proto__      /  new                       
                     |       /                               
                     |     /                  
                    cfn

所以 cfn instanceof fn

所以 当 call 返回的函数，用 new 时，this 指向的是新的 空对象
顺便说明一个问题，prototype可以随意更改，instanceof 来测数据类型不准确
*/
```

知道了原理，来仿写一下

```js
if (!Function.prototype.bind) {
    Function.prototype.bind = function(oThis) {
        // 判断是不是function调用的
        if (typeof this !== 'function') {
            throw new TypeError('不是函数调用')
        }

        var args = Array.prototype.slice.call(arguments, 1);

        var fToBind = this;
        var fTOP = function() {};
        var fBound = function () {
            // 判断是不是 new ， new 的要用 空对象 当this
            return fToBind.apply((
                this instanceof fTOP && oThis ? this : oThis;
            ), args.concat(Array.prototype.slice.call(arguments)));
        }

        fTOP.prototype = this.prototype;
        fBound.prototype = new fTOP();
        
        return fBound;
    }
}
```

## 面试题

1. 定义一个log方法，让它可以代理 console.log 方法

```js
function log() {
    console.log.apply(console, arguments);
}
```

2. 给消息添加一个“(app)”的前缀

```js
function log() {
    var args = Array.prototype.slice.call(arguments);
    args.unshift('(app)');

    console.log.apply(console, args);
}
```

call 和 apply 是立即执行

bind 是返回执行函数
