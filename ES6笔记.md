# ES6笔记

## let和const

### let
  - let命令只在其所在的代码块内有效
  - let在循环中只在本轮循环中有效
  - for循环设置变量的部分是一个父作用域,循环体内部是一个单独的子作用域
  - let没有变量提升
  - 暂时性死区
    - 只要块级作用域内存在let命令,它所声明的变量就"绑定"(binding)这个区域,不再受外部的影响
    ```
    var tmp = 123;
    
    if (true) {
      tmp = 'abc'; // ReferenceError: tmp is not defined
      let tmp;
    }
    ```
    - ES6 明确规定,如果区块中存在let和const命令,这个区块对这些命令声明的变量,从一开始就形成了封闭作用域,凡是在声明之前就使用这些变量,就会报错
    - 在代码块内,使用let命令声明变量之前,该变量都是不可用的。这在语法上,称为"暂时性死区"
    - "暂时性死区"让typeof不再是100%安全的操作
    - 但是如果一个变量根本没有被声明,使用typeof反而不会报错
    - 函数默认参数死区
    ```
    function bar(x = y, y = 2) { // 调用y的时候y还未声明
      return [x, y];
    }
    
    bar(); // 报错
    ```
    - let赋值不能赋值给自身,因为赋值的时候自身还未声明完成
    ```
    // 不报错
    var x = x;
    
    // 报错
    let x = x;
    // ReferenceError: x is not defined
    ```
  - let不允许重复声明
  - let有块级作用域
    - let实际上为JS新增了块级作用域
    - ES6允许块级作用域的任意嵌套
    - 外层作用域无法访问内层作用域的变量
    - 内层作用域可以重新定义外层作用域的同名变量,但是不会修改外层作用域的内容
    ```
    {
    	let a = 1
    	{
    		let a = 2
    		console.log(a) // 2
    	}
    	console.log(a) // 1
    }
    ```
    - 块级作用域的出现让IIFE(自执行函数)不再必要了
  - 块级作用域和函数声明
    - ES5规定函数只能在顶层作用域和函数作用域内声明(但是浏览器兼容非法的写法)
    - ES6中块级作用域中的函数声明类似let(不可在外调用)
    ```
    function f() { console.log('I am outside!'); }
    
    (function () {
      if (false) {
        // 重复声明一次函数f
        function f() { console.log('I am inside!'); }
      }
    
      f();
    }());
    
    // ES5环境会输出inside
    // ES6环境会报错
    ```
    - ES6块级作用域声明函数必须要有大括号

### const
  - const声明一个只读的常量,一旦声明,常量的值就不能改变
  - const一旦声明变量,就必须立即初始化
  - const与let相同只在声明所在的块级作用域有效
  - const也不提升,也有暂时性死区,也不能重复声明
  - const实际上保证的,并不是变量的值不得改动,而是变量指向的那个内存地址所保存的数据不得改动,因此无法保证引用类型内部内容不改变

### ES6声明的6种方法
  1. var 
  2. function
  3. let
  4. const
  5. import
  6. class
  
### 顶层对象的属性
  - 顶层对象,在浏览器环境指的是window对象,在Node指的是global对象
  - ES5之中,顶层对象的属性与全局变量是等价的
  ```
  window.a = 1;
  a // 1
  
  a = 2;
  window.a // 2
  ```
  - ES6为了改变这个问题并保持兼容性
    - var和function声明全局变量依旧是顶层对象的属性
    - let、const、class声明的全局变量不属于顶层对象属性

