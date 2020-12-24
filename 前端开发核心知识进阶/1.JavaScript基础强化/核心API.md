## 核心API

1. 获取一个元素在页面上的真实位置(非当前窗口视角,也就是说如果往下滚动了,那么滚动部分的高度也要算到位置高度里)

方法:
(1)jquery.offset
```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>offset demo</title>
  <style>
  p {
    margin-left: 10px;
    color: blue;
    width: 200px;
    cursor: pointer;
  }
  span {
    color: red;
    cursor: pointer;
  }
  div.abs {
    width: 50px;
    height: 50px;
    position: absolute;
    left: 220px;
    top: 35px;
    background-color: green;
    cursor: pointer;
  }
  </style>
  <script src="../jquery-3.5.0.js"></script>
</head>
<body>
 
<div id="result">Click an element.</div>
<p>
  This is the best way to <span>find</span> an offset.
</p>
<div class="abs">
</div>
 
<script>
$("*", document.body).click(function( event ) {
  var offset = $(this).offset();
  event.stopPropagation();
  $("#result").text(this.tagName + "coords ( " + offset.left + ", " + offset.top + " )" );
});
</script>
</body>
</html>
```

(2)递归

会导致元素位置偏移的属性有哪些?
margin
float
flex
position 非 static 时, left, top
transform 里的 translate scale

```javascript
const offset = ele => {
    let result = {
        top: 0,
        left: 0
    };

    if (window.getComputedStyle(ele)['display'] === 'none') {
        return result;
    }

    let position;

    const getOffset = (node, init) => {
        // 如果 nodeType === 1 说明是 元素节点,那么才能 go on
        if (node.nodeType !== 1) {
            return;
        }

        // 获取 position 属性
        position = window.getComputedStyle(node)['position'];

        // 如果 position 是 static, 也就是没有手动修改 position
        // 而且不是第一次调用,也就是 node 已经是一个 fatherNode 了
        if (typeof init === 'undefined' && position === 'static') {
            getOffset(node.parentNode);
            return;
        }

        // offsetTop - scrollTop
        result.top = result.top + node.offsetTop - node.scrollTop;
        result.left = result.left + node.offsetLeft - node.scrollLeft;

        // 如果是 fixed 那就终止
        if (position === 'fixed') {
            return;
        }

        getOffset(node.parentNode);
    };

    getOffset(ele, true);

    return result;
};
```

(3)getBoundingClientRect
```javascript
const offset = ele => {
    let result = {
        top: 0,
        left: 0
    };

    if (!ele.getClientRects().length) {
        return result;
    }

    // display none 的话,直接返回
    if (window.getComputedStyle(ele).display === 'none') {
        return result;
    }

    result = ele.getBoundingClientRect();
    const docElement = ele.ownerDocument.documentElement;

    return {
        top: result.top + window.pageYOffset - docElement.clientTop,
        left: result.left + window.pageXOffset - docElement.clientLeft
    };
};
```

2. reduce
promise 相关 promise.all promise.race
```javascript
var p1 = Promise.resolve(3);
var p2 = 1337;
var p3 = new Promise((resolve, reject) => {
    setTimeout(resolve, 1000, 'foo');
});
var p4 = new Promise((resolve, reject) => {
    setTimeout(reject, 1000, 'err');
});

Promise.all([p1, p2, p3]).then(values => {
    console.log(values); // [3, 1337, "foo"]
})

Promise.all([p1, p2, p4]).then(values => {
    console.log(values);
}).catch(error => {
    console.log('error', error); // error err
});

Promise.race([p1, p2, p3, p4]).then(values => {
    console.log(values); // 3
}).catch(error => {
    console.log('error', error);
});
```

Promise.all 的同步异步讨论
```javascript
// we are passing as argument an array of promises that are already resolved,
// to trigger Promise.all as soon as possible
var resolvedPromisesArray =
    [Promise.resolve(33), Promise.resolve(44)];

var p = Promise.all(resolvedPromisesArray);
// immediately logging the value of p
console.log(p);

// using setTimeout we can execute code after the stack is empty
setTimeout(function() {
    console.log('the stack is now empty');
    console.log(p); 
});

// logs, in order:
// Promise { <state>: "pending" }
// the stack is now empty
// Promise { <state>: "fulfilled", <value>: Array[2] }

// 说明 Promise.all 执行完之后,才会执行 setTimeout
```

