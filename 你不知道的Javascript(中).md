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
  // 无效语法:
  42.toFixed( 3 ); // SyntaxError 42.被引擎识别为浮点数
  // 下面的语法都有效:
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
  - 从ES6 开始,该值定义在Number.EPSILON 中,我们可以直接拿来用,也可以为ES6 之前的版本写polyfill:
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
  - 所能安全呈现的最大整数是2^53 - 1,即9007199254740991 被定义在Number.MAX_SAFE_INTEGER
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
  - 所能操作的最大数字为Math.pow(-2,31)（-2147483648,约－21 亿）到Math.pow(2,31) - 1（2147483647,约21 亿）
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
   typeof a; // 是"object",不是"String
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
  - 构造函数Array(..) 不要求必须带new 关键字。不带时,它会被自动补上。因此Array(1,2,3) 和new Array(1,2,3) 的效果是一样的
  - Array构造函数只带一个数字参数时,参数会被作为数组的预设长度,而非只充当数组的一个元素
  - 数组并没有预设长度这个概念。这样创建出来的只是一个空数组,只不过它的length 属性被设置成了指定的值
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
  
#### Date(..)和Error(..)
  - ES5之后可以用Date.now()替代new Date().getTime()
  - Error()可以不带new,创建错误对象主要是为了获得当前栈的上下文,大部分JS引擎通过只读属性.stack来访问
  - 栈上下文信息包括函数调用栈信息和产生错误的代码行号
  - 错误对象一般与throw一起使用
  - 通常错误对象至少包含一个message属性,有时也有其他属性(必须作为只读属性访问),如type
  - 除了Error()外还有一些针对指定错误类型的原生构造函数如EvalError(),RangeError()等 很少手动调用都是自动调用
#### Symbol(..)
  - 符号是具有唯一性的特殊值(并非绝对)用它来命名对象属性不容易导致重名,该类型引入主要源于ES6的一些特殊构造
  - 符号可以用作属性名,但无论是在代码还是开发控制台中都无法查看和访问它的值
  - ES6中有一些预定义符号,以Symbol的静态属性形式出现,如Symbol.create、Symbol.iterator等
    ```
    obj[Symbol.iterator] = function(){ /*..*/ };
    ```     
  - 可以使用Symbol(..) 原生构造函数来自定义符号。但它比较特殊,不能带new关键字否则会出错
    ```
    var mysym = Symbol( "my own symbol" );
    mysym; // Symbol(my own symbol)
    mysym.toString(); // "Symbol(my own symbol)"
    typeof mysym; // "symbol"
    var a = { };
    a[mysym] = "foobar";
    Object.getOwnPropertySymbols( a );
    // [ Symbol(my own symbol) ]
    ```
  - 主要用于私有或特殊属性,可以用于替代有下划线(_)前缀的属性

#### 原生原型
  - 原生构造函数有自己的.prototype 对象,如Array.prototype、String.prototype 等
  - 这些对象包含其对应子类型所持有的行为特征
  - 将字符串封装为字符串对象之后就能访问String.prototype中定义的方法
  ```
  String#indexOf(..)
  String#charAt(..)
  String#substr(..)
  String#substring(..)
  String#slice(..)
  String#toUpperCase()
  String#toLowerCase()
  String#trim()
  ```
  - 以上方法并不改变原字符串的值,而是返回一个新字符串。
  - 借助原型代理,所有字符串都可以访问这些方法
  - 其他构造函数的原型包含它们各自类型所特有的行为特征比如Number#tofixed(..)（将数字转换为指定长度的整数字符串）和Array#concat(..)（合并数组）
  - 所有的函数都可以调用Function.prototype中的apply/call/bind
  
  - 原生原型（native prototype）并非普通对象那么简单
  ```
  typeof Function.prototype; // "function"
  Function.prototype(); // 空函数！
  RegExp.prototype.toString(); // "/(?:)/"——空正则表达式
  "abc".match( RegExp.prototype ); // [""]
  Array.isArray( Array.prototype ); // true
  Array.prototype.push( 1, 2, 3 ); // 3
  Array.prototype; // [1,2,3]
  // 需要将Array.prototype设置回空,否则会导致问题！
  Array.prototype.length = 0;
  ```
  - Function.prototype 是一个空函数,RegExp.prototype是一个空正则表达式,而Array.prototype是一个空数组

