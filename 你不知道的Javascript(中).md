# 你不知道的JavaScript笔记(中)
## 一、类型和语法

### ES语言类型
  1. Undefined
  2. Null
  3. Boolean
  4. String
  5. Number
  6. Object
  7. Symbol
  
  - typeOf null === 'object' // true js的bug
  - JS的变量是没有类型的,值才有,变量可以随时持有任何类型的值
  - undefined!==undeclared 未赋值不等于未定义
    - undefined 未赋值
    - undeclared 未声明
  ```
  var a;
  a; // undefined
  b; // ReferenceError: b is not defined
  ```
  ```
  // 这样会抛出错误
  if (DEBUG) {
    console.log( "Debugging is starting" );
  }
  // 这样是安全的
  if (typeof DEBUG !== "undefined") {
    console.log( "Debugging is starting" );
  }
  ```
  - polyfill:补充代码
  ```
  // 或者这样子检查全局变量是否存在
  if (window.DEBUG) {
    // ..
  }
  if (!window.atob) {
    // ..
  }
  ```
  - 简单的依赖注入模式
  ```
  function doSomethingCool(FeatureXYZ){
    var helper = FeatureXYZ || function() { /* 默认执行的行为 */ }
    var val = helper
  }
  ```

## 二、数组
  - delete可以删除数组元素 但是不会改变数组长度
  - 创建稀松数组(有空白或者空缺单元的数组)的时候,单元空白(empty slot)可能会导致出人意料的结果
  ```
   a[0] = 1
   a['name'] = 'dev'
   a['1'] = 2
   a.length //2
   a[0] //1
   a['foobar'] //dev
   a[1] //2
  ```
  - 建议不要在数组中加入字符串键/ 建议用对象来存放
  
### 类数组
  - 类数组是一组数字索引的对象
    1. 一些DOM查询操作会返回DOM元素列表,它们并不是传统意义上的数组
    2. 另一个类数组例子是通过arguments对象将参数当作列表来访问(ES6废止)
   
  - 通过数组工具函数(indexOf、concat、forEach)实现将类数组对象转换成数组
  - 工具slice经常用于这类转换
  ```
  function foo() {
    var arr = Array.prototype.slice.call( arguments );
    arr.push( "bam" );
    console.log( arr );
  }
  foo( "bar", "baz" ); // ["bar","baz","bam"]
  ```
  - 在ES6中的内置工具函数Array.from()也能实现同样的功能

### 字符串
  - 字符串经常被当作字符串数组,字符串内部的实现究竟有没有使用数组并不好说,但是JS中的数组和字符串数组并不是一回事,只是看上去相似
  - JS中字符串是不可变的,数组是可变的,并且String[i]并非是合法语法,在老版本IE中报错,正确用法是a.charAt(1)
  - 字符串不可变是指字符串的成员函数不会改变其原始值,而是创建并返回一个新的字符串
  - 数组的成员函数都是在其原始值上进行操作
  
  - 字符串反转
  ```
  var c = a
          // 将a的值转换成字符串数组
          .split('')
          // 将数组中的字符进行倒转
          .reverse()
          // 转换回数组
          .join('')
          // 不适用于复杂字符串！
  ```
  - 如果需要经常以数组的形式来处理字符串的话,使用字符串数组,需要的时候再通过join('')转换为数组

