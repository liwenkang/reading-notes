## JavaScript 基础强化

### 时间(2020年11月16日开始)

### 1. this指向

由问题开始：this指向谁？那些东西会影响到 this 的指向？影响 this 的东东是否有优先级？如何显示/隐式改变 this 指向？

不准确的描述：谁调用，this指向谁

规则：

1. 直接调用 ( 如何定义直接调用 ? ) 的时候，指向全局对象 window/global（ 非严格模式 ）或者 undefined （ 严格模式 )
2. 使用 new 方法调用构造函数的时候，构造函数内的 this 会被绑定到新创建的对象上 ( 这条没注意过 )
3. 通过 apply、call、bind 方法显式调用函数的时候，this 会绑定在指定参数的对象上
4. 通过上下文对象调用函数时，this 会绑定在对象上 ( 谁调用，this 指向谁 )
5. 在箭头函数中，this 指向外层 ( 函数或全局 ) 作用域

```javascript
// 例子
var o1 = {
    text: 'o1',
    fn: function() {
        return this.text
    }
}
console.log(o1.fn()) // 这里的 this 指向的就是 o1

var o2 = {
    text: 'o2',
    fn: function() {
        return o1.fn()
    }
}
console.log(o2.fn()) // 这里实际执行的还是 o1.fn() 所以 this 指向的还是 o1

// 如果这里你想输出 o2 ,该如何修改呢?
// 方法 1 利用 bind/call/apply 在调用时修改 this 指向 o2
console.log(o2.fn.bind(o2)())
console.log(o2.fn.call(o2))
console.log(o2.fn.apply(o2))

// 方法 2 让 o2 成为实际的调用者
var o2 = {
		text: 'o2',
  	fn: o1.fn
}
console.log(o2.fn()) // 此时 o1.fn 里 this 指向的就是实际调用者 o2 了

var o3 = {
    text: 'o3',
    fn: function() {
    		var fn = o1.fn
        return fn() 
    }
}
console.log(o3.fn()) // 这里的 fn 的调用者就是 window 所以 this 执行的是 window,所以返回 undefined

// 疑问题
function returnThis() {
    console.log('this.name', this.name);
    return {
        fn1: function() {
            console.log('this1', this.name);
            return this.name;
        },
        fn2: () => {
            console.log('this2', this.name);
            return this.name;
        }
    };
}

var o3 = {
    name: 'o3',
    fn1: returnThis().fn1,
    fn2: returnThis().fn2,
};

o3.fn1(); 
// this.name undefined
// this1 o3
o3.fn2();
// this.name undefined
// this2 undefined

// 为何此处的执行结果是先输出两遍 this ? ( 均指向全局对象,若在严格模式下,均为 undefined ) ,然后才去输出 this1 ( 指向 o3 ) 和 this2 ( 指向全局对象 ) ? 也就是此处为什么没有顺序执行 ?

// 涉及构造函数的问题
function Foo() {
    this.bar = "Lucas"
}
const instance = new Foo()
console.log(instance.bar) // 可以发现,构造函数内的 this 会绑定到指定参数的对象上

// new 操作符都做了什么
创建了一个空对象

```

### 2. 闭包

### 3. 核心API

### 4. 高频考点和题库