## 四、强制类型转换

### 值类型转换
   - 类型转换发生在静态类型语言编译时,强制类型转换发生在动态类型语言的运行时
   - 强制类型转换分为"隐式强制类型转换"和"显式强制类型转换"
   ```
   var a = 42;
   var b = a + ""; // 隐式强制类型转换
   var c = String( a ); // 显式强制类型转换
   ```
   
### ToString
   - 基本类型值的字符串化规则为
     - null 转换为"null"
     - undefined 转换为"undefined"
     - true转换为"true"
   - 对普通对象来说,除非自行定义,否则toString返回内部属性的[[Class]]的值(如[Object,Object])
   - 如果对象有自己的toString() 方法,字符串化时就会调用该方法并使用其返回值
   - 数组的默认toString() 方法经过了重新定义,将所有单元字符串化以后再用"," 连接起来
   ```
   var a = [1,2,3];
   a.toString(); // "1,2,3"
   ```
#### JSON.stringify()
   - JSON.stringify(..)在序列化为字符串时也用到了ToString,
   - 与toString()效果基本相同只不过序列化结果总是字符串
   ```
   JSON.stringify( 42 ); // "42"
   JSON.stringify( "42" ); // ""42"" （含有双引号的字符串）
   JSON.stringify( null ); // "null"
   JSON.stringify( true ); // "true"
   ```
   - 不安全的JSON 值:undefined、function、symbol、循环引用都不符合JSON结构标准,支持JSON的语言无法处理
   - JSON.stringify(..) 在对象中遇到undefined、function 和symbol 时会自动将其忽略,在数组中则会返回null（以保证单元位置不变）
   - 如果对象中定义了toJSON方法,JSON字符串化时会首先调用该方法,然后用它的返回值来进行调用JSON.stringify(..)序列化
   - JSON.stringify(..) 传递一个可选参数replacer,它可以是数组或者函数,用来指定对象序列化过程中哪些属性应该被处理,哪些应该被排除,和toJSON() 很像。
   - 如果replacer 是一个数组,那么它必须是一个字符串数组,其中包含序列化要处理的对象的属性名称,除此之外其他的属性则被忽略。
   - 如果replacer 是一个函数,它会对对象本身调用一次,然后对对象中的每个属性各调用一次,每次传递两个参数,键和值。如果要忽略某个键就返回undefined,否则返回指定的值。
   ```
   var a = {
     b: 42,
     c: "42",
     d: [1,2,3]
   };
   JSON.stringify( a, ["b","c"] ); // "{"b":42,"c":"42"}"
   JSON.stringify( a, function(k,v){
     if (k !== "c") return v;
   } ); // "{"b":42,"d":[1,2,3]}"
   ```
   - 如果replacer 是函数,它的参数k 在第一次调用时为undefined
   - JSON.stringfy还有一个可选参数space,用来指定输出的缩进格式。space 为正整数时是指定每一级缩进的字符数,它还可以是字符串,此时最前面的十个字符被用于每一级的缩进
   ```
   var a = {
     b: 42,
     c: "42",
     d: [1,2,3]
   };
   JSON.stringify( a, null, 3 );
   // "{
   //    "b": 42,
   //    "c": "42",
   //    "d": [
   //       1,
   //       2,
   //       3
   //    ]
   // }"
   JSON.stringify( a, null, "-----" );
   // "{
   // -----"b": 42,
   // -----"c": "42",
   // -----"d": [
   // ----------1,
   // ----------2,
   // ----------3
   // -----]
   // }"
   ```
 
