### js应用

![image-20220605211531840](C:\Users\28771\AppData\Roaming\Typora\typora-user-images\image-20220605211531840.png)





### js易错点

![image-20220605211550964](C:\Users\28771\AppData\Roaming\Typora\typora-user-images\image-20220605211550964.png)





### Typescript和JavaScript的关系？

![image-20220605211656725](C:\Users\28771\AppData\Roaming\Typora\typora-user-images\image-20220605211656725.png)



### 浏览器工作原理

![image-20220605211732294](C:\Users\28771\AppData\Roaming\Typora\typora-user-images\image-20220605211732294.png)



### 浏览器内核：WebKit和Blink以及V8引擎之间的关系？

WebKit是苹果公司的浏览器内核，包含WebCore和jsCore两部分。前者负责HTML解析、布局以及dom渲染，后者负责js解析。

Blink是基于WebKit发展而来，由chrome开发，可以进行dom渲染和js解析，内含用于解析JavaScript的V8引擎。



### 浏览器的渲染过程

![image-20220605212158879](C:\Users\28771\AppData\Roaming\Typora\typora-user-images\image-20220605212158879.png)



### V8引擎工作原理

![image-20220605212330732](C:\Users\28771\AppData\Roaming\Typora\typora-user-images\image-20220605212330732.png)

补充：parse阶段包括词法分析和语法分析：

- **词法分析**，词法分析会对代码中的每一个词进行一个切割，切割后最终生成若干个tokens，每个tokens是一个数组结构，每一个数组中存放很多对象：如const name=’xxx‘，词法分析的过程中，const就会被解析成token中的一个键值对，{type:‘keyword’,value:‘const’}，类型为“关键字”，属性值为“const”；name会被解析为token中的另一个键值对，{type:‘identify’,value:‘name’}，类型为“标识符”，属性值为“name”。
- **语法分析**：词法分析后token中就生成了多个对象，且这些对象被划分为了不同的标识（keyword/indentify等）；然后进行语法分析，语法分析将词法分析产生的token，按照某种给定的形式文法转换生成ast（抽象语法树）。



#### 各模块作用

![image-20220605212507628](C:\Users\28771\AppData\Roaming\Typora\typora-user-images\image-20220605212507628.png)

```markdown
小结：要运行的JavaScript脚本会从网络或缓存中被加载。 通过对JavaScript脚本文本的分析可以生成用于描述源代码结构化的数据，抽象语法树（AST）。 接下来Ignition解释器会将AST转化成生成体积更小的字节码，字节码中的每行指令代表着对寄存器的操作，当字节码生后以后AST将会被废弃以节省空间，后续的执行和优化都基于字节码。 在解释器执行字节码时，Object Shapes会试图将代码中对象的类型缓存下来生成Type Feedback，当访问这些对象时会尝试从缓存中获取，如果找不到再动态查找并更新缓存。 TurboFan是V8中的代码优化编译器，它会评估函数是否需要被进一步优化成机器码以提高性能，需要被优化的函数被编译成Optimized Code。 但当编译后的函数被发现函数中变量的数据类型与之前缓存的类型不同时，则需要放弃优化的代码回到字节码重新解释执行。
```



### js执行过程

初始化全局对象GO(内含window属性且指向自身) ---> 开启执行上下文栈ECS ----> 将全局执行上下文GEC放入ECS ----> 执行全局代码 ----> 遇到函数执行 --->  将FEC放入ECS ----> 创造AO对象 -----> 执行函数内部代码

注意：在编译阶段，用var定义的普通变量为undefined，函数变量为地址值，这是因为函数经过编译后则已经在堆内存中创造出了一个函数对象，该函数对象内部保存了prarent-scope属性，即函数的父级作用域在编译阶段便已经确定。函数的执行上下文FEC中存储着作用域链scope-chain：AO + prarent-scope。



**注意：**上面的所有都是基于早期的ECMA规范，规范正文如下：
每一个执行上下文（包含GEC全局执行上下文，用于执行全局代码；FEC函数执行上下文，用于执行函数代码）都会关联一个环境变量**VO**，在源代码中的变量和函数声明会被作为属性添加到**VO**中，对于函数来说，参数也会被添加到**VO**中
在最新的ECMA规范中对一些描述进行了修改如下：
每个执行上下文都会关联到一个**变量环境**（variable environment）（从环境变量修改为了变量环境），在执行代码中变量和函数的声明都会作为 **环境记录**（Enviroment Record）添加到**变量环境**中，对于函数来说，参数也会被作为**环境记录**添加到**变量环境**中



### 作用域提升面试题

面试题1

```js
var n = 100

function foo1(){
    console.log(n);   //100,父级作用域在编译阶段就已经确定，找的是GO中的n
}

function foo2(){
    var n = 200
    console.log(n);   //200，找的是foo2的AO对象中的n
    foo1()
}

foo2()
console.log(n);       //100,全局执行上下文中的代码，找的是GO中的n

```

面试题2

```js
var a = 100
function foo(){
    console.log(a);
    return 
    var a = 100      //在函数代码编译阶段AO中会产生一个a，值为undefined，这就是我们常说的变量提升，即return不影响代码编译
}

foo()
```

面试题3

```js
function foo(){
    var a = b = 10   //本题的考察点，这行代码相当于执行了b = 10;var a = 10;前者会产生全局变量，后者产生的是局部变量
}

foo()

console.log(a);      //全局对象AO中没有a，因此会报错
console.log(b);      //全局对象AO中有b，值为10
```





### 函数式编程

简单来说就是函数作为一等公民的一种编程范式。

- 那么就意味着函数的使用是非常灵活的；

- 函数可以作为另外一个函数的参数，也可以作为另外一个函数的返回值来使用；

### 高阶函数

高阶函数可以用另一个函数作为其输入参数，在某些情况下，它甚至返回一个函数作为其输出参数。

### 闭包

嵌套的内部函数访问了外层作用域的变量，就产生了闭包。

闭包的产生可能会导致内存泄漏。即外层作用域的AO对象不会被释放。

问：AO对象不能被销毁，是否里面所有属性都不会被释放？

答：V8引擎的优化会使AO对象中未使用到的属性正常销毁而不受闭包影响。

### 垃圾回收GC

#### 两种垃圾回收算法

- **引用计数**     -----   缺点：无法解决循环引用的问题
- **标记清除**     -----   设置一个根对象，垃圾回收器会定期从根对象出发，找到所有能找到的对象的引用，如果不可达的对象就会被清除





### this

函数中this的四种绑定方式

- 默认绑定    ----     函数独立调用，this指向全局对象
- 隐式绑定    ----     通过对象调用方法，如obj.foo()，this为调用对象
- 显示绑定    ----     通过call、apply、bind绑定this
- new关键字绑定   



#### call、apply、bind关系

- call第一个参数为要绑定的this，后面的剩余参数为要传递给函数的实际参数
- apply第一个参数为要绑定的this，第二个参数为数组，内含要传递给函数的实际参数
- bind第一个参数同样为要绑定的this，后面的剩余参数为要传递给函数的实际参数，不同的是bind函数会返回一个新的函数



### new绑定过程

1. 创建一个全新的对象；
2. 这个新对象会被执行prototype连接，即新对象的\_\_proto\_\_属性会指向构造函数原型对象；
3. 这个新对象会绑定到函数调用的this上（this的绑定在这个步骤完成）；
4. 如果函数没有返回其他对象，表达式会返回这个新对象；



```js
//手动实现new关键字
function _new(func, ...rest) {
    // 创建空对象，并将新对象的__proto__属性指向构造函数的prototype
    const obj = Object.create(func.prototype)
    // 执行构造函数，改变构造函数的this指针，指向新创建的对象（新对象也就有了构造函数的所有属性）
    func.apply(obj, rest)
    return obj
}


```



### 规则优先级

new绑定  >    显示绑定(bind)   > 隐式绑定  >  默认绑定

补充：new绑定与call、apply不能同时使用



### 特殊规则

- 忽略显示绑定：call和apply若接收的thisArg参数为null或undefined，在非严格模式下则会自动绑定为全局对象(window)

- 忽略显示绑定：如果bind函数的参数列表为空或thisArg为null或undefined，则执行作用域的this将被视为新函数的thisArg。

- 间接函数引用：(obj2.foo = obj1.foo)返回的是foo函数，相当于函数独立调用，this指向全局对象

  ```js
  var obj1 = {
      name:"obj1",
      foo:function(){
          console.log(this.name);
      }
  }
  
  var obj2 = {
      name:"obj2"
  }
  
  obj1.foo();       //obj1
  (obj2.foo = obj1.foo)()     //window，相当于独立函数调用
  ```


- 箭头函数（arrow function）：箭头函数没有this，会沿着作用域链找到外层作用域的this

面试题一：

```javascript
var name = "window"
var person = {
    name:"person",
    sayName:function(){
        console.log(this.name);
    }
};

function sayName(){
    var sss = person.sayName
    sss();    //相当于函数独立调用，this指向window
    person.sayName();   //隐式绑定，this指向person
    (person.sayName)();  //解析后和上一句效果一样，属于隐式绑定
    (b = person.sayName)();   //相当于函数独立调用，this指向window
}

sayName()
```

面试题二：