### global对象 
  - ES5 的顶层对象,本身也是一个问题,因为它在各种实现里面是不统一的
    - 浏览器里面,顶层对象是window,但Node和WebWorker没有window
    - 浏览器和WebWorker里面,self也指向顶层对象,但是Node 没有self
    - Node里面,顶层对象是global,但其他环境都不支持
  - 同一段代码为了能够在各种环境,都能取到顶层对象,现在一般是使用this变量,但是有局限性
    - 全局环境中,this会返回顶层对象,但是,Node模块和ES6模块中,this返回的是当前模块
    - 函数里面的this,如果函数不是作为对象的方法运行,而是单纯作为函数运行,this会指向顶层对象,但是,严格模式下,这时this会返回undefined。
    - 不管是严格模式,还是普通模式,new Function('return this')(),总是会返回全局对象,但是,如果浏览器用了 CSP（Content Security Policy,内容安全策略）,那么eval、new Function这些方法都可能无法使用
  - 勉强能用的解决方案
  ```
  // 方法一
  (typeof window !== 'undefined'
     ? window
     : (typeof process === 'object' &&
        typeof require === 'function' &&
        typeof global === 'object')
       ? global
       : this);
  
  // 方法二
  var getGlobal = function () {
    if (typeof self !== 'undefined') { return self; }
    if (typeof window !== 'undefined') { return window; }
    if (typeof global !== 'undefined') { return global; }
    throw new Error('unable to locate global object');
  };
  ```
  - 现在有个提案,所有环境都有global

## 变量的解构赋值
  - ES6 允许按照一定模式,从数组和对象中提取值,对变量进行赋值,这被称为解构
### 数组的解构赋值
  - 现在允许这么写
  ```
  let [a, b, c] = [1, 2, 3]
  ```
  - 如果解构不成功,变量的值就等于undefined
  - 左右数组不匹配时也能成功解构
  - 如果等号的右边不是数组(或者严格地说,不是可遍历的结构,参见《Iterator》一章),那么将会报错
  ```
  let [foo] = 1;
  let [foo] = false;
  let [foo] = NaN;
  let [foo] = undefined;
  let [foo] = null;
  let [foo] = {};
  ```
  - 上面的语句都会报错,因为等号右边的值,要么转为对象以后不具备Iterator 接口(前五个表达式),要么本身就不具备 Iterator 接口(最后一个表达式)
  - 某种数据结构具有Iterator接口,都可以采用数组形式的解构赋值
  ```
  function* fibs() {
    let a = 0;
    let b = 1;
    while (true) {
      yield a;
      [a, b] = [b, a + b];
    }
  }
  
  let [first, second, third, fourth, fifth, sixth] = fibs();
  sixth // 5
  ```
  - 解构赋值允许指定默认值
  ```
  let [foo = true] = [];
  foo // true
  let [x, y = 'b'] = ['a']; // x='a', y='b'
  let [x, y = 'b'] = ['a', undefined]; // x='a', y='b'
  
  let [x = 1] = [undefined];
  x // 1
  
  let [x = 1] = [null];
  x // null null不严格等于undefined
  ```
  - 如果默认值是表达式,那么这个表达式是惰性求值,只有调用的时候才会求值
  - 默认值可以引用解构赋值的其他变量,但该变量必须已经声明

### 对象的解构赋值
  - 解构不仅可以用于数组，还可以用于对象。
  ```
  let { foo, bar } = { foo: "aaa", bar: "bbb" };
  foo // "aaa"
  bar // "bbb"
  ```
  - 对象的解构是没有次序的,变量必须喝属性同名,才能取到正确值
  - 如果对象名和属性名不一致时,必须写成这样
  ```
  let { foo: baz } = { foo: 'aaa', bar: 'bbb' };
  baz // "aaa"
  ```
  - 对象解构能用于嵌套对象,也能设置默认值,同时触发默认值必须严格等于undefined
  - 对已声明变量进行解构赋值的时候,请用括号扩起来,不然{}会被解释成代码块
  - 对象的解构赋值，可以很方便地将现有对象的方法，赋值到某个变量。
  ```
  let { log, sin, cos } = Math;
  ```
  - 由于数组本质是特殊的对象，因此可以对数组进行对象属性的解构。
  ```
  let arr = [1, 2, 3];
  let {0 : first, [arr.length - 1] : last} = arr;
  first // 1
  last // 3
  ```