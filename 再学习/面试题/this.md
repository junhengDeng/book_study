# 关于 this

## this 误解

- 指向自身
- 它的作用域

## this 到底是什么

this 是在运行时进行绑定的，并不是在编写时绑定，它的上下文取决于函数调用时的各种条件。this 的绑定和函数声明的位置没有任何关系，只取决于函数的调用方式。

当一个函数被调用时，会创建一个活动记录（有时候也称为执行上下文）。这个记录会包含函数在哪里被调用（调用栈），函数的调用方法、传入的参数等信息。this 就是记录的其中一个属性，会在函数执行的过程中用到。

this 实际上是在函数被调用时发生的绑定，它指向什么完全取决于函数在哪里被调用。

# this 全面解析

## 调用位置

在理解 this 的绑定过程之前，首先要理解调用位置；调用位置就是函数在代码中被调用的位置（而不是声明的位置）。只有仔细分析调用位置才能回答这个问题：这个 this 到底引用的是什么？

通常来说，寻找调用位置就是寻找“函数被调用的位置”，但是做起来并没有这么简单，因为某些编程模式可能会隐藏真正的调用位置。

最重要的时分析调用栈（就是为了到达当前执行位置所调用的所有函数）。我们关心的调用位置就在当前正在执行的函数的前一个调用中。

```js
function baz() {
 // 当前调用栈是：baz
 // 因此，当前调用位置是全局作用域
 console.log( "baz" );
 bar(); // <-- bar 的调用位置
}
function bar() {
// 当前调用栈是 baz -> bar
 // 因此，当前调用位置在 baz 中
 console.log( "bar" );
 foo(); // <-- foo 的调用位置
}
function foo() {
 // 当前调用栈是 baz -> bar -> foo
 // 因此，当前调用位置在 bar 中
 console.log( "foo" );
}
baz(); // <-- baz 的调用位置
```

注意我们是如何（从调用栈中）分析出真正的调用位置的，因为它决定 this 的绑定。

## 绑定规则

看绑定规则之前，必须先找到调用位置。然后再判断需要应用下面四条规则中的哪一条。

### 默认绑定

```js
function foo() { 
 console.log( this.a );
}

var a = 2;
foo(); // 2
```

可以通过分析调用位置来看看 foo() 是如何调用的。
foo() 是直接使用不带任何修饰的函数引用进行调用的，因此只能使用
默认绑定，无法应用其他规则。

注意，严格模式下，无法使用默认绑定。


### 隐式绑定

```js
function foo() { 
 console.log( this.a );
}
var obj = { 
 a: 2,
 foo: foo 
};
obj.foo(); // 2
```

调用位置在 obj 上下文中，当函数引用有上下文对象时，隐式绑定规则会把函数调用中的 this 绑定到这个上下文对象。

#### 隐式丢失

一个最常见的 this 绑定问题就是被隐式绑定的函数会丢失绑定对象，也就是它会应用默认绑定，从而把 this 绑定到全局对象或者 undefined 上。

```js
function foo() { 
 console.log( this.a );
}
var obj = { 
 a: 2,
 foo: foo 
};
var bar = obj.foo; // 函数别名！
var a = "oops, global"; // a 是全局对象的属性
bar(); // "oops, global"
```

虽然 bar 是 obj.foo 的一个引用，但是实际上，它引用的是 foo 函数本身，因此此时的 bar() 其实是一个不带任何修饰的函数调用，因此应用了默认绑定。

还有一种更微妙的情况

```js
function foo() { 
 console.log( this.a );
}
function doFoo(fn) {
 // fn 其实引用的是 foo
 fn(); // <-- 调用位置！
}
var obj = { 
 a: 2,
 foo: foo 
};
var a = "oops, global"; // a 是全局对象的属性
doFoo( obj.foo ); // "oops, global"
```

参数传递其实就是一种隐式赋值，因此我们传入函数时也会被隐式赋值，所以结果和上一个例子一样。

```js
function foo() { 
 console.log( this.a );
}

var obj = { 
 a: 2,
 foo: foo 
};
var a = "oops, global"; // a 是全局对象的属性
setTimeout( obj.foo, 100 ); // "oops, global"
```