### ToNumber
   - ToNumber 对字符串的处理基本遵循数字常量的相关规则/语法,处理失败时返回NaN
   - ToNumber 对以0 开头的十六进制数并不按十六进制处理,而是按十进制
   - 对象（包括数组）会首先被转换为相应的基本类型值,如果返回的是非数字的基本类型值,则再遵循以上规则将其强制转换为数字
   - 为了将值转换为相应的基本类型值,抽象操作ToPrimitive会首先（通过内部操作DefaultValue）检查该值是否有valueOf() 方法
   - 如果有并且返回基本类型值,就使用该值进行强制类型转换。如果没有就使用toString()的返回值（如果存在）来进行强制类型转换
   - 如果valueOf() 和toString() 均不返回基本类型值,会产生TypeError 错误
   - 从ES5 开始,使用Object.create(null) 创建的对象[[Prototype]] 属性为null,并且没有valueOf() 和toString() 方法,因此无法进行强制类型转换
   
### ToBoolean
   - JavaScript 中的值可以分为以下两类
     1. 可以被强制类型转换为false 的值
     2. 其他（被强制类型转换为true 的值）
        - undefined
        - null
        - false
        - +0、-0 和NaN
        - ""

### 显式强制类型转换

#### 字符串和数字之间的显式转换
   ```
   Number()
   String()
   '1'.toString()
   +'1'(容易产生误会)
   ```
#### 日期显式转换为数字
   ```
   +new Date()
   +new Date   // JS的奇特语法 构造函数没有参数的时候可以不带()
   new Date().getTime()
   Date.now()
   ```
#### ~运算符
   ```
   // |运算符的空操作可以执行ToInt32转换
   0 | -0; // 0
   0 | NaN; // 0
   0 | Infinity; // 0
   0 | -Infinity; // 0
   // ~返回数字的补码
   ~42; // -(42+1) ==> -43
   // 用于处理indexOf返回-1时转换为0(假值)
   Math.floor( -49.6 ); // -50
   ~~-49.6; // -49      //两次转换
   ```
#### 显式解析数字字符串
   - 解析允许字符串中含有非数字字符,解析按从左到右的顺序,如果遇到非数字字符就停止。而转换不允许出现非数字字符,否则会失败并返回NaN
   ```
   var a = "42";
   var b = "42px";
   Number( a ); // 42
   parseInt( a ); // 42
   Number( b ); // NaN
   parseInt( b ); // 42
   ```
   - parseInt(..)最好添加第二个参数否则会出现bug 根据第一个字符决定基数
   - ES5 开始parseInt(..) 默认转换为十进制数,除非另外指定
#### 解析非字符串
   ```
   parseInt( 1/0, 19 ); // 18 1/0是Infinity parseInt('I',19)为18
   ```
   - 非字符串会被首先强制转换成能解析的字符串,然后转换成parseInt("Infinity", 19)
   ```
   parseInt( 0.000008 ); // 0 ("0" 来自于 "0.000008")
   parseInt( 0.0000008 ); // 8 ("8" 来自于 "8e-7")
   parseInt( false, 16 ); // 250 ("fa" 来自于 "false")
   parseInt( parseInt, 16 ); // 15 ("f" 来自于 "function..")
   parseInt( "0x10" ); // 16
   parseInt( "103", 2 ); // 2
   ```
#### 显式转换为布尔值
   ```
   var a = "0";
   var b = [];
   var c = {};
   var d = "";
   var e = 0;
   var f = null;
   var g;
   Boolean( a ); // true
   Boolean( b ); // true
   Boolean( c ); // true
   Boolean( d ); // false
   Boolean( e ); // false
   Boolean( f ); // false
   Boolean( g ); // false
   以上用法不常用
   var b = [];
   var c = {};
   var d = "";
   var e = 0;
   var f = null;
   var g;
   !!a; // true
   !!b; // true
   !!c; // true
   !!d; // false
   !!e; // false
   !!f; // false
   !!g; // false
   ```
#### 隐式转换为布尔值
   - 如果+的其中一个操作符是字符串那么执行字符串拼接,否则执行数字加法
   - a + ""（隐式）和前面的String(a)（显式）之间有一个细微的差别
   - 根据ToPrimitive 抽象操作规则,a + "" 会对a 调用valueOf() 方法,然后通过ToString 抽象操作将返回值转换为字符串
   - 而String(a) 则是直接调用ToString()
   ```
   var a = {
     valueOf: function() { return 42; },
     toString: function() { return 4; }
   };
   a + ""; // "42"
   String( a ); // "4"
   ```
   - 定制valueOf() 和toString() 方法时需要特别小心,因为这会影响强制类型转换的结果
   