```javascript
var name = "window"
var person1 = {
    name:"person1",
    foo1:function(){
        console.log(this.name);
    },
    foo2:() => {
      console.log(this.name);  
    },
    foo3:function(){
        return function(){
            console.log(this.name);
        }
    },
    foo4:function(){
        return () => {
            console.log(this.name);
        }
    }
}
var person2 = {
    name:"person2"
}

person1.foo1()    //person1,隐式绑定
person1.foo1.call(person2)   //person2，显式绑定

person1.foo2()   //window，箭头函数没有this，向外层作用域查找(对象本身不产生作用域，向外直接找到window)
person1.foo2.call(person2)   //window，箭头函数不能绑定this，因为箭头函数没有this，需要向外层作用域查找

person1.foo3()()   //window，相当于内部独立函数调用
person1.foo3.call(person2)()  //window，相当于内部函数独立调用
person1.foo3().call(person2)  //person2，内部函数显式绑定this为person2

person1.foo4()()  //person1，外层函数隐式绑定，this为person1，内部函数为箭头函数没有this，向外层作用域查找找到person1
person1.foo4.call(person2)()  //person2，外层函数显式绑定this为person2，内层函数为箭头函数没有this，向外找到person2
person1.foo4().call(person2)  //person1，外层函数隐式绑定，this为person1，内层箭头函数call不起作用，向外找到person1
```

面试题三：

```javascript
var name = "window"
function Person(name){
    this.name=name
    this.foo1=function(){
        console.log(this.name);
    }
    this.foo2=() => {
      console.log(this.name);  
    }
    this.foo3=function(){
        return function(){
            console.log(this.name);
        }
    }
    this.foo4=function(){
        return () => {
            console.log(this.name);
        }
    }
}

var person1 = new Person("person1")
var person2 = new Person("person2")

person1.foo1()  person1.foo1.call(person2)   //person2，显式绑定

person1.foo2()   //person1，内层函数为箭头函数没有this，向外找，外层this为person1的隐式绑定
person1.foo2.call(person2)   //person1，外层this为person1

person1.foo3()()   //window，内层函数为独立函数调用
person1.foo3.call(person2)()  //window，内层函数独立函数调用
person1.foo3().call(person2)  //person2，内层函数调用this显式绑定

person1.foo4()()  //person1，内层函数为箭头函数没有this，向外找，外层this为person1
person1.foo4.call(person2)()  //person2，内层函数为箭头函数没有this，向外找，外层this为person2
person1.foo4().call(person2)  //person1，内层函数为箭头函数没有this，向外找，外层this为person1
```

注意：对象本身var obj = { }不产生作用域，与es6代码块 { }产生的块级作用域有区别。

面试题四：

```javascript
var name = "window"
function Person(name){
    this.name = name
    this.obj = {
        name:'obj',
        foo1:function(){
            return function(){
                console.log(this.name);
            }
        },
        foo2:function(){
            return () => {
                console.log(this.name);
            }
        }
    }
}

var person1 = new Person('person1')
var person2 = new Person('person2')

person1.obj.foo1()()    //window,内层函数独立调用
person1.obj.foo1.call(person2)()    //window,内层函数独立调用
person1.obj.foo1().call(person2)    //person2,内层函数显示绑定

person1.obj.foo2()()   //obj,外层函数隐式绑定this为obj,内层函数为箭头函数向外找this为obj
person1.obj.foo2.call(person2)()    //person2,外层函数显式绑定this为person2,内层函数为箭头函数向外找this为person2
person1.obj.foo2().call(person2)    //obj,外层函数隐式绑定this为obj,内层函数为箭头函数向外找this为obj
```





### 手写call、apply、bind

**手写call**

```js
Function.prototype.myCall = function(thisArg,...args){
    //判断传入的this参数是否为null或undefined，若是则默认为window，若不是则转为对应的对象类型
    thisArg = (thisArg !== null && thisArg !== undefined) ? Object(thisArg) : window
    //使用Symbol类型防止属性名冲突
    let fn = Symbol('fn')
    //进行函数类型判断
    if(typeof this !== 'function'){
        throw new TypeError('error')
    }
    //这里的this是调用myCall的函数
    thisArg[fn] = this
    let result = thisArg[fn](...args)
    //将新添加的属性删除
    delete thisArg[fn]

    //若函数有返回值则将函数执行结果返回
    return result
}

  function sum(n, m) {
    console.log(this);
    return n + m;
  }
  function foo(a){
    console.log(this,a);
  }

  // console.log(sum.call(true,10,20));
  // console.log(sum.myCall(true, 10, 20));
  console.log(foo.call(undefined));
  console.log(foo.myCall(undefined));
```

**手写apply**

```javascript
Function.prototype.myApply = function(thisArg,argsArray){
    //判断传入的this参数是否为null或undefined，若是则默认为window，若不是则转为对应的对象类型
    thisArg = (thisArg !== null && thisArg !== undefined) ? Object(thisArg) : window
    //使用Symbol类型防止属性名冲突
    let fn = Symbol('fn')
    //进行函数类型判断
    if(typeof this !== 'function'){
        throw new typeError('error')
    }
    //这里的this是调用myApply的函数
    thisArg[fn] = this
    //由于apply函数接收参数是以数组形式接收的，因此要防止未传的情况,未传则相当于空数组(...[]得到的是undefined)
    let args = argsArray ? [...argsArray] : []
    let result = thisArg[fn](...args)
    //将新添加的属性删除
    delete thisArg[fn]

    return result
}

function sum(n, m) {
    console.log(this);
    return n + m;
  }
  function foo(){
    console.log(this);
    console.log(arguments);
  }

//   console.log(sum.apply(true,[10,20]));
//   console.log(sum.myApply(true,[ 10, 20]));
  console.log(foo.apply('abc'));
  console.log(foo.myApply('abc'));
```

**手写bind**

```javascript
Function.prototype.myBind = function(thisArg,...args1){
    //判断传入的this参数是否为null或undefined，若是则默认为window，若不是则转为对应的对象类型
    thisArg = (thisArg !== null && thisArg !== undefined) ? Object(thisArg) : window
    //进行函数类型判断
    if(typeof this !== 'function'){
        throw new TypeError('error')
    }
    //使用Symbol类型防止属性名冲突
    let fn = Symbol('fn')
    //这里的this是调用myBind的函数
    thisArg[fn] = this
    function proxyFn(...args2){  
        //参数为两个部分参数拼接而成
        let result = thisArg[fn](...args1,...args2)
        delete thisArg[fn]
        return result
    }
    //返回一个新的函数
    return proxyFn
}

function sum(num1,num2,num3,num4){
    console.log(this);
    return num1 + num2 + num3 + num4
}

let newSum1 = sum.bind('abc',10,20)
let newSum2 = sum.myBind('abc',10,20)
console.log(newSum1(30,40));
console.log(newSum2(30,40));
```





### arguments

**arguments常见的操作：**

1. 获取参数的长度， `arguments.length` ，注意是实参的长度，函数名.length获取到的是形参的长度
2. 根据索引值获取某一个参数， `arguments[2]` 
3. callee获取当前arguments所在的函数（用的较少）， `arguments.callee` 

注意：arguments是一个类数组，不具有数组的foreach、slice等方法。

**将arguments转成数组的方法：**

- for循环遍历arguments中所有元素
- 利用数组的slice方法，` var newArr2 = Array.prototype.slice.call(arguments)` 
- ES6语法，` var newArr4 = Array.from(arguments)` ，` var newArr5 = [...arguments]` 

自己实现一个slice方法：

```js
Array.prototype.hyslice = function(start, end) {
  var arr = this
  start = start || 0
  end = end || arr.length
  var newArray = []
  for (var i = start; i < end; i++) {
    newArray.push(arr[i])
  }
  return newArray
}
```

**箭头函数不绑定arguments，所以我们在箭头函数中使用arguments会去上层作用域查找。**





### 纯函数

- 确定的输入，一定会产生确定的输出； 

- 函数在执行过程中，不能产生副作用；



### 函数柯里化(Currying)

只传递给函数一部分参数来调用它，让它返回一个函数去处理剩余的参数； 这个过程就称之为柯里化；

示例：

```JavaScript
//原函数
function add(x, y, z) {
  return x + y + z
}

//柯里化之后的函数
function sum1(x) {
  return function(y) {
    return function(z) {
      return x + y + z
    }
  }
}

// 简化柯里化的代码
var sum2 = x => y => z => {
  return x + y + z
}
```

**函数柯里化的作用**

- **让函数的职责单一**，我们不希望一个函数处理过多过复杂的逻辑，这样不便于维护
- **实现参数逻辑复用**，当一个函数的部分参数较为固定为某个数时，我们可以将该参数固定为某个值并返回一个新函数，如add5()函数



**手动实现自动柯里化函数**

```javascript
// 柯里化函数的实现hyCurrying
function hyCurrying(fn) {
  function curried(...args) {
    // 判断当前已经接收的参数的个数, 可以参数本身需要接受的参数是否已经一致了
    // 1.当已经传入的参数 大于等于 需要的参数时, 就执行函数
    if (args.length >= fn.length) {
      // fn(...args)
      // fn.call(this, ...args)
      return fn.apply(this, args)    //保证this指向正确
    } else {
      // 没有达到个数时, 需要返回一个新的函数, 继续来接收的参数
      function curried2(...args2) {
        // 接收到参数后, 需要递归调用curried来检查函数的个数是否达到
        return curried.apply(this, args.concat(args2))
      }
      return curried2
    }
  }
  return curried
}
```



**组合函数**

当我们需要调用多个函数，前一个函数的返回值需要作为后一个函数的参数，即多个函数依次调用时，我们就可以实现一个组合函数完成多个函数依次调用的功能。

示例：

```javascript
function double(num) {
  return num * 2
}

function square(num) {
  return num ** 2
}

// 实现最简单的组合函数
function composeFn(m, n) {
  return function(count) {
    return n(m(count))
  }
}
```



**手动实现通用的组合函数**