### 数字
  - 数字包括"整数"和浮点数,整数带引号是因为和其他语言不通,JS没有真正意义上的整数,JS的整数就是没有小数的十进制数
  - JS中的数字类型是基于IEEE754标准来实现的,该标准也通常被称为浮点数,JS用的是双精度格式
  - 特别大和特别小的函数默认用指数格式显示,与toExponential()函数输出结果相同
  ```
  var a = 5E10
  a;                 // 50000000000
  a.toExponential(); // "5e+10"
  var b = a * a;
  b;                 // 2.5e+21
  var c = 1 / a;
  c;                 // 2e-11
  ```
  - 由于数字值可以使用Number对象进行封装,因此数字值可以调用Number.prototype的方法
  - 数字调用.运算符会优先识别为常量 容易报错
  ```
  // 无效语法：
  42.toFixed( 3 ); // SyntaxError 42.被引擎识别为浮点数
  // 下面的语法都有效：
  (42).toFixed( 3 ); // "42.000"
  0.42.toFixed( 3 ); // "0.420"
  42..toFixed( 3 ); // "42.000" 
  42 .toFixed( 3 ); // "42.000" 注意空格
  ```
  - 二进制浮点数(IEEE754)最大的问题是计算的时候会产生细微偏差
  ```
  0.1 + 0.2 === 0.3; // false 并非正好等于0.3
  ```
  - 设置一个误差范围值,通常称为“机器精度”(machine epsilon), 对JavaScript的数字来说,这个值通常是2^-52 (2.220446049250313e-16)来处理精度问题
  - 从ES6 开始,该值定义在Number.EPSILON 中,我们可以直接拿来用,也可以为ES6 之前的版本写polyfill：
  ```
  if (!Number.EPSILON) {
    Number.EPSILON = Math.pow(2,-52);
  }
  ```
  - 使用Number.EPSILON来比较两个数字是否相等(在指定的误差范围内)
  ```
  function numbersCloseEnoughToEqual(n1,n2) {
    return Math.abs( n1 - n2 ) < Number.EPSILON;
  }
  var a = 0.1 + 0.2;
  var b = 0.3;
  numbersCloseEnoughToEqual( a, b ); // true
  numbersCloseEnoughToEqual( 0.0000001, 0.0000002 ); // false
  ```
  - 所能呈现的最大浮点数大约是1798e+308 被定义在Number.MAX_VALUE
  - 所能呈现的最小浮点数大约是5e-324 被定义在Number.MIN_VALUE
  - 所能安全呈现的最大整数是2^53 - 1，即9007199254740991 被定义在Number.MAX_SAFE_INTEGER
  - 所能安全呈现的最大整数是-9007199254740991 被定义在Number.MIN_SAFE_INTEGER
  
  - 检测一个值是否为整数
  ```
  Number.isInteger() // ES6
  
  if (!Number.isInteger) { // polyfill
    Number.isInteger = function(num) {
      return typeof num == "number" && num % 1 == 0;
    };
  }
  ```
  - 检测一个值是否为安全整数
  ```
  Number.isSafeInteger() // ES6
  if (!Number.isSafeInteger) {
    Number.isSafeInteger = function(num) {
      return Number.isInteger( num ) &&
      Math.abs( num ) <= Number.MAX_SAFE_INTEGER;
    };
  }
  ```
  - 所能操作的最大数字为Math.pow(-2,31)（-2147483648,约－21 亿）到Math.pow(2,31) - 1（2147483647，约21 亿）
  ```
  a | 0 // 将a中变量转换为32位有符号证书
  ```
### 特殊数值
  
#### 不是值的值
  - undefined只有一个值 ———— undefined ——————  空值    —————— 曾赋过现在没有值
  - null类型也只有一个值 ———— null      ——————  没有值  —————— 从未赋值过
  
  - 非严格模式下,可以使用全局标识符undefined赋值
  - 即使在严格模式下也可以声明名为undefined的局部变量(禁止这么做)
  - void __ 可以返回undefined(因为void没有返回值)
  
#### 特殊的数字
  - NaN不等于一切 需要用windows.isNaN()来判断(但是有bug 不是数字也不是NaN也会返回true)
  - ES6后可以用Number.isNaN()来判断
  - 也可以用NaN不等于自身来判断
  - JS使用有限数字表示法(IEEE754浮点数)所以运算结果可能溢出 结果为+-Infinity
  - 可以从有穷走向无穷 但是无法从无穷走向有穷
  - 因为Infinity/Infinity是未定义操作所以返回NaN
  - 有穷正数/Infinity === 0
  - 有穷负数/Infinity === -0
  
#### 零值
  - 负数的乘除法可能会产生-0
  - 对负数的字符串化会返回-0,但是某些浏览器不符合规范
  - JS保留-0是为了保留方向信息以保证需要级数形式的程序正常运行(比如动画帧的移动速度)
  - isNegZero()判断是否是-0
  
### 特殊等式
  - Object.is()判断两个值是否绝对相等 可以处理NaN,-0这类情况
  - 能使用===尽量不要用Object.is()

### 值和引用
  - JS没有指针,引用的工作机制是指向值,如果一个值有十个引用,这些引用都指向同一个值,互相之间没有引用/指向关系
  - 简单值都是通过复制的方式来赋值/传递(null,undefined,字符串,数字,Boolean,Symbol)
  - 复合值:对象和函数 总是通过引用复制的方式来赋值/传递
  - slice()不带参数会返回当前数组的一个浅复本,由于传递给函数的是指向 该副本的引用,所以不会影响传入的参数指向的数组

