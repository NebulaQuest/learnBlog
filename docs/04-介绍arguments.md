## arguments

- `arguments` 是一个类数组对象，在函数内部可用，它包含了传递给函数的参数信息。

### 类数组对象

arguments 不是真正的数组，但具有类似数组的特性： 为什么不是真正的数组，这是一个历史原因，无需考虑
但是可以通过索引进行访问，可以使用.length,但是数组的方法是不可以使用的

```javascript
    function example(a, b, c) {
        console.log(arguments.length); // 实际传入的参数个数
        console.log(arguments[0]);     // 第一个参数
        console.log(arguments[1]);     // 第二个参数
    }
    
    example(1, 2, 3);
    // 输出: 3, 1, 2
```

### 与命名参数的关系

- 注意非严格模式下生效，严格模式下这种关联是失效的

```javascript
    function test(a, b) {
        console.log("a:", a);           // 1
        console.log("b:", b);           // 2
        console.log("arguments[0]:", arguments[0]); // 1
        console.log("arguments[1]:", arguments[1]); // 2
    
        // 修改命名参数会影响 arguments
        a = 10;
        console.log("修改后 arguments[0]:", arguments[0]); // 10
    
        // 修改 arguments 也会影响命名参数
        arguments[1] = 20;
        console.log("修改后 b:", b); // 20
    }
    
    test(1, 2);
```

### 可以使用的方法

- length

```javascript
    function checkArgs() {
        console.log("参数个数:", arguments.length);
        
        for (let i = 0; i < arguments.length; i++) {
            console.log(`参数 ${i}:`, arguments[i]);
        }
    }
    
    checkArgs(1, "hello", true);
    // 输出:
    // 参数个数: 3
    // 参数 0: 1
    // 参数 1: hello
    // 参数 2: true
```

### 如何转化为真正的数组

- Array.from
  - 这里先介绍一下 `Array.from`
  - 参数
    - arrayLike
      想要转换成数组的类数组或可迭代对象。
    - mapFn 可选
      调用数组每个元素的函数。如果提供，每个将要添加到数组中的值首先会传递给该函数，然后将 mapFn 的返回值增加到数组中。使用以下参数调用该函数：
      - element 当前正在处理的元素 
      - index 处理元素的索引
    - thisArg 可选
      执行 mapFn 时用作 this 的值。
  
```javascript
javascript
function convertToArray() {
    const argsArray = Array.from(arguments);
    console.log(Array.isArray(argsArray)); // true
    
    // 现在可以使用数组方法
    return argsArray.map(x => x * 2);
}

console.log(convertToArray(1, 2, 3)); // [2, 4, 6]
```

- 扩展运算符
```javascript
function convertToArray(...args) {
    console.log(Array.isArray(args)); // true
    return args;
}

// 或者在函数内部使用
function oldWay() {
    const argsArray = [...arguments];
    return argsArray;
}
```

- Array.prototype.slice.call()

1. Array.prototype.slice 方法的核心特点是：它不修改原数组，而是返回一个新数组，并且只需要对象具有 length 属性和数字索引就能工作
2. slice的基本原理
```javascript
// slice 方法的基本工作原理模拟
function simulatedSlice(start, end) {
    const result = [];
    const len = this.length;
    // 计算起始和结束位置
    start = start || 0;
    end = end || len;
    // 处理负数索引
    start = start < 0 ? Math.max(len + start, 0) : Math.min(start, len);
    end = end < 0 ? Math.max(len + end, 0) : Math.min(end, len);
    // 遍历并复制元素
    for (let i = start; i < end; i++) {
        result[result.length] = this[i];
    }
    return result;
}
```

- 我们将Array.prototype.slice.call(arguments) 拆解一下
```javascript
const step1 = Array.prototype.slice; // 获取 slice 方法
const step2 = step1.call(arguments); // 用 call 调用，this 指向 arguments
```

```javascript
function convertToArray() {
    const argsArray = Array.prototype.slice.call(arguments);
    return argsArray;
}
```
- 这里插一句 为什么Array.prototype.slice.call() 和 [].slice.call() 为什么相同
- 原型链继承，[] 是属于 Array.prototype 实例化对象，所以在当前找不到的方法就会向上层去寻找，所以两者是相同的
方法调用本质
```javascript
function explainMethodCall() {
    const array = [1, 2, 3];
    
    // 这两种调用方式本质相同：
    // 方式1: 方法调用
    const result1 = array.slice(1);
    
    // 方式2: 原型方法 + call
    const result2 = Array.prototype.slice.call(array, 1);
    
    console.log("结果相同:", JSON.stringify(result1) === JSON.stringify(result2)); // true
    
    // 证明：方法调用本质上就是原型方法 + 正确this
    console.log("方法调用的本质:");
    console.log("array.slice(1) 等价于:");
    console.log("Array.prototype.slice.call(array, 1)");
}
```

### 现代替代方案

- 剩余参数

剩余参数是一个数组，而不是类数组

```javascript
// 使用剩余参数替代 arguments
function modernFunction(...args) {
    console.log("参数个数:", args.length);
    console.log("是数组吗?", Array.isArray(args));
    
    // 可以直接使用数组方法
    return args.filter(arg => typeof arg === 'number');
}

console.log(modernFunction(1, "hello", 3, true, 5)); // [1, 3, 5]
```

- 使用默认参数

```javascript
// 替代传统的参数检查
function modernExample(a = 1, b = 2, c = 3) {
    return a + b + c;
}

console.log(modernExample());     // 6
console.log(modernExample(10));   // 15
console.log(modernExample(10, 20)); // 33
```

#### 注意

- 箭头函数中没有 arguments 但是可以使用剩余参数 ...args
- 严格模式下的变化
```javascript
"use strict";

function strictExample(a, b) {
    a = 10;
    console.log(arguments[0]); // 1（不会同步修改）
    
    arguments[1] = 20;
    console.log(b); // 2（不会同步修改）
}

strictExample(1, 2);
```
- 不推荐频繁使用arguments