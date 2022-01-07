## new 干了什么

1. 生一个空对象
2. 将构造函数的原型挂载在空对象的__proto__上
3. 构造函数在空对象的上下文中执行
4. 执行结果如果不是对象则返回上面生成的对象；

## 手写一个 new

```js
// 用原生的写
function _new() {
    // 将参数提取出来
    var args = [];
    for(var i=0; i<arguments.length; i++) {
        args[i] = arguments[i]
    }
    // 提取第一个参数，作为构造函数，剩下的为参数
    var constructor = args.shift();
    // 核心
    // 生成空对象，并且将对象的__proto__绑定构造函数的原型
    var context = {};
    context.__proto__ = constructor.prototype;
    // 在context环境下执行构造函数
    var result = constructor.apply(context, args);
    // 如果result是对象，则返回result,否则返回context
    return typeof result === 'object' ? result : context;
}
```

```js
// 用es6写
function _new(constructor, ...args) {
    const context = Object.create(null, constructor.prototype)
    const result = constructor.apply(context, args);
    return typeof result === 'object' ? result : context;
}
```

## 小结

这个没有小结