```javascript
function hyCompose(...fns) {
  var length = fns.length
  //判断参数类型，必须为function
  for (var i = 0; i < length; i++) {
    if (typeof fns[i] !== 'function') {
      throw new TypeError("Expected arguments are functions")
    }
  }

  function compose(...args) {
    var index = 0
    var result = length ? fns[index].apply(this, args): args  //初始化index、result
    while(++index < length) {
      result = fns[index].call(this, result)    //保证this指向正确
    }
    return result
  }
  return compose
}
```





### JS补充

**with语句（不建议使用）**

将代码的作用域设置到一个特定的对象中。

语法格式：

```js
with (expression) {
 
    statement;
 
}
```

实例：

```javascript
var foo = 1;
var bar = {
    foo: 2
}

with(bar) {
    console.log(foo);	 //2
    foo = 3
    console.log(foo);	 //3
    var foo = 4          //这里用不用var效果一样，操作的都是bar中的foo
    console.log(foo); 	//4
}
console.log(bar.foo);	 //4

console.log(foo); 	//1
if (true) {
    foo = 5;
}
console.log(foo); 	//5
```



**eval函数（不建议使用）**

eval是一个特殊的函数，它可以将传入的字符串当做JavaScript代码来运行。（不产生新的作用域）

缺点：

- 代码可读性差
- eval是一个字符串，那么有可能在执行的过程中被刻意篡改，那么可能会造成被攻击的风险；
- eval的执行必须经过JS解释器，不能被JS引擎优化；



### 严格模式

**开启严格模式**  **“use strict”**

- 可以对一个js文件开启严格模式
- 也可以对一个函数单独开启严格模式

**严格模式下的限制**

**8. 严格模式下，this绑定不会默认转成对象**

```js
// 1. 禁止意外创建全局变量
// message = "Hello World"
// console.log(message)

// function foo() {
//   age = 20
// }

// foo()
// console.log(age)

// 2.不允许函数有相同的参数名称
// function foo(x, y, x) {   非严格模式下相当于后面的x会覆盖前面，输出为30，20，30
//   console.log(x, y, x)
// }

// foo(10, 20, 30)


// 3.静默错误，用抛出异常来处理非严格模式下的静默错误
// true.name = "abc"
// NaN = 123
// var obj = {}
// Object.defineProperty(obj, "name", {
//   configurable: false,   //不可配置属性不能delete
//   writable: false,      //只读属性不能修改
//   value: "why"
// })
// console.log(obj.name)
// // obj.name = "kobe"

// delete obj.name

// 4.不允许使用原先的八进制格式 0123
// var num = 0o123 // 八进制
// var num2 = 0x123 // 十六进制
// var num3 = 0b100 // 二进制
// console.log(num, num2, num3)

// 5.with语句不允许使用

// 6.eval函数不会向上引用变量了
//var jsString = '"use strict"; var message = "Hello World!" ;console.log(message);'
//eval(jsString)

//console.log(message)  严格模式下eval语句中的var message并不会创建出全局变量

//7.严格模式下试图删除不可删除的属性
/**
* 严格模式下试图删除不可删除的属性会直接报错，非严格模式下相当于不起作用但不会报错
*/

//8.严格模式下this不再默认指向window，而是undefined。
//"use strict"
//function foo(){
//  console.log(this);
//}
//foo()  //undefined，即自执行的函数this不再指向window，而是指向undefined
```







### 面向对象

面向对象编程和函数式编程一样，都属于一种编程范式，JavaScript支持这两种编程范式。



创建对象的两种方式

```js
//1.字面量方式
var obj = {
	a:1,
	b:[],
	c:'abc'
}

//2.通过new Object创建
var obj = new Object()
obj.a = 1
obj.b = []
obj.c = 'abc'
```



#### **属性描述符**

如果我们想要对一个属性进行比较精准的操作控制，那么我们就可以使用属性描述符。通过属性描述符可以精准的添加或修改对象的属性。属性描述符需要使用 **Object.defineProperty** 来对属性进行添加或者修改

**Object.defineProperty()** 方法会直接在一个对象上定义一个新属性，或者修改一个对象的现有属性，并返回此对象。

语法格式：

```js
Object.defineProperty(obj,prop,descriptor)    //返回值为obj
```

**两种属性描述符（descriptor）**

- 数据属性描述符
- 存取属性描述符

![image-20220608183219176](C:\Users\28771\AppData\Roaming\Typora\typora-user-images\image-20220608183219176.png)

属性描述符中各属性的含义：

- **configurable**：表示是否可配置，即属性是否可以通过delete删除属性，是否可以修改它的特性，或者是否可以将它修改为存取属性

  描述符；

- **Enumerable**：表示是否可枚举，即是否可以通过for-in或者Object.keys()返回该属性；

- **Writable**：表示是否可写，即属性是否能被修改

- **value**：即属性的value值

- **get**：获取属性时会执行的函数。默认为undefined

- **set**：设置属性时会执行的函数。默认为undefined

注意：**在不使用属性描述符给对象添加属性值时，configurable、enumerable、writable属性值默认为true。使用属性描述符时默认值为false。**

示例：

```js
// 数据属性描述符
// 用了属性描述符, 那么会有默认的特性
Object.defineProperty(obj, "address", {
  value: "北京市", // 默认值undefined
  //该特性表示不可删除,也不可以重新定义属性描述符
  configurable: false, // 默认值false
  // 该特性是配置对应的属性(address)是否是可以枚举
  enumerable: true, // 默认值false
  // 该特性是属性是否是可以赋值(写入值) 
  writable: false // 默认值false
})

// 存取属性描述符
// 1.隐藏某一个私有属性被希望直接被外界使用和赋值
// 2.如果我们希望截获某一个属性它访问和设置值的过程时, 也会使用存储属性描述符
Object.defineProperty(obj, "address", {
  enumerable: true,
  configurable: true,
  get: function() {
    foo()
    return this._address
  },
  set: function(value) {
    bar()
    this._address = value
  }
})
```



**定义多个属性描述符**

语法格式：

```js
Object.defineProperties(obj,{
	name:{

	},
	age:{

	},
	……
})
```

示例：

```js
Object.defineProperties(obj, {
  name: {
    configurable: true,
    enumerable: true,
    writable: true,
    value: "why"
  },
  age: {
    configurable: true,
    enumerable: true,
    get: function() {
      return this._age
    },
    set: function(value) {
      this._age = value
    }
  }
})
```



**获取对象的属性描述符**

```js
// 获取某一个特性属性的属性描述符
console.log(Object.getOwnPropertyDescriptor(obj, "name"))
console.log(Object.getOwnPropertyDescriptor(obj, "age"))

// 获取对象的所有属性描述符
console.log(Object.getOwnPropertyDescriptors(obj))
```



**Object对象方法的补充**

```js
// 1.禁止对象继续添加新的属性

Object.preventExtensions(obj)

// 2.禁止对象配置/删除里面的属性。Object.seal()方法被用来封闭一个对象，阻止添加新属性并将所有现有属性标记为不可配置。当前属性的值只要可写就可以改变, 并且返回一个被密封的对象的引用.
Object.seal(obj)

// 3.让属性不可以修改(writable: false)
Object.freeze(obj)
```





**复习new操作符的执行过程**

1. 在内存中创建一个新的对象（空对象）；

2. 这个对象内部的[[prototype]]属性会被赋值为该构造函数的prototype属性；（后面详细讲）；

3. 构造函数内部的this，会指向创建出来的新对象；

4. 执行函数的内部代码（函数体代码）；

5. 如果构造函数没有返回非空对象，则返回创建出来的新对象；



### 创建多个对象的方法

#### 工厂函数

```js
// 工厂模式: 工厂函数
function createPerson(name, age, height, address) {
  var p = {}
  p.name = name
  p.age = age
  p.height = height;
  p.address = address

  p.eating = function() {
    console.log(this.name + "在吃东西~")
  }

  p.running = function() {
    console.log(this.name + "在跑步~")
  }

  return p
}
```

缺点：创建出来的对象没有类型



#### 构造函数

```js
// 规范: 构造函数的首字母一般是大写
function Person(name, age, height, address) {
  this.name = name
  this.age = age
  this.height = height
  this.address = address

  this.eating = function() {
    console.log(this.name + "在吃东西~")
  }

  this.running = function() {
    console.log(this.name + "在跑步")
  }
}
```

缺点：对象内部方法重复创建

解决方法：将方法添加到对象原型上





### 原型和原型链

![image-20220608210202576](C:\Users\28771\AppData\Roaming\Typora\typora-user-images\image-20220608210202576.png)

注：在控制台打印一个对象时，该对象的类型实际上打印的是其原型链上的constructor属性。





### ES5实现继承

**原型链实现继承**

```js
function Person(){
    this.name = "why"
    this.friends = []
}

function Student(){
    this.sno = 111
}

var p = new Person()
Student.prototype = p
```

弊端：

1. 第一个弊端: 打印stu对象, 继承的属性是看不到的，直接修改对象上的属性, 是给本对象添加了一个新属性

2. 第二个弊端: 创建出来两个stu的对象，引用类型属性指向同一块内存地址

3. 第三个弊端: 在前面实现类的过程中都没有传递参数



**借用构造函数继承**

```js
// 子类: 特有属性和方法
function Student(name, age, friends, sno) {
  Person.call(this, name, age, friends)    //调用父类的构造函数，this为子类实例对象
  this.sno = 111
}

var p = new Person()
Student.prototype = p
```

弊端:

1.第一个弊端: Person函数至少被调用了两次

2.第二个弊端: stu的原型对象上会多出一些属性, 但是这些属性是没有存在的必要





补充Object有关原型的方法：

1. **`Object.create()`** 方法创建一个新对象，使用现有的对象来提供新创建的对象的 \__proto__。

   ```js
   const me = Object.create(person);
   //相当于me.__proto__ = person
   ```

