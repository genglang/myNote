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
  - 解构不仅可以用于数组,还可以用于对象。
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
  - 对象的解构赋值,可以很方便地将现有对象的方法,赋值到某个变量。
  ```
  let { log, sin, cos } = Math;
  ```
  - 由于数组本质是特殊的对象,因此可以对数组进行对象属性的解构。
  ```
  let arr = [1, 2, 3];
  let {0 : first, [arr.length - 1] : last} = arr;
  first // 1
  last // 3
  ```
  
### 字符串的解构赋值
  - 字符串会被转成类数组对象进行解构赋值
  ```
  const [a, b, c, d, e] = 'hello';
  a // "h"
  b // "e"
  c // "l"
  d // "l"
  e // "o"
  ```
  - 类数组对象都有一个length属性,也可以进行解构赋值
  ```
  let {length : len} = 'hello';
  len // 5
  ```
  
### 数值和布尔值的解构赋值
  - 解构赋值时,如果等号右边是布尔值,会被先转为对象
  ```
  let {toString: s} = 123;
  s === Number.prototype.toString // true
  
  let {toString: s} = true;
  s === Boolean.prototype.toString // true
  ```
  - 解构赋值的规则是,只要右边的值不是对象或数组,就先转为对象,因此null和undefined无法进行解构赋值
  ```
  let { prop: x } = undefined; // TypeError
  let { prop: y } = null; // TypeError
  ```
  
 ### 函数参数解构
  - 当参数为对象时,设置默认值可能会有所不同
  ```
  function move({x = 0, y = 0} = {}) {
    return [x, y];
  }
  
  move({x: 3, y: 8}); // [3, 8]
  move({x: 3}); // [3, 0]
  move({}); // [0, 0]
  move(); // [0, 0]
  ```
  - 结果取决于给什么设置默认值
  ```
  function move({x, y} = { x: 0, y: 0 }) {
    return [x, y];
  }
  
  move({x: 3, y: 8}); // [3, 8]
  move({x: 3}); // [3, undefined]
  move({}); // [undefined, undefined]
  move(); // [0, 0]
  ```
  
### 圆括号问题
  - ES6规定,只要可能导致解构歧义,就不能使用圆括号
  1. 变量声明语句
  ```
  // 全部报错
  let [(a)] = [1];
  
  let {x: (c)} = {};
  let ({x: c}) = {};
  let {(x: c)} = {};
  let {(x): c} = {};
  
  let { o: ({ p: p }) } = { o: { p: 2 } };
  ```
  2. 函数参数
  ```
  // 报错
  function f([(z)]) { return z; }
  // 报错
  function f([z,(x)]) { return x; }
  ```
  3. 赋值语句模式
  ```
  // 全部报错
  ({ p: a }) = { p: 42 };
  ([a]) = [5];
  [({ p: a }), { x: c }] = [{}, {}];
  ```
  - 可以使用的情况
    - 只有赋值语句的非模式部分可以使用圆括号
    ```
    [(b)] = [3]; // 正确
    ({ p: (d) } = {}); // 正确
    [(parseInt.prop)] = [3]; // 正确
    ```
### 用途
  1. 交换变量
  ```
  let x = 1;
  let y = 2;
  
  [x, y] = [y, x];
  ```
  2. 方便地从函数返回多个值提取参数
  3. 函数参数的定义
  4. 方便提取JSON数据
  5. 函数参数默认值
  6. 遍历Map结构
      ```
      const map = new Map();
      map.set('first', 'hello');
      map.set('second', 'world');
      for (let [key, value] of map) {
        console.log(key + " is " + value);
      }
      // first is hello
      // second is world
      ```
      - 如果只想获取键名,或者只想获取键值,可以写成下面这样
      ```
      // 获取键名
      for (let [key] of map) {
        // ...
      }
      
      // 获取键值
      for (let [,value] of map) {
        // ...
      }
      ```
    
  7. 输入模块的指定方法
  ```
  const { SourceMapConsumer, SourceNode } = require("source-map");
  ```
  
## 字符串的拓展

### 字符的 Unicode 表示法
  - JavaScript允许采用\uxxxx形式表示一个字符,其中xxxx表示字符的Unicode码点
  ```
  "\u0061"
  // "a"
  ```
  - 这种表示法只限于码点在\u0000~\uFFFF之间的字符
  - 超出这个范围的字符,必须用两个双字节的形式表示
  ```
  "\uD842\uDFB7"
  // "𠮷"
  
  "\u20BB7"
  // " 7"
  ```
  - 上面代码表示,如果直接在\u后面跟上超过0xFFFF的数值(比如\u20BB7)
  - JavaScript会理解成\u20BB+7,由于\u20BB是一个不可打印字符,所以只会显示一个空格,后面跟着一个7
  - ES6对这一点做出了改进,只要将码点放入大括号,就能正确解读该字符
  ```
  "\u{20BB7}"
  // "𠮷"
  
  "\u{41}\u{42}\u{43}"
  // "ABC"
  
  let hello = 123;
  hell\u{6F} // 123
  
  '\u{1F680}' === '\uD83D\uDE80'
  // true

  ```
  
### codePointAt
  - JS内部,字符串以UTF-16的格式存储,每个字符固定为2字节,对于四个字节大小的字符会被当成两个字符
  - 因此charAT无法正确读取部分字符串,charCodeAt只能分别返回前两个字节和后两个字节的值
  ```
  var s = "𠮷";
  s.length // 2
  s.charAt(0) // ''
  s.charAt(1) // ''
  s.charCodeAt(0) // 55362
  s.charCodeAt(1) // 57271
  ```
  - ES6提供了codePointAt方法能够正确处理四个字节储存的字符串,返回一个字节码点
  - 参数是字符在字符串中的位置,占两位大小的字节会在第一个字节的时候获取全部的码点,第二个字节获取第二个字节的码点
  ```
  let s = '𠮷a';
  s.codePointAt(0) // 134071
  s.codePointAt(1) // 57271
  s.codePointAt(2) // 97
  ```
  - codePointAt方法返回的是码点的十进制值,如果想要十六进制的值,可以使用toString方法转换一下
  ```
  let s = '𠮷a';
  s.codePointAt(0).toString(16) // "20bb7"
  s.codePointAt(2).toString(16) // "61"
  ``` 
  - 如果需要对应位置的参数,使用for-of循环,它会正确识别32位UTF-16字符
  ```
  let s = '𠮷a';
  for (let ch of s) {
    console.log(ch.codePointAt(0).toString(16));
  }
  // 20bb7
  // 61
  ```