## 三、原生函数
  - 原生函数是JS的内建函数
  - 原生函数可以被当做构造函数使用,但是其构造出来的对象可能会和设想的有所出入
  ```
   var a = new String( "abc" );
   typeof a; // 是"object"，不是"String
   a instanceof String; // true
   Object.prototype.toString.call(a); // "[object String]"
  ```
  通过构造函数创建出来的是封装的基本类型值的封装对象

### 内部属性[[Class]]
  所有typeof返回object的对象都包含一个内部属性[[Class]] (可以看做一个内部的分类)无法直接访问一般通过toString来查看
  ```
  Object.prototype.toString.call( [1,2,3] );
  // "[object Array]"
  Object.prototype.toString.call( /regex-literal/i );
  // "[object RegExp]"
  ```
  - 多数情况下[[Class]]属性和创建该对象的内建原生构造函数相对应,但也有例外
  ```
  Object.prototype.toString.call( null );
  // "[object Null]"
  Object.prototype.toString.call( undefined );
  // "[object Undefined]
  ```

### 封装对象包装
  - 基本类型没有.length和.toString这样的属性和方法,需要通过封装对象才能访问,这个时候JS会自动为基本类型包装一个封装对象
  - 如果需要经常用到这些字符串属性和方法,比如for循环中使用i<a.length,那么从一开始就创建一个封装对象也许更方便,这样JS引擎就不用每次都自动创建了
  - 但是事实证明这是一个弱智方法,因为浏览器已经为.length这样的常见情况进行了性能优化,手动优化反而会降低效率
  - 所以一般最好的办法是由JS殷勤决定什么时候使用封装对象(尽量少用原生函数构造函数)
  ```
  var a = new Boolean( false );
  if (!a) {
    console.log( "Oops" ); // 执行不到这里
  }
  ```
  - 因此封装基本类型值,用Object()函数(没有new)
  ```
  var a = "abc";
  var b = new String( a );
  var c = Object( a );
  typeof a; // "string"
  typeof b; // "object"
  typeof c; // "object"
  b instanceof String; // true
  c instanceof String; // true
  Object.prototype.toString.call( b ); // "[object String]"
  Object.prototype.toString.call( c ); // "[object String]"
  ```
  
### 拆封
  - 需要拆封对象中的基本类型值,可以使用valueOf()函数
  ```
  var a = new String( "abc" );
  var b = new Number( 42 );
  var c = new Boolean( true );
  a.valueOf(); // "abc"
  b.valueOf(); // 42
  c.valueOf(); // true
  ```
  - 在需要用到封装对象中的基本值的地方会发生隐式拆封

### 原生函数作为构造函数

#### Array(..)
  - 构造函数Array(..) 不要求必须带new 关键字。不带时，它会被自动补上。因此Array(1,2,3) 和new Array(1,2,3) 的效果是一样的
  - Array构造函数只带一个数字参数时,参数会被作为数组的预设长度,而非只充当数组的一个元素
  - 数组并没有预设长度这个概念。这样创建出来的只是一个空数组，只不过它的length 属性被设置成了指定的值
  - 这会导致一些怪异的行为,这一切都是已经被废止的旧特性
  - 并且不同浏览器显示的结果也不同
  - 数组join的实现代码参考:
  ```
  function fakeJoin(arr,connector) {
    var str = "";
    for (var i = 0; i < arr.length; i++) {
      if (i > 0) {
        str += connector;
      }
      if (arr[i] !== undefined) {
        str += arr[i];
      }
    }
    return str;
  }
  var a = new Array( 3 );
  fakeJoin( a, "-" ); // "--"
  ```
  - 可以通过下述方式来创建包含undefined 单元（而非“空单元”）的数组
  ```
  var a = Array.apply( null, { length: 3 } );
  a; // [ undefined, undefined, undefined ]
  ```
#### Object(..)、Function(..) 和RegExp(..)
  - 同样除非万不得已,否则尽量不要用Object/Function/RegExp
  - 用字面量定义对象
  - 构造函数Function只在极少数情况下很有用,比如说动态定义函数参数和函数体的时候,不要当作eval的替代品,基本上不会通过这种方式来定义函数
  - 用常量形式定义正则表达式,不仅语法简单,执行效率也更高,因为JS引擎在代码执行前会预编译和缓存,与前面的构造函数不同RegExp有时挺有用的
  - 比如说动态定义正则表达式
  ```
  var name = "Kyle";
  var namePattern = new RegExp( "\\b(?:" + name + ")+\\b", "ig" );
  var matches = someText.match( namePattern );
  ```