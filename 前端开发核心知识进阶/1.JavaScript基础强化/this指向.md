## this指向

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

// 涉及构造函数的问题,也会影响 this 的指向
function Foo() {
    this.bar = "Lucas"
}
const instance = new Foo()
console.log(instance.bar) // 可以发现,构造函数内的 this 会绑定到指定参数的对象上
```

```javascript
// 简易版 new
function Car(name) {
  this.name = name
}
const obj = {}
obj.__proto__ = Car.prototype
Car.call(obj)
```

new 操作符都做了什么,根据 mdn 上的定义
The new keyword does the following things:
1. Creates a blank, plain JavaScript object;
```javascript
// 如何生成一个空对象?
const temp1 = Object.create(null)
const temp2 = Object.create({})
const temp3 = {}
// temp1 里啥都没有
// temp2.__proto__.__proto__.constrctor === Object
// temp3.__proto__.constrctor === Object

// 如何 polyfill Object.create ?
const x = Object.create(null)
// 等价于 
const y = {}
y.__proto__ = null
```

2. Links (sets the constructor of) the newly created object to another object by setting the other object as its parent prototype;
```javascript
function myNew(constrctor) {
  const temp1 = Object.create(null)
  temp.__proto__ = constrctor.prototype

  // 同下
  const temp2 = Object.create(constrctor.prototype)
}
```

todo 如何正确区分 prototype(只有函数才有) 和 __proto__(只要是对象就有,是对象的内部属性)

3. Passes the newly created object from Step 1 as the this context;
```javascript
function myNew(constrctor, ...args) {
  const temp = Object.create(constrctor.prototype)
  const ctorReturnResult = constrctor.apply(temp, argsArr);
}
```

4. Returns this if the function doesn't return an object.
```javascript
function myNew(constructor, ...args) {
    if (typeof constructor !== 'function') {
        return new Error(constructor + 'must be a function, please fix it')
    }
    // constructor 必须是一个函数
    const temp = Object.create(constructor.prototype);
    const returnResult = constructor.apply(temp, args);
    if (((typeof returnResult) === 'object' && returnResult !== null) || (typeof returnResult) === 'function') {
        // 如果 returnResult 是非 null 的 object 或者 function 的话,可以直接返回 returnResult
        return returnResult;
    } else {
        return temp;
    }
}
```

对于箭头函数来说, this 指向的是外层作用域
```javascript
const test = {
    fn1: function() {
        // 指向 test
        console.log('fn1', this) // test
        setTimeout(function() {
            console.log('fn1 in', this); // 此处匿名函数的调用者是 window, 所以 this 指向 window
        });
    },
    fn2: function() {
        // 指向 test
        console.log('fn2', this) // test
        setTimeout(() => {
            console.log('fn2 in', this); // 此处由于是箭头函数,this 指向外层作用域,也就是外层 function 所在的作用域
        });
    },
    fn3: () => {
        // 指向 window
        console.log('fn3', this) // window
        setTimeout(function() {
            console.log('fn3 in', this); // 此处匿名函数的调用者是 window, 所以 this 指向 window
        });
    },
    fn4: () => {
        // 指向 window
        console.log('fn4', this) // window
        setTimeout(() => {
            console.log('fn4 in', this); // 此处由于是箭头函数,this 指向外层作用域,也就是外层箭头函数所在的作用域,也就是在外面一层函数的作用域,也就是 window
        });
    }
};

test.fn1();
test.fn2();
test.fn3();
test.fn4();
```

会影响 this 的东西是不是有优先级呢?
结论: new > bind(apply, call)显式绑定 > 隐式绑定

```javascript
// 1. 显式 > 隐式
function foo() {
    console.log('this', this);
    console.log('this.a', this.a);
}

const obj1 = {
    a: 1,
    foo: foo
};

const obj2 = {
    a: 2,
    foo: foo
};

obj1.foo(); // 1
obj2.foo(); // 2
obj1.foo.call(obj2); // 2 此处说明 bind 的优先级比隐式绑定更高
obj2.foo.call(obj1); // 1 此处说明 bind 的优先级比隐式绑定更高

// 2. new > bind(apply, call)
function foo(a) {
    this.a = a;
}

const obj = {};

var bar = foo.bind(obj);
bar(1);
console.log(obj); // { a: 1 }
console.log(obj.a); // 1

var baz = new bar(2);
console.log('baz', baz); // { a: 2 }
console.log('baz.a', baz.a); // 2

// 3. 箭头函数的情况
function foo() {
    console.log('this out', this);  // { a: 2 }
    return a => {
        console.log('this in', this); // { a: 2 }
    };
}

var obj1 = {
    a: 2
};

var obj2 = {
    a: 3
};

// 第一次 call 只是修改了 foo 这个函数的 this 指向, 从而印象了箭头函数里面的 this 指向(因为箭头函数里面的 this 指向是由外层作用域决定的)
var bar = foo.call(obj1);
// 重新用 call 绑定失败, 因为箭头函数的绑定不能修改(其实只能通过修改箭头函数外层的作用域,从而间接修改箭头函数的 this 指向)
console.log(bar.call(obj2));
```

待办事项:
1. 解决疑问(执行顺序和 __proto__ 和 prototype 的联系和区别)
2. 实现一个简易版 bind
3. 实现 es5-shim 版本的 bind