仅当 Promise.all(array) array 为空数组的时候,会同步执行
```javascript
var p = Promise.all([]); // will be immediately resolved 此时为同步
var p2 = Promise.all([1337, 'hi']); // non-promise values will be ignored, but the evaluation will be done asynchronously
console.log('p', p);
console.log('p2', p2);
setTimeout(function() {
    console.log('the stack is now empty');
    console.log('p2 in', p2);
});

// Promise { <state>: "fulfilled", <value>: Array[0] }
// Promise { <state>: "pending" }
// the stack is now empty
// Promise { <state>: "fulfilled", <value>: Array[2] }
```

```javascript
// 按照顺序执行 promise
const runPromiseInSequence =
    (array, value) =>
        array.reduce((promiseChain, currentFunction) =>
            promiseChain.then(currentFunction), Promise.resolve(value));

const f1 = () => new Promise((resolve, reject) => {
    setTimeout(() => {
        console.log('f1 running');
        resolve('f1');
    }, 1000);
});

const f2 = () => new Promise((resolve, reject) => {
    setTimeout(() => {
        console.log('f2 running');
        resolve('f2');
    }, 1000);
});

// 先后顺序 reduce
runPromiseInSequence([f1, f2])

// async await
async function main1() {
    await f1();
    await f2();
}

// promise 嵌套
function main2() {
    f1().then(response => {
        console.log('response', response)
        f2();
    }).catch(error => {
        console.log('error', error);
    });
}
```

```javascript
// pipe
const pipe = (...functions) => input => functions.reduce(
    (accumulator, functionItem) => functionItem(accumulator),
    input
);

const f = function(value) {
    return value + 1;
};

const g = function(value) {
    return value + 2;
};

const h = function(value) {
    return value + 3;
};

console.log(h(g(f(1))));
console.log(pipe(f, g, h)(1));
```

```javascript
// 实现 reduce
Array.prototype.myReduce = function(func, initialValue) {
    var array = this;
    var base;
    var startIndex;
    // 区分一下 初始值,以及从哪个开始迭代?
    if (initialValue) {
        base = initialValue;
        startIndex = 0;
    } else {
        base = this[0];
        startIndex = 1;
    }
    for (let i = startIndex; i < array.length; i++) {
        base = func(base, array[i], i, array);
    }
    return base;
};
```

// Koa Only 模块
```javascript
var o = {
    a: 'a',
    b: 'b',
    c: 'c'
};

const only1 = function(obj = {}, keys) {
    const newObj = {};
    for (let key in obj) {
        if (keys.includes(key)) {
            if (obj[key] !== undefined) {
                newObj[key] = obj[key];
            }
            // 说明没有,直接舍弃
        }
    }
    return newObj;
};
const only2 = function(obj = {}, keys) {
    if (typeof keys === 'string') {
        keys = keys.split(/ +/);
    }

    return keys.reduce((newObj, key) => {
        if (obj[key] === undefined) {
            // 说明没有,直接舍弃
            return newObj;
        }

        newObj[key] = obj[key];
        return newObj;
    }, {});
};

console.log(only1(o, 'a b'));
console.log(only1(o, 'a b d'));
console.log(only2(o, 'a b'));
console.log(only2(o, 'a b d'));
```

3. compose(pipe的逆向)
```javascript
let funcs = [fn1, fn2, fn3, fn4]
let composeFunc = compose(...funcs)
// compose 就是 
// fn1, fn2, fn3, fn4 
// <=================
fn1(fn2(fn3(fn4(args))))

// pipe 就是
// fn1, fn2, fn3, fn4 
// =================>
fn4(fn3(fn2(fn1(args))))

const pipeOrigin = (...functions) => input => {
    for (var i = 0; i < functions.length; i++) {
        input = functions[i](input);
    }
    return input;
};

const composeOrigin = (...functions) => input => {
    for (var i = functions.length - 1; i >= 0; i--) {
        input = functions[i](input);
    }
    return input;
};

const pipe = (...functions) => input => functions.reduce(
    (accumulator, currentValue) => currentValue(accumulator),
    input
);

const compose = (...functions) => input => functions.reverse().reduce(
    (acc, curr) => curr(acc),
    input
);

// todo 没搞懂为啥要用递归的方法? 更符合直觉?
const composeInBook = function(...args) {
    let length = args.length;
    let count = length - 1;
    let result;
    return function f1(...args1) {
        result = args[count].apply(this, args1);
        if (count <= 0) {
            count = length - 1;
            return result;
        }
        count--;
        return f1.call(null, result);
    };
};
```