JavaScript 环境中内置的 setTimeout() 函数实现和下面的伪代码类似：

```js
function setTimeout(fn,delay) {
 // 等待 delay 毫秒
 fn(); // <-- 调用位置！
}
```

### 显式绑定

call 和 apply

它们第一个参数是一个对象，它们会把这个对象绑定到 this，接着在调用函数时执行这个 this。

可惜，显式绑定仍然无法解决我们之前提出的丢失绑定问题。

#### 硬绑定

但是显式绑定的一个变种可以解决这个问题。

```js
function foo() {
    console.log(this.a);
}

var obj = {
    a: 2
}

var bar = function() {
    foo.call(obj)
}

bar(); // 2
setTimeout(bar, 100) // 2

// 硬绑定的 bar 不可能再修改它的this
bar.call(window); // 2
```

硬绑定的应用场景就是创建一个包裹函数，传入所有的参数并返回接收到的所有值。

bind 就是硬绑定。

bind() 会返回一个硬编码的新函数，它会把参数设置为 this 的上下文并调用原始函数。

### new 绑定

javascript 中的 new 的机制实际上和面向类的语言完全不同。

使用 new 来调用函数，或者说发生构造函数调用时，会自动执行下面的操作。

1. 创建一个全新的对象
2. 这个新对象会被执行 [[ 原型 ]] 连接
3. 这个新对象会绑定到函数调用的 this
4. 如果函数没有返回其他对象，那么 new 表达式中的函数调用会自动返回这个新对象

根据描述，手写一个new

```js
function NEWAPI(...args) {
    // 提取第一个参数，作为函数调用
    const fn = args.shift();
    // 构建新的对象,并且将fn的prototype绑定到当前对象的__proto__
    const context = Object.create({}, fn.prototype);
    // 新对象绑定到函数调用的 this
    const res = fn.apply(context, args);
    // 如果函数没有返回对象，就返回context
    return typeof res === 'object' ? res : context;
}

function Person(name) {
    this.name = name;
}

new Person('tony')

NEWAPI(Person, 'tom')
```

## 优先级

new > 隐式

显式 > 隐式

## 判断 this

1. 函数是否在 new 中调用（new 绑定）？如果是的话 this 绑定的是新创建的对象

var bar = new foo()

2. 函数是否通过 call， apply 或者硬绑定调用？如果是的话，this 绑定的是指定的对象。

var bar = obj1.foo()

3. 函数是否在某个上下文对象中调用？如果是的话，this 绑定的是那个上下文对象

var bar = obj1.foo()

4. 如果都不是的话，使用默认绑定。如果在严格模式下，就绑定到 undefined，否则绑定到全局对象。

var bar = foo();

## 箭头函数

箭头函数不使用 this 的四种标准规则，而是根据外层（函数或者全局）作用域来决定 this

```js
function foo() {
    var self = this; // lexical capture of this 
    setTimeout( function(){
        console.log( self.a );
    }, 100 );
} 
var obj = {
    a: 2
};
foo.call( obj ); // 2
```

用外层的 this

# 小结

1. 先确定调用位置
2. 规则
- 默认绑定
- 显式绑定（call, apply）
- 硬绑定（bind）
- new
3. new处理过程
- 生成新对象
- 将函数的prototype绑定到新对象的原型
- 将新对象绑定为函数的this调用
- 如果函数返回值为非对象，则返回新对象
4. 隐藏的坑
```js
// 1
function foo() { 
 console.log( this.a );
}
var obj = { 
 a: 2,
 foo: foo 
};
var bar = obj.foo; // 函数别名！
var a = "oops, global"; // a 是全局对象的属性
bar(); // "oops, global"

// 2
function foo() { 
 console.log( this.a );
}
function doFoo(fn) {
 // fn 其实引用的是 foo
 fn(); // <-- 调用位置！
}
var obj = { 
 a: 2,
 foo: foo 
};
var a = "oops, global"; // a 是全局对象的属性
doFoo( obj.foo ); // "oops, global"
```

5. 箭头函数

将外层的this，拿到里面来用
```js
function foo() {
    let self = this;
    setTimeout(function() {
        console.log(self.name)
    }, 100)
}
foo.call({name:'tom'})
```