2. **`Object.setPrototypeOf()`** 方法设置一个指定的对象的原型 ( 即，内部 [[Prototype]] 属性）到另一个对象或  `null` 。

   ```js
   Object.setPrototypeOf(newObj, obj)
   //相当于newObj.__proto__ = obj
   ```

   注：MDN上说由于JS引擎优化，建议不再修改对象的prototype属性。



**原型式继承**

```js
// 原型式继承函数,两种写法效果一样，都是实现了newObj.__proto__  = obj
function createObject1(obj) {
  var newObj = {}
  Object.setPrototypeOf(newObj, obj)
  return newObj
}

function createObject2(obj) {   
  function Fn() {}
  Fn.prototype = obj
  var newObj = new Fn()
  return newObj       
}
```



**寄生式继承**

```js
var personObj = {
  running: function() {        //相当于在原型上添加方法
    console.log("running")
  }
}

function createStudent(name) {
  var stu = Object.create(personObj)   //实现stu.__proto__ = personObj
  stu.name = name
  stu.studying = function() {    //给对象本身添加方法
    console.log("studying~")
  }
  return stu
}
```



**组合式继承**

```js
//继承函数
function inheritPrototype(SubType, SuperType) {
  SubType.prototype = Objec.create(SuperType.prototype)  //SubType.prototype.__proto__ = SuperType.prototype
  Object.defineProperty(SubType.prototype, "constructor", {
    enumerable: false,                      //将SubType.prototype的constructor属性指向子类SubType本身
    configurable: true,
    writable: true,
    value: SubType
  })
}
//定义父类
function Person(name, age, friends) {
  this.name = name
  this.age = age
  this.friends = friends
}
//父类原型上的方法
Person.prototype.running = function() {
  console.log("running~")
}
//定义子类
function Student(name, age, friends, sno, score) {
  Person.call(this, name, age, friends)
  this.sno = sno
  this.score = score
}
//子类原型上的方法
Student.prototype.studying = function() {
  console.log("studying~")
}
//实现继承关系
inheritPrototype(Student, Person)
```





### JS原型补充

- **hasOwnProperty()方法判断**

  判断对象本身是否含有某个属性

- **in 操作符**

  判断某个对象的原型链上是否存在某个属性，不管在当前对象还是原型中返回的都是true

- **for in**

  枚举对象身上所有可枚举的属性

- **instanceof**

  判断后一个**对象的原型对象**是否在前一个对象的原型链上

  ```js
  console.log(stu instanceof Student) // true
  console.log(stu instanceof Person) // true
  console.log(stu instanceof Object) // true
  ```

- **isPrototypeOf**

  判断一个**对象**是否在另一个对象的原型链上

  ```js
  var p = new Person()
  
  console.log(p instanceof Person)   //true
  console.log(Person.prototype.isPrototypeOf(p))   //true
  ```

  



### Class类

ES6语法中的class类与ES5中的构造函数的特性是一致的。Class类的继承通过babel工具转成ES5语法实际上是通过组合式继承实现的。可以看成是一种语法糖。

**类的访问器方法**

```js
class Person {
  constructor(name, age) {
    this.name = name
    this.age = age
    this._address = "广州市"
  }
  // 类的访问器方法
  get address() {
    console.log("拦截访问操作")
    return this._address       //默认写法
  }

  set address(newAddress) {
    console.log("拦截设置操作")
    this._address = newAddress
  }
    
  // 类的静态方法(类方法)--------可以直接通过类名调用
  // Person.createPerson()
  static randomPerson() {
    var nameIndex = Math.floor(Math.random() * names.length)
    var name = names[nameIndex]
    var age = Math.floor(Math.random() * 100)
    return new Person(name, age)
  }
}


//ES6实现继承使用extends关键字
class Student extends Person{
    
}

```



#### super关键字

在子（派生）类的构造函数中使用this或者返回默认对象之前，必须先通过super调用父类的构造函数！ 

super的使用位置有三个：

1. 子类的构造函数
2. 子类重写父类的实例方法
3. 子类重写父类的静态方法



```js
class Person {
  constructor(name, age) {
    this.name = name
    this.age = age
  }
    
  personMethod() {
    console.log("处理逻辑1")
    console.log("处理逻辑2")
    console.log("处理逻辑3")
  }

  static staticMethod() {
    console.log("PersonStaticMethod")
  }
}

// Student称之为子类(派生类)
class Student extends Person {
  // JS引擎在解析子类的时候就有要求, 如果我们有实现继承
  // 那么子类的构造方法中, 在使用this之前必须使用super关键字调用父类的构造函数
  constructor(name, age, sno) {
    super(name, age)        //使用super位置一：子类构造函数
    this.sno = sno
  }
  
    // 重写personMethod方法
  personMethod() {
    // 复用父类中的处理逻辑
    super.personMethod()    //使用super位置二：重写实例方法

    console.log("处理逻辑4")
    console.log("处理逻辑5")
    console.log("处理逻辑6")
  }

  // 重写静态方法
  static staticMethod() {
    super.staticMethod()    //使用super位置三：重写静态方法
    console.log("StudentStaticMethod")
  }
}
```





### ES6相关语法

#### 字面量增强写法

- 属性的简写

  ```js
  let name = 'zs'
  const obj = {
      name
  }
  ```

- 函数的简写

  ```js
  const obj = {
      bar(){
          
      }
  }
  ```

- 计算属性名

  ```js
  obj[name + 123] = 'hahha'
  ```



#### 数组的结构赋值

```js
var names = ["abc", "cba", "nba"]

// 对数组的解构: []
var [item1, item2, item3] = names
console.log(item1, item2, item3)      //abc cba nba

// 解构后面的元素
var [, , itemz] = names
console.log(itemz)                  //nba

// 解构出一个元素,后面的元素放到一个新数组中
var [itemx, ...newNames] = names
console.log(itemx, newNames)          //abc [ 'cba', 'nba' ]

// 解构的默认值
var [itema, itemb, itemc, itemd = "aaa"] = names
console.log(itema, itemb, itemc,itemd)     //abc cba nba aaa

```



#### 对象的解构赋值

```js
var obj = {
  name: "why",
  age: 18,
  height: 1.88
}

// 对象的解构: {}
var { name, age, height } = obj
console.log(name, age, height)

var { age } = obj
console.log(age)

var { name: newName } = obj
console.log(newName)
//解构赋值的同时自定义变量名并赋默认值
var { address: newAddress = "广州市" } = obj
console.log(newAddress)

function bar({name, age}) {
  console.log(name, age)
}

bar(obj)
```



#### let/const

- 会产生块级作用域
- 不能重复声明
- const定义的变量不能被修改，所以声明时必须初始化

**关于let/const是否存在作用域提升？**

通过查阅官方文档可知，在执行上下文的词法环境被创建出来的时候，let/const声明的变量其实已经被创建了，但是不可以被访问。

**作用域提升**：在声明变量的作用域中，如果这个变量可以在声明之前被访问，那么我们可以称之为作用域提升；

由作用域提升的定义可知，let/const并不存在作用域提升，但是会在解析阶段被创建出来。



**let/const创建的变量如何保存？**

在最新的ECMA规范中，不再使用VO、GO的说法，而是采用变量环境VE的说法。

![image-20220626175947721](C:\Users\28771\AppData\Roaming\Typora\typora-user-images\image-20220626175947721.png)

也就是说在最新的ECMA规范中，我们声明的变量和环境记录是被添加到变量环境VE中的。不同的JS引擎对VE的实现不同，在v8引擎中，是通过一个VaribleMap的hashmap来存储的。对于window对象，在以前的ECMA规范中，GO对象和window对象指向的是同一个对象。在全局环境下定义的变量是作为window的属性存在的。而在最新的ECMA规范中，window其实是浏览器添加的一个全局对象，并且一直保持了window和var之间值的相等性（为了兼容旧代码）。

![image-20230318030642680](C:\Users\28771\AppData\Roaming\Typora\typora-user-images\image-20230318030642680.png![image-20230318031605787](C:\Users\28771\AppData\Roaming\Typora\typora-user-images\image-20230318031605787.png)

参考链接：[JS夯实之执行上下文与词法环境](https://juejin.cn/post/6844904145372053511)





#### 块级作用域

在ES6中新增了块级作用域，并且通过let、const、function、class声明的标识符是具备块级作用域的限制的：

```js
// ES6的代码块级作用域
// 对let/const/function/class声明的类型是有效
{
  let foo = "why"
  function demo() {
    console.log("demo function")
  }
  class Person {}
}

// console.log(foo) // foo is not defined
// 不同的浏览器有不同实现的(大部分浏览器为了兼容以前的代码, 让function是没有块级作用域)
// demo()    //我们发现函数拥有块级作用域，但是外面依然是可以访问的，这是为了兼容旧代码
var p = new Person() // Person is not defined

```

if、switch、for循环语句中的大括号也会形成块级作用域。

易错点：注意区分代码块{ }与对象obj = { }的区别：前者内部是js代码，后者内部是对象的属性及属性值

**暂时性死区**：使用let/const声明的变量在声明之前不允许访问。



#### 模板字符串

```js
let age = 18
console.log(`小明今年${age}岁`)
```

**标签模板字符串**：用于函数调用，即f()和f``都可以起到调用函数的作用

```js
function foo(m, n, x) {
  console.log(m, n, x, '---------')
}

const name = "why"
const age = 18
// ['Hello', 'Wo', 'rld']
foo`Hello${name}Wo${age}rld`

// 第一个参数依然是模块字符串中整个字符串, 只是被切成多块,放到了一个数组中，即m = ['Hello', 'Wo', 'rld']
// 第二个参数是模块字符串中, 第一个 ${}，即n = 'why',x = '18'
```



#### 函数的默认参数

```js
// 1.ES6可以给函数参数提供默认值
function foo(m = "aaa", n = "bbb") {
  console.log(m, n)
}

// 2.对象参数和默认值以及解构
function printInfo({name, age} = {name: "why", age: 18}) {
  console.log(name, age)
}

// 另外一种写法
function printInfo1({name = "why", age = 18} = {}) {
  console.log(name, age)
}

// 3.有默认值的形参最好放到最后，JavaScript允许不将其放到最后，但是意味着还是会按照顺序来匹配
function bar(x, y, z = 30) {
  console.log(x, y, z)
}

// 4.有默认值的函数的length属性
function baz(x, y, z, m, n = 30) {
  console.log(x, y, z, m, n)
}

console.log(baz.length)    //4，即默认值会改变函数的length的个数，默认值以及后面的参数都不计算在length之内了
```



#### 剩余参数

```js
function foo(m,n,...args){
    console.log(m,n);
    console.log(args);
}
```

- 剩余参数只包含那些没有对应形参的实参，而 arguments 对象包含了传给函数的所有实参

- arguments对象不是一个真正的数组，而rest参数是一个真正的数组，可以进行数组的所有操作

- 剩余参数必须放到最后一个位置，否则会报错



#### 展开语法

- 可以在函数调用/数组构造时，将数组表达式或者string在语法层面展开；

- 还可以在构造字面量对象时, 将对象表达式按key-value的方式展开；

使用场景如下：

```js
const names = ["abc", "cba", "nba"]
const info = {name: "why", age: 18}

// 1.函数调用时
function foo(x, y, z) {
  console.log(x, y, z)
}

// foo.apply(null, names)
foo(...names)

// 2.构造数组时
const newNames = [...names]
console.log(newNames)

// 3.构建对象字面量时ES2018(ES9)
const obj = { ...info, address: "广州市", ...names}
console.log(obj)  //结果如下
// {
//   '0': 'abc',
//   '1': 'cba',
//   '2': 'nba',
//   name: 'why',
//   age: 18,
//   address: '广州市'
// }
```

注意：展开运算符其实是一种浅拷贝；

```js
const info = {
  name: "why",
  friend: { name: "kobe" }
}

const obj = { ...info, name: "coderwhy" }
// console.log(obj)
obj.friend.name = "james"

console.log(info.friend.name)   // "james"
```



#### 数值的表示

```js
// b -> binary
const num2 = 0b100 // 二进制
// o -> octonary
const num3 = 0o100 // 八进制
// x -> hexadecimal
const num4 = 0x100 // 十六进制

console.log(num1, num2, num3, num4)    //100 4 64 256

// 大的数值的连接符(ES2021 ES12)
const num = 10_000_000_000_000_000
console.log(num)             //10000000000000000
```



#### Symbol













### iterator-generator

#### iterator迭代器

迭代器是帮助我们对某个数据结构进行遍历的对象，该对象满足迭代器协议（iterator protocol）

- 迭代器协议定义了产生一系列值（无论是有限还是无限个）的标准方式；在js中这个标准就是一个特定的**next方法**；

**next方法**有如下的要求：

一个无参数或者一个参数的函数，返回一个应当拥有以下两个属性的对象：

- done（boolean类型）
  - 如果迭代器可以产生序列中的下一个值，则为 false。
  - 如果迭代器已将序列迭代完毕，则为 true。这种情况下，value 是可选的，如果它依然存在，即为迭代结束之后的默认返回值。

- value
  - 迭代器返回的任何 JavaScript 值。done 为 true 时value可省略，默认值为undefined。

##### 可迭代对象

当一个对象实现了iterable protocol协议时，它就是一个可迭代对象。可迭代协议（iterable protocol）要求这个对象必须实现 @@iterator 方法，在代码中我们使用 `Symbol.iterator` 访问该属性。该方法会返回一个迭代器。

```js
// 创建一个迭代器对象来访问数组
const iterableObj = {
  names: ["abc", "cba", "nba"],
  [Symbol.iterator]: function() {
    let index = 0
    return {
      next: () => {  //使用箭头函数是为了使this的指向正确
        if (index < this.names.length) {
          return { done: false, value: this.names[index++] }
        } else {
          return { done: true, value: undefined }
        }
      }
    }      //返回一个迭代器
  }
}
```

迭代器的中断

可以在 `Symbol.iterator` 方法返回的对象中添加一个return方法用来监听迭代器是否在没有完全迭代的情况下提前中断。

```js
  [Symbol.iterator]() {
    let index = 0
    return {
      next: () => {
        if (index < this.students.length) {
          return { done: false, value: this.students[index++] }
        } else {
          return { done: true, value: undefined }
        }
      },
      return: () => {     //该方法用于监听迭代器是否在没有完全迭代的情况下提前中断，如break终止循环
        console.log("迭代器提前终止了~")
        return { done: true, value: undefined }
      }
    }
  }

  for (const stu of classroom) {
  		console.log(stu)
  		if (stu === "why") break
  }
```



#### generator生成器

生成器是ES6中新增的一种函数控制、使用的方案，它可以让我们更加灵活的控制函数什么时候继续执行、暂停执行等。

##### 生成器函数

生成器函数也是一个函数，但是和普通的函数有一些区别：

- 首先，生成器函数需要在function的后面加一个`*`符号
- 其次，生成器函数可以通过**yield**关键字来控制函数的执行流程： 
- 最后，生成器函数的返回值是一个Generator（生成器）：生成器是一种特殊的迭代器。

通过返回的generator的next方法控制生成器函数的执行,next方法的返回值与迭代器一样，格式类似{done:  false，value:  undefined}。可在yield关键字后跟value值，该value将会作为next方法返回值中的value值。其中，next方法可以带参数，该参数将会作为上一个yield语句的返回值。

```js
function* foo(num) {
  console.log("函数开始执行~")

  const value1 = 100 * num          //500
  console.log("第一段代码:", value1)
  const n = yield value1    //这里的n的值就是第二次调用next方法时传进来的实参值10

  const value2 = 200 * n
  console.log("第二段代码:", value2)
  const count = yield value2
}
// 生成器上的next方法可以传递参数
const generator = foo(5)
console.log(generator.next())    //执行到第一个yield便停止执行

console.log(generator.next(10))     // 第二段代码, 第二次调用next的时候执行的
```

##### 生成器的return、throw方法

`generator.return(15)`相当于在上一个yield语句后执行了 `return 15` 语句。

`generator.thorw('error')`相当于在当前执行的yield语句中抛出异常

在很多场景下，可用生成器替代迭代器使用，`yeid*`后跟可迭代对象就会依次迭代这个可迭代对象，每次迭代其中的一个值。



### async-await

#### async

async关键字用于声明异步函数

##### 关于异步函数的返回值

异步函数的返回值一定是一个Promise

- 情况一：异步函数也可以有返回值，但是异步函数的返回值会被包裹到Promise.resolve中；

- 情况二：如果我们的异步函数的返回值是Promise，最终异步函数返回的Promise的状态会由return的Promise状态决定；

- 情况三：如果我们的异步函数的返回值是一个对象并且实现了thenable，那么会由对象的then方法来决定；

**如果我们在async中抛出了异常，那么程序它并不会像普通函数一样报错，而是会作为Promise的reject来传递；**

### await

await关键字只能用在async函数中。

- 通常使用await是后面会跟上一个表达式，这个表达式会返回一个Promise；

- 那么await会等到Promise的状态变成fulfilled状态，之后继续执行异步函数；如果await后面的Promise状态为pending或rejected，则await语句后的代码将不会执行。

  - 如果await后面是一个普通的值，那么会直接返回这个值；

  - 如果await后面是一个thenable的对象，那么会根据对象的then方法调用来决定后续的值；

  - 如果await后面的表达式，返回的Promise是reject的状态，那么会将这个reject结果直接作为**函数返回值的Promise**的

    reject值；

- await后面的代码将作为返回的pormise的then方法中的resolve函数代码异步执行，会被放入微任务队列中。



### 事件循环

#### 进程和线程

**进程**是对运行时程序的封装，是系统进行**资源调度和分配的的基本单位**，实现了操作系统的并发； **线程**是**进程**的子任务，是**CPU调度和分派的基本单位**，用于保证程序的实时性，实现**进程**内部的并发；一个进程至少包含一个主线程，一个线程只能属于一个进程。所以我们也可以说进程是线程的容器；

##### 浏览器中的js线程

- 目前多数的浏览器其实都是多进程的，当我们打开一个tab页面时就会开启一个新的进程，这是为了防止一个页 面卡死而造成所有页面无法响应，整个浏览器需要强制退出；
- 每个进程中又有很多的线程，其中包括执行JavaScript代码的线程；
- **JavaScript是单线程的**，常见的比较耗时的操作如网络请求、定时器则是由浏览器来执行的，js只需在特定的时间执行应有的回调即可。

#### 浏览器的事件循环

![image-20220914144317344](C:\Users\28771\AppData\Roaming\Typora\typora-user-images\image-20220914144317344.png)

同步代码直接执行，遇到异步代码时这些异步操作会被浏览器的线程接管，如定时器、网络请求、DOM监听等操作，到了特定的时间之后，便会将这些代码放入一个事件队列中，当同步代码全部执行完后，便会从事件队列（微任务队列的优先级高于宏任务队列）中取出事件（回调函数）进行执行。

##### 宏任务和微任务

事件循环中并非只维护着一个队列，实际上分为微任务队列（microtask queue）和宏任务队列（macrotask queue）。

- 宏任务队列：ajax、setTimeout、setInterval、DOM监听、UI Rendering等 
- 微任务队列：Promise的then回调、 Mutation Observer API、queueMicrotask()等

- 在执行任何一个宏任务之前（不是队列，是一个宏任务），都会先查看微任务队列中是否有任务需要执行

**then方法的代码会在promise的状态变为fulfilled时被添加到微任务队列中。**



#### Node的事件循环

node的EvenetLoop是通过libuv实现的。libuv是一个多平台的专注于异步IO的库，libuv中主要维护了一个EventLoop和worker threads（线程池）；EventLoop负责调用系统的一些其他操作：文件的IO、Network、child-processes等。

示意图如下：

![image-20220914155109730](C:\Users\28771\AppData\Roaming\Typora\typora-user-images\image-20220914155109730.png)

**事件循环像是一个桥梁**，是连接着应用程序的JavaScript和系统调用之间的通道：

- 无论是我们的文件IO、数据库、网络IO、定时器、子进程，在完成对应的操作后，都会将对应的结果和回调函数放到事件循环（任务队列）中；
- 事件循环会不断的从任务队列中取出对应的事件（回调函数）来执行；

**在Node中一次完整的事件循环Tick分成很多个阶段：**（以下均属于**宏任务**，且优先级依次降低）

- **定时器（Timers）**：本阶段执行已经被 setTimeout() 和 setInterval() 的调度回调函数。
- **待定回调（Pending Callback）**：对某些系统操作（如TCP错误类型）执行回调，比如TCP连接时接收到 ECONNREFUSED。
- **idle, prepare**：仅系统内部使用。
- **轮询（Poll）**：检索新的 I/O 事件；执行与 I/O 相关的回调；
- **检测（check）**：setImmediate() 回调函数在这里执行                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               。
- **关闭（close）的回调函数**：一些关闭的回调函数，如：socket.on('close', ...)。

如图：（同样遵循每个宏任务执行前都必须清空微任务队列）

![image-20220914155709578](C:\Users\28771\AppData\Roaming\Typora\typora-user-images\image-20220914155709578.png)

**常考点：**

- **setTimeout和setInterval是属于Timers queue，优先级要高于setImmediate所在的check queue。**

- 与浏览器事件循环不同的是，Node中的微任务队列分为`nextTick microqueue`和`other microqueue`。前者的优先级高于后者。

**Node事件循环面试题**

```js
async function async1() {
  console.log('async1 start')
  await async2()
  console.log('async1 end')
}

async function async2() {
  console.log('async2')
}

console.log('script start')

setTimeout(function () {
  console.log('setTimeout0')
}, 0)

setTimeout(function () {
  console.log('setTimeout2')
}, 300)

setImmediate(() => console.log('setImmediate'));

process.nextTick(() => console.log('nextTick1'));

async1();

process.nextTick(() => console.log('nextTick2'));

new Promise(function (resolve) {
  console.log('promise1')
  resolve();
  console.log('promise2')
}).then(function () {
  console.log('promise3')
})

console.log('script end')

//输出
// script start
// async1 start
// async2
// promise1
// promise2
// script end
// nexttick1
// nexttick2
// async1 end
// promise3
// settimetout0
// setImmediate
// setTimeout2
```

**补充一道promsie题**

```js
Promise.resolve().then(() => {
  console.log(0);
  return 4              // 情况1.直接return一个值 相当于resolve(4)，非thenable的普通对象状态确定后直接将其                                         PormiseFulfillReaction（即log(res)）加入微任务队列
    
  // 情况2.return thenable的值:可以调用then方法，会产生一个thenableJob的微任务，非标准的then方法执行后确定pormise状态，状态      确定后enqueue其PormiseFulfillReaction
  return {
    then: function(resolve,reject) {
      // 大量的计算
      // resolve(4)
      reject(4)
    }
  }

  // 情况3.return Promise值，可以调用then方法加一次微任务（thenableJob），标准then会enqueue其父promise的reslove方法（再加一次微任务）
  return Promise.resolve(4)
}).then((res) => {
  console.log(res)
},(err) => {
    console.log(err,"err");
})

Promise.resolve().then(() => {
  console.log(1);
}).then(() => {
  console.log(2);
}).then(() => {
  console.log(3);
}).then(() => {
  console.log(5);
}).then(() =>{
  console.log(6);
})

// 1.return 4
// 0
// 1
// 4
// 2
// 3
// 5
// 6

// 2.return thenable
// 0
// 1
// 2
// 4
// 3
// 5
// 6

// 3.return promise
// 0
// 1
// 2
// 3
// 4
// 5
// 6
```





### 错误处理方案

#### throw关键字

- throw语句用于**抛出一个用户自定义的异常**；
- 当遇到throw语句时，当前的函数执行会被停止（throw后面的语句不会执行）；

- throw关键字后可以跟基本数据类型，也可以跟对象类型，对象能保存更多信息，因此更为常用。既可以使用自定义的错误类，也可以使用 JS 为我们提供的Error类。Error对象包含三个属性，即message错误信息、name错误类别、stack调用栈。其中Error包含一些子类，如**RangeError**：下标值越界时使用的错误类型、**SyntaxError**：解析语法错误时使用的错误类型、**TypeError**：出现类型错误时，使用的错误类型；

#### 如何处理异常--------try  catch

如果我们在调用一个函数时，这个函数抛出了异常，但是我们并没有对这个异常进行处理，那么这个异常会继续传递到上一个函数调用中；如果到了最顶层（全局）的代码中依然没有对这个异常的处理代码，这个时候就会报错并且终止程序的运行；

通常我们会使用try catch对异常进行捕获。

```js
try{
    //可能出现异常的代码片段
}catch(err){    //在ES10(2019)后err如不需要可省略
    //出现异常后所需执行的回调函数
}finally{
    //一些必须执行的代码，多为一些收尾工作代码
    return ;//如果finally中存在返回值，那么这个值将会成为整个try-catch-finally的返回值，无论是否有return语句在try和catch中。这包括在catch块里抛出的异常，即catch中抛出异常也会被finally拦截不会继续向外抛出
}
```





### 模块化

#### commonJS（多用于Node服务器端）

`CommonJS`规范规定，每个模块内部，`module`变量代表当前模块。这个变量是⼀个对象，它的`exports`属性（即
`module.exports`）是对外的接⼝。加载某个模块，其实是加载该模块的`module.exports`属性。`require`⽅法⽤于加载模块。

```js
//导出方式
module.exports = value
exports.xxx = value      //注意不能使用exports = value的方式，这相当于改变了exports指向
//导入方式----用一个变量去接收导入的模块
var module1 = require('模块路径')
```

Node为每⼀个模块提供了⼀个exports变量(可以说是⼀个对象)，指向 module.exports。这相当于每个模块中都有⼀句这样的命令 `var
exports = module.exports;` 而每个模块所导出的其实是module.exports指向的对象。因此使用exports = value的方式将不能正常导出。

##### require

在conmonJS中用于导入模块，使用方法，require（x），常见的查找规则

- x 是一个核心模块，如path、http，直接返回核心模块
- x是以./、../或/（根目录）等开头的路径，表示按相对路径查找指定文件，若该文件没有后缀名，则按照普通文件、js文件、json文件、node文件的顺序依次查找。若没有找到，则将x作为目录继续查找该目录下的index.js、index.json、index.node顺序依次查找，若还是没有找到目标文件，则报错并返回`not found`。
- 直接是一个x，则当作是一个下载模块进行查找，从当前路径依次到根目录下的node_modules中进行查找，没找到则返回`not found`。

##### commonJS中模块的加载

- commonJS中存在缓存机制，即使多次导入模块，该模块中的js代码也只会执行一次。
- 出现循环引用时，按照深度优先遍历的算法进行模块引入（与函数执行顺序一致）

#### ES Module

主文件进行模块引入

```html
<script src="main.js" type="module"></script>
```

了解：使用ES Module将会自动开启严格模式。使用ES模块化开发时，不能通过本地加载html文件，会出现跨域问题。因为HTML使用type="module"会默认产生跨域请求，我们是在本地打开的文件，而file协议并不支持。Live Server插件会在本地开启一台服务器，因此

##### 暴露方式----export

- 默认暴露

```js
//导出方式
export default {}      //在一个模块中只能有一个默认导出
//导入方式----可以使用变量的方式接收，即不用大括号括起来
import module from '模块名/模块路径'
```

- 多行暴露

```js
//导出方式
export function foo(){
    
}
export function bar(){
    
}
//导入方式----必须使用对象的解构赋值
import { foo,bar } from '模块名/模块路径'    //这里的{}也不是一个对象，里面只是存放导入的标识符列表内容；
```

- 统一暴露

```js
//导出方式
function fun1(){
    
}
function fun2(){
    
}
export { fun1,fun2}     //这里的{}不表示对象

//导入方式----必须使用对象的解构赋值
import { fun1,fun2} from '模块名/模块路径'
```

扩展：`export`和`import`关键字在导入和导出时都可以起别名

```js
import * as moudleA from '模块名/模块路径'
```

##### import和import函数

通过import加载一个模块，是不可以在其放到逻辑代码中的，因为ES Module是静态加载模块，在解析时就需要知道其依赖关系。若需要动态引入，则需使用import函数来实现动态加载。

```js
// import函数返回的结果是一个Promise
import("./foo.js").then(res => {
  console.log("res:", res.name)
})

console.log("后续代码~")
```



##### ES Module解析流程

- 阶段一：**构建**（Construction），根据地址查找js文件，并且下载，将其解析成模块记录（Module Record）；
- 阶段二：**解析**，下载完模块文件后，需要解析模块文件为模块记录以便浏览器识别模块文件的内容。模块记录文件创建完成后，会被保存在模块映射中，在模块加载器处理任何其他模块时可以通过模块映射中的缓存来取得已经解析好的模块记录。
-  阶段二：**实例化**（Instantiation），对模块记录进行实例化，并且分配内存空间，解析模块的导入和导出语句，把模块指向对应的内存地址。 （此时的变量的值还是undefined）。
- 阶段三：**运行**（Evaluation），运行代码，计算值，并且将值填充到内存地址中；

详细流程请查看：①[ 英文源地址 ：ES Module是如何被浏览器解析并且让模块之间可以相互引用的呢？](https://hacks.mozilla.org/2018/03/es-modules-a-cartoon-deep-dive/)  ② [中文搬运版]([详解ES模块系统 - 掘金 (juejin.cn)](https://juejin.cn/post/6844904007278788622#comment))



##### 关于commonJS和ES6 module的区别

- commonjs是运行时加载模块，而`ES6 Module`时编译时加载，即在编译阶段就已经确立了模块的依赖关系。

- commonjs导出的是一个值拷贝，会对加载结果进行缓存，一旦内部再修改这个值，则不会同步到外部。`ES6 Module`是导出的一个引用，内部修改可以同步到外部。
- commonjs不会提升require，ES6 Module在编译期间会将所有import提升到顶部。
- commonjs中顶层的this指向这个模块本身，而ES6中顶层this指向undefined。
- CommonJS是同步导入，因为用于服务端，文件都在本地，同步导入即使卡主主线程影响也不大。而ES6 Module是异步导入，因为用于浏览器，需要下载文件，如果采用同步导入对渲染有很大影响。

##### 如何理解运行时加载和编译时加载？

```js
//CommonJS运行时加载
let { stat, exists, readFile } = require('fs');

// 等同于
let _fs = require('fs');
let stat = _fs.stat;
let exists = _fs.exists;
let readfile = _fs.readfile;

//上面代码的实质是整体加载fs模块（即加载fs的所有方法），生成一个对象（_fs），然后再从这个对象上面读取 3 个方法。这种加载称为“运行时加载”，因为只有运行时才能得到这个对象，导致完全没办法在编译时做“静态优化”。


//ES6 Module编译时加载

import { stat, exists, readFile } from 'fs';

//上面代码的实质是从fs模块加载 3 个方法，其他方法不加载。这种加载称为“编译时加载”或者静态加载，即 ES6 可以在编译时就完成模块加载，效率要比 CommonJS 模块的加载方式高。
```





### 包管理工具

#### npm

##### 简单介绍

npm，即Node-Package-Manager，Node包管理器。npm是Node的一个包管理工具，安装node后电脑就自带了npm。npm进行包的发布会将包发布到registry仓库中，同样，下载也是从registry仓库中下载包。

##### 关于配置文件

我们每一个项目都会有一个对应的配置文件，这个配置文件会记录着你项目的名称、版本号、项目描述等；也会记录着你项目所依赖的其他库的信息和依赖库的版本号；

如何创建得到配置文件package.json？

- `npm init -y(表示使用默认信息创建)`
- 使用脚手架创建的项目会自动生成package.json文件

常见属性：

- private属性，记录当前项目是否私有，若为true则该项目将无法publish
- main属性，设置程序的入口。
- script属性，可以编写一些简单的脚本命令，以键值对形式进行配置，配置后使用时按 `npm run xxx` 的格式使用。
- dependencies属性，记录项目生产环境和开发环境都依赖的包。
- devdependencies属性，记录项目开发环境下需要的包。
- peerDependencies属性，有一种项目依赖关系是对等依赖，也就是你依赖的一个包，它必须是以另外一个宿主包为前提的；

##### 版本规范X.Y.Z

- X主版本号（major）：当你做了不兼容的 API 修改（可能不兼容之前的版本）；
- Y次版本号（minor）：当你做了向下兼容的功能性新增（新功能增加，但是兼容之前的版本）；
- Z修订号（patch）：当你做了向下兼容的问题修正（没有新功能，修复了之前版本的bug）；

补充：①^x.y.z：表示x是保持不变的，y和z永远安装最新的版本；

​			②~x.y.z：表示x和y保持不变的，z永远安装最新的版本；

常用命令：

```js
npm install  //快速安装package.json中所有的依赖包
npm install packageName //安装一个包
npm install packageName  -g  //全局安装一个包
npm install packageName --save-dev  //安装开发依赖
npm rebuild  //强制重新build，相当于删除node_modules文件夹然后重新执行npm install
npm cache clean  //清除缓存
```

##### npm原理图

![image-20220920112542695](C:\Users\28771\AppData\Roaming\Typora\typora-user-images\image-20220920112542695.png)

**另一种版本的原理图**

![img](https://img-blog.csdnimg.cn/bb070933512d409480e463a95647ec60.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5qyi5qyi5a2m57yW56iL,size_20,color_FFFFFF,t_70,g_se,x_16)

**重难点**：当执行npm install时，会先检查本地是否有lock文件:

1、如果没有会从package.json中获取包的一些信息然后检查缓存，没有缓存则下载包资源，然后添加到缓存。有缓存则直接解压，最终生成package-lock.json。

2、有package-lock.json时会判断与package.json版本是否一致，如果不一致，不同的npm版本会有不同的处理方式，所以当前npm版本需要特殊注意，一般我们现在安装的都是大于v5.4.2版本的，他的特性是在安装时，如果package-lock.json安装版本兼容package.json则按照package-lock.json安装，如果不兼容，按照package.json安装并更新package-lock.json。

补充：**lock文件的作用**：package-lock.json的作⽤就是锁定安装时的版本号并上传到git上，保证其他⼈在npm install时下载的依赖都保持一致。

##### npm发布自己的包

```js
npm login   //1.登录npm
            //2.修改package.json
npm publish //3.发布/更新包

npm unpublish   //删除发布的包
npm deprecate   //让发布的包过期
```



#### yarn工具

早期npm尚未成熟时为了弥补npm的一些缺陷而由facebook、google等公司联合推出的一款包管理工具。

常用指令：

![image-20220920133027250](C:\Users\28771\AppData\Roaming\Typora\typora-user-images\image-20220920133027250.png)

注意：npm和yarn不能混用。

#### cnpm工具

使用上和npm几乎一致，常用来搭配淘宝镜像源使用。

```js
npm config get registry   //查看仓库源

npm config set registry https://registry.npm.taobao.org   //设置镜像源

npm install -g cnpm --registry=https://registry.npm.taobao.org   //安装cnpm

cnpm config get registry # https://r.npm.taobao.org/
```

#### npx工具

npx是npm5.2后自带的一个工具。npx的作用非常多，但是比较常见的是使用它来调用项目中的某个模块的指令。

npx会到当前目录的node_modules/.bin目录下查找对应的命令，从而避免使用的是全局环境下的某一命令。例：全局安装的是webpack5.1.3，项目安装的是webpack3.6.0，在当前项目直接使用`webpack --version` 指令得到的是全局环境中的webpack5.1.3，若使用 `npx webpack --version`指令则可以使用本项目的webpack。另一种方法 `"scripts": {"webpack": "webpack --version"}` 也能实现该效果。



### JSON

全称是JavaScript Object Notation，是一种可以在服务器和客户端之间传输的数据格式，独立于编程语言，可以在各个编程语言中使用。

常见的使用场景：

- 用于网络传输JSON数据
- 配置文件
- 非关系型数据库将json作为存储格式

#### 基本语法

JSON的顶层支持三种类型的值：（**注意不支持undefined**和**function**）

1. 简单值：数字（Number）、字符串（String，不支持单引号）、布尔类型（Boolean）、null类型；
2. 对象值：由key、value组成，key是字符串类型，并且必须添加双引号，值可以是简单值、对象值、数组值；
3. 数组值：数组的值可以是简单值、对象值、数组值；

两个基本方法：

1. `JSON.stringify(obj)`，用于将一个obj对象或值转化为 `json` 字符串

2. `JSON.paese(str)` ，用于将一个json字符串转化为对应的js类型

补充了解，`stringify` 方法还可以带有另外两个参数，分别为replace和space。

**replace：**为数组时指明那些属性需要转换，为函数时可以对转换后的value值做修改

![image-20220926200222055](C:\Users\28771\AppData\Roaming\Typora\typora-user-images\image-20220926200222055.png)

**space**：该参数用于改变输出格式，可以是“--------”等。

将改变输出格式为自动缩进，默认缩进为2个空格。

![image-20220926200340736](C:\Users\28771\AppData\Roaming\Typora\typora-user-images\image-20220926200340736.png)



可以利用JSON序列化的特点进行简单的深拷贝。即对目标对象先使用`JSON.stringify()` 方法，在使用`JSON.parse()` 方法转回普通对象。注意：该方法不适用于含有function的对象。



### Storage

**Storage分为localStorage和sessionStorage，有如下的属性和方法：**

**属性**：

​	Storage.length：只读属性。返回一个整数，表示存储在Storage对象中的数据项数量； 

**方法**：

- `Storage.key()`：该方法接受一个数值n作为参数，返回存储中的第n个key名称； 
- `Storage.getItem()`：该方法接受一个key作为参数，并且返回key对应的value； key不存在返回null
- `Storage.setItem()`：该方法接受一个key和value，并且将会把key和value添加到存储中。如果key已经存在，则更新其对应的值；
- `Storage.removeItem()`：该方法接受一个key作为参数，并把该key从存储中删除； 
- `Storage.clear()`：该方法的作用是清空存储中的所有key；



### Cookie

某些网站为了辨别用户身份而存储在用户本地终端（Client Side）上的数据。

浏览器会在特定的情况下携带上cookie来发送请求。按照保存在客户端的位置，Cookie又可以分为内存Cookie和硬盘Cookie。

- 内存Cookie由浏览器维护，保存在内存中，浏览器关闭时Cookie就会消失，其存在时间是短暂的；即没有设置过期时间，默认情况下cookie是内存cookie，在关闭浏览器时会自动删除；
- 硬盘Cookie保存在硬盘中，有一个过期时间，用户手动清理或者过期时间到时，才会被清理；即有设置过期时间，并且过期时间不为0或者负数的cookie，是硬盘cookie，需要手动或者到期时，才会删除；

**cookie的作用域**

Domain：指定哪些主机可以接受cookie

Path：指定主机下哪些路径可以接受cookie

**客户端设置cookie**

```js
document.cookie = ”name=why;max-age=10“     //添加cookie并设置过期时间
```



### IndexDB

IndexedDB是一种底层的API，用于在客户端存储大量的结构化数据。

它是一种事务型数据库系统，是一种基于JavaScript面向对象数据库，有点类似于NoSQL（非关系型数据库）。IndexDB本身就是基于事务的，我们只需要指定数据库模式，打开与数据库的连接，然后检索和更新一系列事务即可；

**使用步骤：**

```js
// 1. 打开数据(和数据库建立连接)
const dbRequest = indexedDB.open("why", 3)
dbRequest.onerror = function(err) {
  console.log("打开数据库失败~")
}
let db = null
dbRequest.onsuccess = function(event) {
  db = event.target.result
}
// 第一次打开/或者版本发生升级
dbRequest.onupgradeneeded = function(event) {
  const db = event.target.result

  console.log(db)

  // 创建一些存储对象
  db.createObjectStore("users", { keyPath: "id" })
}

class User {
  constructor(id, name, age) {
    this.id = id
    this.name = name
    this.age = age
  }
}

const users = [
  new User(100, "why", 18),
  new User(101, "kobe", 40),
  new User(102, "james", 30),
]

// 获取btns, 监听点击
const btns = document.querySelectorAll("button")
for (let i = 0; i < btns.length; i++) {
  btns[i].onclick = function() {
    const transaction = db.transaction("users", "readwrite")  //创建一个事务对象，第一个参数可以是数组，表示该事务拥有多个存储对象
    console.log(transaction)
    const store = transaction.objectStore("users")       //获取其中具体想操作的存储对象

    switch(i) {
      case 0:
        console.log("点击了新增")
        for (const user of users) {
          const request = store.add(user)
          request.onsuccess = function() {
            console.log(`${user.name}插入成功`)
          }
        }
        transaction.oncomplete = function() {
          console.log("添加操作全部完成")
        }
        break
      case 1:
        console.log("点击了查询")

        // 1.查询方式一(知道主键, 根据主键查询)
        // const request = store.get(102)
        // request.onsuccess = function(event) {
        //   console.log(event.target.result)
        // }

        // 2.查询方式二:
        const request = store.openCursor()
        request.onsuccess = function(event) {
          const cursor = event.target.result
          if (cursor) {
            if (cursor.key === 101) {
              console.log(cursor.key, cursor.value)
            } else {
              cursor.continue()
            }
          } else {
            console.log("查询完成")
          }
        }
        break
      case 2:
        console.log("点击了删除")
        const deleteRequest = store.openCursor()
        deleteRequest.onsuccess = function(event) {
          const cursor = event.target.result
          if (cursor) {
            if (cursor.key === 101) {
              cursor.delete()
            } else {
              cursor.continue()
            }
          } else {
            console.log("查询完成")
          }
        }
        break
      case 3:
        console.log("点击了修改")
        const updateRequest = store.openCursor()
        updateRequest.onsuccess = function(event) {
          const cursor = event.target.result
          if (cursor) {
            if (cursor.key === 101) {
              const value = cursor.value;
              value.name = "curry"
              cursor.update(value)
            } else {
              cursor.continue()
            }
          } else {
            console.log("查询完成")
          }
        }
        break
    }
  }
}
```







### BOM

BOM，浏览器对象模型，可以将BOM看成是连接JavaScript脚本与浏览器窗口的桥梁。

**常见的BOM对象**

- window：包括全局属性、方法，控制浏览器窗口相关的属性、方法；
-  location：浏览器连接到的对象的位置（URL）；

- history：操作浏览器的历史；

- document：当前窗口操作文档的对象；

#### window全局对象

浏览器中的全局对象，在Node环境中全局对象为global。在JS执行过程中充当GO对象。

全局环境下，用var声明的变量会作为属性被添加到window上

提供的常用类，如Date、setTimeout、Object、Math等

作为窗口对象

- 第一：包含大量的属性，localStorage、console、location、history、screenX、scrollX等等（大概60+个属性）；

- 第二：包含大量的方法，alert、close、scrollTo、open等等（大概40+个方法）；

- 第三：包含大量的事件，focus、blur、load、hashchange等等（大概30+个事件）；

- 第四：包含从EventTarget继承过来的方法，addEventListener、removeEventListener、dispatchEvent方法；

```js
//常见的监听事件
window.onload = function() {
  console.log("window窗口加载完毕~")
}

window.onfocus = function() {
  console.log("window窗口获取焦点~")
}

window.onblur = function() {
  console.log("window窗口失去焦点~")
}

const hashChangeBtn = document.querySelector("#hashchange")
hashChangeBtn.onclick = function() {
  location.hash = "aaaa"
}
window.onhashchange = function() {
  console.log("hash发生了改变")
}
```



#### document对象

常用属性及方法

```js
document.documentElement   //返回当前页面的根元素HTML，只读
document.body    //返回页面的body元素
document.location    //返回当前网页的url
document.URL    //与document.location.href全等，但该属性是只读的
document.title   //用于读取或修改当前文档的标题

document.getElementById('box1')    //根据id获取指定元素
Document.createElement('div')   //创建新元素
document.getElementsByClassName('box')   //返回一个包含指定类名的元素集合
document.getElementsByTagName('p')  //获取所有的p元素，返回一个html集合
document.querySelector('.box');   //参数为一个选择器，返回满足条件的第一个
document.querySelectorAll('.box');    //返回一个集合
```



#### location对象

>Location对象用于表示window上当前链接到的URL信息。
>
>>href: 当前window对应的超链接URL, 整个URL；
>>
>>protocol: 当前的协议；
>>
>>host: 主机地址；
>>
>>hostname: 主机地址(不带端口)；
>>
>>port: 端口；
>>
>>pathname: 路径；
>>
>>search: 查询字符串；
>>
>>hash: 哈希值；
>>
>>username：URL中的username（很多浏览器已经禁用）；
>>
>>password：URL中的password（很多浏览器已经禁用）；

![image-20221012202827235](C:\Users\28771\AppData\Roaming\Typora\typora-user-images\image-20221012202827235.png)

>location有如下常用的方法：
>
>>assign：赋值一个新的URL，并且跳转到该URL中；
>>
>>replace：打开一个新的URL，并且跳转到该URL中（不同的是不会在浏览记录中留下之前的记录）；
>>
>>reload：重新加载页面，可以传入一个Boolean类型；为true则会向服务器重新请求



#### history对象

该对象记录了浏览器的历史访问记录

```js
//属性
length   //会话中的记录条数
state    //当前保留的状态值

//方法
back()  //返回上一页，相当于go(-1)
forward()  //前进一页，相当于go(1)
go(n)   //指定前进或后退n页
pushState()   //打开一个指定的地址；
replaceState()   //打开一个新的地址，并且使用replace；
```





### DOM

DOM，文档对象模型，DOM给我们提供了一系列对象用于帮助我们更方便的操作DOM。

**关系继承图**

![image-20221012205550068](C:\Users\28771\AppData\Roaming\Typora\typora-user-images\image-20221012205550068.png)



##### EventTarget

```js
function clickFn(){
  console.log("div元素被点击")
}
//addEventListener方法
divEl.addEventListener("click",clickFn)
//removeEventListener方法
divEl.removeEventListener('click',clickFn)
```



##### Node

> 常用属性
>
> nodeName：node节点的名称。
>
> nodeType：可以区分节点的类型。
>
> nodeValue：node节点的值；
>
> childNodes：所有的子节点；
>
> >常用方法
> >
> >appendChild()    //父节点调用,新增子节点
> >
> >insertBefore()     //parentNode.insertBefore(newNode, referenceNode),父节点调用,在指定节点前插入新节点
> >
> >removeChild()    //父节点调用该方法移除子节点



##### Document对象

参见[Document](#document对象)



##### Element对象

```js
// 常见的属性
id
tagName
children
className
classList
clientWidth
clientHeight
offsetLeft
offsetTop

// 常见的方法
const value = divEl.getAttribute("age")

divEl.setAttribute("height", 1.88)
```



### 事件监听

方式一：通过元素的on来监听事件；

方式二：通过EventTarget中的addEventListener来监听；

**区别：通过on绑定的事件监听函数会覆盖,而addEventListener添加的事件监听函数会叠加。**



#### event对象

事件回调函数中包含一个参数event，也称事件对象。该对象给我们提供了想要的一些属性，以及可以通过该对象进行某些操作；

```js
// 常用属性
// type：事件的类型；
// target：当前事件发生的元素；
// currentTarget：当前处理事件的元素；
// offsetX、offsetY：点击元素的位置；

//常见方法
event.preventDefault()   //阻止默认事件触发，如a标签的跳转
event.stopPropagation()   //阻止事件进一步传递，包括冒泡和捕获
```



####  事件冒泡和事件捕获

![image-20221012213338970](C:\Users\28771\AppData\Roaming\Typora\typora-user-images\image-20221012213338970.png)



addEventListener()方法接受三个参数，第一个为事件类型type如click，第二个参数`listenr`，常为事件的回调函数，第三个参数为boolean类型，表示 `listener` 会在该类型的事件捕获阶段传播到该 `EventTarget` 时触发。默认为false，即在事件冒泡时触发事件的回调函数。