总结 pipe 的四种实现方法
1. for 循环
2. reduce 输入值放外面
3. reduce 输入值放里面
4. reduce + Promise
```javascript
const f = function(value) {
    console.log('f 1');
    return value + 1;
};

const g = function(value) {
    console.log('g 2');
    return value + 2;
};

const h = function(value) {
    console.log('h 3');
    return value + 3;
};

// for 循环
const pipe1 = (...functions) => {
    return (initialValue) => {
        for (let i = 0; i < functions.length; i++) {
            initialValue = functions[i](initialValue);
        }
        return initialValue;
    };
};

// reduce + promise
const pipe2 = (...functions) => {
    const initialFunction = functions.shift();
    return (initialValue) => functions.reduce(
        (previousValue, currentFunction) => previousValue.then(result => currentFunction.call(null, result)),
        Promise.resolve(initialFunction.call(null, initialValue))
    );
};

// reduce + 外部 input
const pipe3 = (...functions) => initialValue => functions.reduce(
    (previousValue, currentValue) => currentValue(previousValue),
    initialValue
);

// reduce + 内部 input
const pipe4 = (...functions) => functions.reduce(
    (previousValue, currentValue) => (initialValue) => currentValue(previousValue(initialValue))
);

console.log(pipe1(f, g, h)(1));
pipe2(f, g, h)(1).then(res => console.log('res', res));
console.log(pipe3(f, g, h)(1));
console.log(pipe4(f, g, h)(1));
```

4. apply, bind 的实现

在使用 setTimeout 的时候,一定要注意这里的 xxx 功能有没有提前绑定过 this 或者换成箭头函数的形式调用
```javascript
setTimeout(this.xxx(), 1000)
setTimeout(() => this.xxx(), 1000) // 这样就不用绑定 this 了
```

```javascript
// 模拟一个 bind
// 最简单版本 直接用 call/apply
Function.prototype.needFixBind1 = function(self, ...rest) {
    return () => this.apply(self, rest);
};

// 如果想要实现预设参数传递(也就是里面的 innerRest 的参数),就要把 内部返回的那个函数的参数也加上
Function.prototype.needFixBind2 = function(self, ...rest) {
    return (...innerRest) => this.apply(self, [...rest, ...innerRest]);
};

// 如果要考虑 new 的优先级更高的问题,这里要综合 new 的实现来看
function myNew(constructor, ...args) {
    if (typeof constructor !== 'function') {
        return new Error(constructor + 'must be a function, please fix it')
    }
    // constructor 必须是一个函数
    const temp = Object.create(constructor.prototype);
    const returnResult = constructor.apply(temp, args);
    if ((typeof returnResult === 'object' && returnResult !== null) || typeof returnResult === 'function') {
        // 如果 returnResult 是非 null 的 object 或者 function 的话,可以直接返回 returnResult
        return returnResult;
    } else {
        return temp;
    }
}

Function.prototype.needFixBind3 = function(self, ...outArgs) {
    const me = this
    const F = function(){}
    F.prototype = this.prototype
    
    const bound = function(...innerArgs) {
        return me.apply(this instanceof F ? this : self || this, [...outArgs, ...innerArgs])
    }
    bound.prototype = new F()
    return bound
};

// ES5-shim
Function.prototype.es5ShimBind = function(that) {
    var target = this;
    if (typeof target !== 'function' || Object.prototype.toString.call(target) !== '[object Function]') {
        throw new TypeError('Function.prototype.bind called on incompatible ' + target);
    }
    var args = Array.prototype.slice.call(arguments, 1);

    var binder = function () {
        if (this instanceof bound) {
            var result = target.apply(this, args.concat(Array.prototype.slice.call(arguments)));
            // 对象,数组,函数
            if (Object(result) === result) {
                return result;
            }
            return this;
        } else {
            return target.apply(that, args.concat(Array.prototype.slice.call(arguments)));
        }
    };

    var boundLength = Math.max(0, target.length - args.length);
    var boundArgs = [];
    for (var i = 0; i < boundLength; i++) {
        boundArgs.push('$' + i);
    }

    // magic
    var bound = Function(
        'binder',
        'return function (' + boundArgs.join(',') + '){ return binder.apply(this,arguments); }')(binder);

    if (target.prototype) {
        var Empty = function Empty() {};
        Empty.prototype = target.prototype;
        bound.prototype = new Empty();
        Empty.prototype = null;
    }
    return bound;
};
```

todo
1. 整理 element.offsetTop, element.top 等涉及高度的 api
2. 会导致元素位置偏移的属性有哪些?
