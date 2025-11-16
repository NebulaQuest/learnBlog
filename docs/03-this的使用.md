## this

- 先查看下面这段代码

```javascript
    function identify() {
        return this.name.toUpperCase();
    }
    function speak() {
        var greeting = "Hello, I'm " + identify.call( this );
        console.log( greeting );
    }
    var me = {
        name: "Kyle"
    };
    var you = {
        name: "Reader"
    };
    identify.call( me ); // 1 KYLE
    identify.call( you ); // 2 READER
    speak.call( me ); // 3 Hello, 我是 KYLE
    speak.call( you ); // 4 Hello, 我是 READER
```

1. 先解释call,他可以将函数内部的this绑定到一个实例上，也就是函数内部的this被绑定到后面传入的对象中
2. 首先1中是绑定到me这个对象上，所以函数中的 `this.name.toUpperCase()` 中的就指向了me中的name，也就是kyle
3. 然后2同理
4. 3中首先给speak中绑定了me这个对象，其内部的this指向了me，然后内部 `identify.call( this )` 又将函数identify中的this指向当前函数，也就是me
5. 所以可以理解成两个函数的this都被绑定成了me，所以会输出Hello, 我是 KYLE
6. 4同理

- this 实际上是在函数被调用时发生的绑定，它指向什么完全取决于函数在哪里被调用。

- 首先，默认情况this是指向全局对象也就是window，在**非严格模式下** ，在严格模式下返回的是undefind

例子：
```javascript

    // 例子1
    function foo() {
	    console.log( this.a );
    }
    var obj = {
	    a: 2,
	    foo: foo
    };
    obj.foo(); // 2
    // 例子2
    function foo() {
	    console.log( this.a );
    }
    var obj2 = {
	    a: 42,
	    foo: foo
    };
    var obj1 = {
	    a: 2,
	    obj2: obj2
    };
    obj1.obj2.foo(); // 42
    // 例子3
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
    // 例子4
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
    // 例子5

```
- 例子1中毫无争议，函数foo被obj调用了，那么this指向对应obj
- 例子2中需要解释的是，之后最后一个调用的会改变this的指向
- 例子3中是因为bar引用的是函数本身，所以会直接使用默认的this
- 例子4中，注意看，obj.foo是赋值给函数中的变量fn,调用的是在函数内部，所以指向window

#### 注意

- 例子3和4中都展示了回调函数中this丢失的情况，所以写代码的时候要注意

### 显示绑定

- 使用apply，bind，call 进行绑定，三者的区别是apply接受两个参数，第一个参数是要绑定的this指向，第二个参数是一个数组，数组中的参数是传入函数的形参，调用的时候是立即执行的，call接受多个参数，第一个参数是要绑定的this，后续的参数是传入绑定函数的参数，同时也是立即执行的，bind是接受多个参数，第一个参数是要绑定的this，后续的参数是传入函数的参数，但是不会立即执行，除非手动调用，可以理解成，复制了一个函数
- 使用javaScript的内部API传入参数进行绑定

```javascript
    // 示例
    // 使用call和apply和bind
    const persion1 = {
	  name:'张三',
      age:20
    }
	const persion2 = {
	  name:'李四',
      age:24
    }
    const persion3 = {
        name:'王五',
        age:23
    }
    function showInfo(compoany,salay) {
	    console.log(`${this.name},${this.age}岁，在${compoany}工作，月薪${salay}`)
    }
    showInfo.call(persion1,'阿里巴巴',20000)
    showInfo.apply(persion2,['腾讯',10000])
    let useBind = showInfo.bind(persion3,'字节',30000)
    useBind()

    // JavaScript的API绑定
    function foo(el) {
        console.log( el, this.id );
    }
    var obj = {
        id: "awesome"
    };
    // 调用 foo(..) 时把 this 绑定到 obj
    [1, 2, 3].forEach( foo, obj );
    // 1 awesome 2 awesome 3 awesome
```

### new 绑定

- new 绑定中的this指向当前创建的对象
- 注意构造函数不能使用箭头函数，因为使用了箭头函数会破坏this的指向

```javascript
    function Person(name, age) {
        // new 绑定：this 指向新创建的对象
        this.name = name;
        this.age = age;
        
        this.introduce = function() {
            console.log(`我叫${this.name}，今年${this.age}岁`);
        };
    }
    
    // 使用 new 调用构造函数
    const person1 = new Person('张三', 25);
    const person2 = new Person('李四', 30);
    
    person1.introduce(); // 我叫张三，今年25岁
    person2.introduce(); // 我叫李四，今年30岁
    
    console.log(person1 instanceof Person); // true
    console.log(person2 instanceof Person); // true
```
### 绑定的优先级

- 默认的绑定优先级最低
- 其次是隐式绑定
- 然后是显示绑定

```javascript
    // 隐式绑定和显式绑定比较
    function foo() {
        console.log( this.a );
    }
    var obj1 = {
        a: 2,
        foo: foo
    };
    var obj2 = {
        a: 3,
        foo: foo
    };
    obj1.foo(); // 2
    obj2.foo(); // 3
    obj1.foo.call( obj2 ); // 3
    obj2.foo.call( obj1 ); // 2

    // 隐式绑定和new
    function foo(something) {
        this.a = something;
    }
    var obj1 = {
        foo: foo
    };
    var obj2 = {};
    obj1.foo( 2 );
    console.log( obj1.a ); // 2
    obj1.foo.call( obj2, 3 );
    console.log( obj2.a ); // 3
    var bar = new obj1.foo( 4 );
    console.log( obj1.a ); // 2
    console.log( bar.a ); // 4
```

// TODO 对于new的过程还不了解，没有手写apply，call和bind，没有练习更多的this指向的练习 看到111