#### 布尔值到数字的隐式强制类型转换
   - true+=xxx会进行隐式转换转换成1并进行累加
        
#### 隐式强制转换成布尔值
   - if、for、while、?:三目运算符、||和&&会触发隐式转换把非布尔值转换为布尔值

#### ||和&&
   - 与其他语言不同,JS的||和&&返回的并不是布尔值而是两个操作数中的一个
   - 然后通过if等操作符的隐式强制转换转换为布尔值
   - || 和&& 首先会对第一个操作数执行条件判断,如果其不是布尔值（如上例）就先进行ToBoolean 强制类型转换,然后再执行条件判断。
   ```
   a = a || "hello"
   a && foo()        // 守护运算符
   ```

#### 符号的强制类型转换
   - ES6允许从符号到字符串的显式强制类型转换,但是隐式类型转换会发生错误
   ```
   var s1 = Symbol( "cool" );
   String( s1 ); // "Symbol(cool)"
   
   var s2 = Symbol( "not cool" );
   s2 + ""; // TypeError
   ```
   - 符号不能够被强制类型转换为数字（显式和隐式都会产生错误）,但可以被强制类型转换为布尔值（显式和隐式结果都是true）

#### 宽松相等和严格相等
   - == 允许在相等比较中进行强制类型转换,而=== 不允许

#### 相等比较操作的性能
   - 在会发生强制类型转换的情况下==和===的性能差异只有微秒级的差异,不用考虑

#### 抽象相等
   - == 最容易出错的一个地方是true 和false 与其他类型之间的相等比较
     - 规范11.9.3.6-7
       1. 如果Type(x) 是布尔类型,则返回ToNumber(x) == y 的结果
       2. 如果Type(y) 是布尔类型,则返回x == ToNumber(y) 的结果
       ```
       var a = "42";
       var b = true;
       a == b; // false b会转换为1
       ```
     - 规范11.9.3.2-3
       1. 如果x 为null,y 为undefined,则结果为true
       2. 如果x 为undefined,y 为null,则结果为true

     - 规范11.9.3.8-9
       1. 如果Type(x) 是字符串或数字,Type(y) 是对象,则返回x == ToPrimitive(y) 的结果
       2. 如果Type(x) 是对象,Type(y) 是字符串或数字,则返回ToPromitive(x) == y 的结果
       ```
       var a = 42;
       var b = [ 42 ]; // ToPrimitive转换为'42'
       a == b; // true
       
       var a = "abc";
       var b = Object( a ); // 和new String( a )一样
       a === b; // false
       a == b; // true b被拆封
       ```
   - 但有一些值不这样,原因是== 算法中其他优先级更高的规则
   ```
   var a = null;
   var b = Object( a ); // 和Object()一样
   a == b; // false
   
   var c = undefined;
   var d = Object( c ); // 和Object()一样
   c == d; // false
   
   var e = NaN;
   var f = Object( e ); // 和new Number( e )一样
   e == f; // false
   ```
   - 因为没有对应的封装对象,所以null 和undefined 不能够被封装（boxed）,Object(null)和Object() 均返回一个常规对象
   - NaN 能够被封装为数字封装对象,但拆封之后NaN == NaN 返回false,因为NaN 不等于NaN
   - == 中的隐式强制类型转换最为人诟病的地方是假值的相等比较
   ```
   "0" == null; // false
   "0" == undefined; // false
   "0" == false; // true
   ... ... ... ...
   ```
   - 极端情况
   ```
   [] == ![]
   2 == [2]; // true
   "" == [null]; // true
   0 == "\n"
   ```
   - 抽象关系比较
     - 比较双方先调用ToPrimitive
     - 如果结果出现非字符串,就根据ToNumber 规则将双方强制类型装换为数字进行比较
     - 如果比较双方都是字符串,则按字母顺序来进行比较
     - JS中<=是不大于的意思
   
## 五、语法
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   