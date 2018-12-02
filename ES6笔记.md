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
  
### String.fromCodePoint()
  - ES5提供String.fromCharCode,用于从码点返回对应字符串,但是不能识别UTF-16字符
  - ES6提供了String.fromCodePoint方法,解决了这个问题,正好和codePointAt相反
  - 如果其有多个参数会被合成一个字符串返回
  
### 字符串的遍历器接口
  - ES6为字符串添加了遍历器的接口,使所有字符串可以被for-of遍历
  - 遍历器最大的优点是可以识别大于0xFFFF的码点,传统的for循环无法识别这样的码点

### normalize
  - 许多欧洲语言有语调符号和重音符号
  - ES6提供了两种方法处理这种符号
    1. 直接提供带重音符号的字符
    2. 提供合成符号
  - 两种方法视觉上等价,但是JS不能识别
  - 合成字符会被识别成两个字符串
  - normalize方法可以接受一个参数来指定normalize方式
    1. NFC
       - 默认参数
       - 标准等价合成
       - 返回多个简单字符串的合成字符串
       - 标准等价是说视觉和语义上的等价
    2. NFD
       - 标准等价分解
       - 标准等价前提下,返回合成字符分解的多个简单字符
    3. NFKC
       - 兼容等价合成
       - 返回合成字符
       - 兼容等价指的是语义上等价视觉上不等价
    4. NFKD
       - 兼容等价分解
       - 兼容等价前提下,返回合成字符分解的简单字符

### includes(),startsWith(),endsWith()
  - 传统上,JavaScript 只有indexOf方法,可以用来确定一个字符串是否包含在另一个字符串中
  - ES6又提供了三种新方法
    1. includes
       - 返回布尔值
       - 表示是否找到了字符串
    2. startsWith
       - 返回布尔值
       - 表示参数字符是否在原字符串头部
    3. endsWith
       - 返回布尔值
       - 表示参数字符是否在原字符串尾部
  - 都支持第二个参数表开始搜索的位置
  
### repeat
  - reqeat返回一个新字符串,表示将原字符串重复N次
  - 参数如果是小数,会被取整
  - 参数如果是负数或者Infinity就会报错
  - 如果是0到-1的小数,等同于0
  - NaN等同于0
  - 字符串会被转为数字
  
### padStart(),padEnd()
  - ES2017引入的字符串补全长度功能,如果字符串不够某一长度,会在头部或尾部补全
    1. padStart
       - 在头部补全
    2. padEnd
       - 在尾部补全
  - 第一个参数为补全生效的长度
  - 第二个参数表补全的字符串
  - 省略第二个参数默认补全空格
  - 一般用于补全数字位数活着提示字符串格式
  ```
  '12'.padStart(10, 'YYYY-MM-DD') // "YYYY-MM-12"
  '09-12'.padStart(10, 'YYYY-MM-DD') // "YYYY-09-12"
  ```
  
### 模版字符串
  - 模版字符串可以嵌套
  - 可以用eval或者Function构造函数引用模版字符串本身
  ```
  // 写法一
  let str = 'return ' + '`Hello ${name}!`';
  let func = new Function('name', str);
  func('Jack') // "Hello Jack!"
  
  // 写法二
  let str = '(name) => `Hello ${name}!`';
  let func = eval.call(null, str);
  func('Jack') // "Hello Jack!"
  ```
  
### 模版编译
  ```
  let template = `
  <ul>
    <% for(let i=0; i < data.supplies.length; i++) { %>
      <li><%= data.supplies[i] %></li>
    <% } %>
  </ul>
  `;
  ```
  此模版使用<%...%>放置JS代码用<%=...%>放置表达式
  - 两种编译模版字符串思路
    1. 转换为JS表达式字符串
    ```
    echo('<ul>');
    for(let i=0; i < data.supplies.length; i++) {
      echo('<li>');
      echo(data.supplies[i]);
      echo('</li>');
    };
    echo('</ul>');

    ```
      - 用正则表达式转换
    ```
    let evalExpr = /<%=(.+?)%>/g;
    let expr = /<%([\s\S]+?)%>/g;
    
    template = template
      .replace(evalExpr, '`); \n  echo( $1 ); \n  echo(`')
      .replace(expr, '`); \n $1 \n  echo(`');
    
    template = 'echo(`' + template + '`);';
    ```
    - 然后把template封装到一个函数内返回
    - 封装成一个模版编译函数
    ```
    function compile(template){
      const evalExpr = /<%=(.+?)%>/g;
      const expr = /<%([\s\S]+?)%>/g;
    
      template = template
        .replace(evalExpr, '`); \n  echo( $1 ); \n  echo(`')
        .replace(expr, '`); \n $1 \n  echo(`');
    
      template = 'echo(`' + template + '`);';
    
      let script =
      `(function parse(data){
        let output = "";
    
        function echo(html){
          output += html;
        }
    
        ${ template }
    
        return output;
      })`;
    
      return script;
    }
    // 调用
    let parse = eval(compile(template));
    div.innerHTML = parse({ supplies: [ "broom", "mop", "cleaner" ] });
    ```

### 模版标签
  - 模版字符串还能紧跟在一个函数后面,该函数被调用用于处理这个模版字符串
  ```
  alert`123`
  // 等同于
  alert(123)
  ```
  - 标签模板其实不是模板,而是函数调用的一种特殊形式
  - 标签指的就是函数,紧跟在后面的模板字符串就是它的参数
  - 如果模版字符串内有变量,会将模版字符串预先处理成多个参数,再调用函数
  ```
  let a = 5;
  let b = 10;
  
  tag`Hello ${ a + b } world ${ a * b }`;
  // 等同于
  tag(['Hello ', ' world ', ''], 15, 50);
  
  // function tag(stringArr, ...values){}
  ```
  - 可以把字符串全部重新拼回去
  ```
  function passthru(literals, ...values) {
    let output = "";
    let index;
    for (index = 0; index < values.length; index++) {
      output += literals[index] + values[index];
    }
  
    output += literals[index]
    return output;
  }
  ```
  - 模版标签的重要作用
    1. 过滤HTML字符串,防止用户输入恶意内容
       - 用模版标签转义所有用户输入的特殊字符
    2. 多语言转化
    3. 嵌入其他语言
  - 模版处理函数的第一个参数(模版字符串数组)还有一个raw属性
  - raw属性保存转义后的原字符串
  ```
  tag`First line\nSecond line`
  
  function tag(strings) {
    console.log(strings.raw[0]);
    // strings.raw[0] 为 "First line\\nSecond line"
    // 打印输出 "First line\nSecond line"
  }
  ```
  
### String.raw()
  - ES6为原生String方法提供了一个raw方法,多用于充当模版字符串的处理函数,返回一个斜杠都被转义(斜杠前再加一个斜杠)的字符串
  - raw方法既可以作为处理模版字符串的基本方法,也可以作为正常函数使用,此时第一个参数必须是一个具有raw属性的对象,并且值为数组
  ```
  String.raw({ raw: 'test' }, 0, 1, 2);
  // 't0e1s2t'
  
  // 等同于
  String.raw({ raw: ['t','e','s','t'] }, 0, 1, 2);
  ```
  - raw的实现如下
  ```
  String.raw = function (strings, ...values) {
    let output = '';
    let index;
    for (index = 0; index < values.length; index++) {
      output += strings.raw[index] + values[index];
    }
  
    output += strings.raw[index]
    return output;
  }
  ```
  
### 模板字符串的限制 
  - 模版字符串默认会把字符串转义,导致无法嵌入其他语言
  - ES6放开了对转义的限制
  
## 正则的扩展
  - ES5中,RegExp构造函数有两种情况
    1. 参数是字符串,这时第二个参数表示正则表达式的修饰符(flag)
    ```
    var regex = new RegExp(/xyz/i);
    // 等价于
    var regex = /xyz/i;
    ```
    2. 参数是一个正则表示式,这时会返回一个原有正则表达式的拷贝
    ```
    var regex = new RegExp(/xyz/i);
    // 等价于
    var regex = /xyz/i;
    ```
  - ES5不允许传递正则表达式时传递第二个参数当修饰符
  - ES6修复了这个问题,第二个参数的修饰符会覆盖原有修饰符
  
### 字符串的正则方法
  1. match
  2. replace
  3. search
  4. split
  - ES6将这4个方法,在语言内部全部调用RegExp的实例方法,从而做到所有与正则相关的方法,全都定义在RegExp对象上

### u修饰符
  - ES6对正则表达式添加了u修饰符,含义为Unicode模式
  - 用于正确处理大于\uFFFF的Unicode字符(正确处理UTF-16编码)
  ```
  /^\uD83D/u.test('\uD83D\uDC2A') // false
  /^\uD83D/.test('\uD83D\uDC2A') // true
  ```
  - u修饰符会修改以下行为
    1. 点字符
       - .字符在正则表达式中,含义是除了换行之外的任意单个字符,但是对于大于\uFFFF的字符无法识别必须加上u修饰符
    2. Unicode字符表示法
       - ES6新增了使用大括号表示Unicode字符,这种表示法必须加上u修饰符才能识别大括号
    3. 量词
       - 添加u修饰符之后,所有量词都会正确识别大于\uFFFF的字符
    4. 预定义模式
    5. i修饰符
    
 ### RegExp.prototype.unicode
  - 表示是否设置了u修饰符
  
### y修饰符
  - 除了u修饰符,ES6还为正则表达式添加了y修饰符,叫做'粘连'(sticky)修饰符
  - 与g修饰符类似,都是全局匹配,后一次匹配从前一次成功未知开始
  - 不同在于g修饰符只要剩余位置中存在匹配就行,y修饰符确保匹配必须从剩余的第一个位置开始
  ```
  var s = 'aaa_aa_a';
  var r1 = /a+/g;
  var r2 = /a+/y;
  
  r1.exec(s) // ["aaa"]
  r2.exec(s) // ["aaa"]
  
  r1.exec(s) // ["aa"]
  r2.exec(s) // null 因为剩余字符串第一个字符是_
  ```
  - y修饰符号隐含了头部匹配的标志^,y修饰符的设计本意,就是让头部匹配的标志^在全局匹配中都有效
  - 单单一个y修饰符对match方法,只能返回第一个匹配,必须与g修饰符联用,才能返回所有匹配
  ```
  'a1a2a3'.match(/a\d/y) // ["a1"]
  'a1a2a3'.match(/a\d/gy) // ["a1", "a2", "a3"]
  ```
  - y修饰符的一个应用,是从字符串提取token(词元)y修饰符确保了匹配之间不会有漏掉的字符
  ```
  const TOKEN_Y = /\s*(\+|[0-9]+)\s*/y;
  const TOKEN_G  = /\s*(\+|[0-9]+)\s*/g;
  
  tokenize(TOKEN_Y, '3 + 4')
  // [ '3', '+', '4' ]
  tokenize(TOKEN_G, '3 + 4')
  // [ '3', '+', '4' ]
  
  function tokenize(TOKEN_REGEX, str) {
    let result = [];
    let match;
    while (match = TOKEN_REGEX.exec(str)) {
      result.push(match[1]);
    }
    return result;
  }
  
  tokenize(TOKEN_Y, '3x + 4')
  // [ '3' ] 
  tokenize(TOKEN_G, '3x + 4')
  // [ '3', '+', '4' ] g会忽略非法字符x
  ```
  
### RegExp.prototype.sticky
  - 正则表达式是否包含y修饰符
  
### RegExp.prototype.sticky
  - 返回正则表达式修饰符
  
### s修饰符:dotAll模式
  - 正则表达式中,(.)是一个特殊字符,代表任意单个字符,但是有两个例外
    1. 四个字节的UTF16字符,可以用u修饰符解决
    2. 行终止符
  - 行终止符指的是一行的终结
    1. U+000A 换行符（\n）
    2. U+000D 回车符（\r）
    3. U+2028 行分隔符（line separator）
    4. U+2029 段分隔符（paragraph separator）
  - s修饰符可以让.匹配任意字符
  - 这被称作dotAll模式,因此还引入了一个dotAll属性,表示正则表达式是否有s修饰符
  - /s修饰符和多行修饰符/m不冲突,两者一起使用的情况下,.匹配所有字符,而^和$匹配每一行的行首和行尾

### 后行断言
  - JavaScript语言的正则表达式,只支持先行断言(lookahead)和先行否定断言(negative lookahead)不支持后行断言(lookbehind)和后行否定断言(negative lookbehind)
  - ES2018 引入后行断言,V8引擎4.9版(Chrome 62)已经支持
  - 先行断言
    - 只有x在y前面才匹配,必须写成`/x(?=y)/`
  - 先行否定断言
    - x只有不在y前面才匹配,必须写成`/x(?!y)/`
  - 断言内部的内容如y是不计入返回结果里的
  - 后行断言正好与先行断言相反,x只有在y后面才匹配,必须写成`/(?<=y)x/`
  - 后行否定断言,`/(?<!y)x/`
  - 括号内部分也不计入返回结果
  - 后行断言从后开始匹配
  
### Unicode属性类
  - ES6引入了新的类写法`\p{}`和`\P{}`,允许正则匹配符合Unicode某种属性的所有字符
  ```
  const regexGreekSymbol = /\p{Script=Greek}/u;
  regexGreekSymbol.test('π') // true
  ```
  - 上面代码中,`\p{Script=Greek}`指定匹配一个希腊文字母,所以匹配π成功
  - Unicode属性类要指定属性名和属性值
  ```
  \p{UnicodePropertyName=UnicodePropertyValue}
  ```
  - `\P{}`是`\p{}`的反向匹配
  - 只对Unicode有效
  ```
  const regex = /^\p{Decimal_Number}+$/u;
  regex.test('𝟏𝟐𝟑𝟜𝟝𝟞𝟩𝟪𝟫𝟬𝟭𝟮𝟯𝟺𝟻𝟼') // true
  // 属性类指定匹配所有十进制字符,可以看到各种字型的十进制字符都会匹配成功。
  ```
  - `\p{Number}`甚至能匹配罗马数字
  - 其他一些例子
  ```
  // 匹配所有空格
  \p{White_Space}
  
  // 匹配各种文字的所有字母，等同于 Unicode 版的 \w
  [\p{Alphabetic}\p{Mark}\p{Decimal_Number}\p{Connector_Punctuation}\p{Join_Control}]
  
  // 匹配各种文字的所有非字母的字符，等同于 Unicode 版的 \W
  [^\p{Alphabetic}\p{Mark}\p{Decimal_Number}\p{Connector_Punctuation}\p{Join_Control}]
  
  // 匹配 Emoji
  /\p{Emoji_Modifier_Base}\p{Emoji_Modifier}?|\p{Emoji_Presentation}|\p{Emoji}\uFE0F/gu
  
  // 匹配所有的箭头字符
  const regexArrows = /^\p{Block=Arrows}+$/u;
  regexArrows.test('←↑→↓↔↕↖↗↘↙⇏⇐⇑⇒⇓⇔⇕⇖⇗⇘⇙⇧⇩') // true
  ```
  
### 具名组匹配
  - ES2018引入了具名组匹配,允许给匹配组取名
  ```
  const RE_DATE = /(\d{4})-(\d{2})-(\d{2})/;
  // 不易读
  const matchObj = RE_DATE.exec('1999-12-31');
  const year = matchObj[1]; // 1999
  const month = matchObj[2]; // 12
  const day = matchObj[3]; // 31
  
  const RE_DATE = /(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})/;
  // 易读
  ```
  
### 解构赋值和替换
  - 有了具名组匹配以后,可以使用解构赋值直接从匹配结果上为变量赋值
  - 字符串替换时,使用$<组名>引用具名组
  ```
  let re = /(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})/u;
  
  '2015-01-02'.replace(re, '$<day>/$<month>/$<year>')
  // '02/01/2015'
  ```
  - 可以通过`\k<组名>`调用具名组匹配
  ```
  const RE_TWICE = /^(?<word>[a-z]+)!\k<word>$/;
  RE_TWICE.test('abc!abc') // true
  RE_TWICE.test('abc!ab') // false
  ```
### String.prototype.matchAll
  - 还在提按阶段
  - 一次性返回所有匹配,但是返回一个Iterator,因此可以用for-of取出
  
## 数值的拓展

### 二进制和八进制表示法
  - ES6提供了二进制和八进制数值的新的写法,分别用前缀0b(或0B)和0o(或0O)表示
  ```
  0b111110111 === 503 // true
  0o767 === 503 // true
  ```
  - 从ES5开始,在严格模式之中,八进制就不再允许使用前缀0表示,ES6进一步明确,要使用前缀0o表示
  - 如果要将0b和0o前缀的字符串数值转为十进制,要使用Number方法
    
### Number.isFinite(),Number.isNaN()
  - Number.isFinite()用来检测一个数值是否是有限的
  - Number.isNaN()用来检测一个数值是否是NaN
  
### Number.parseInt(),Number.parseFloat()
  - 从全局方法移到了Number对象上

### Number.isInteger()
  - Number.isInteger()判断一个数字是否是整数
  - JS中整数和浮点数用的是相同的存储方式,25和25.0是同样的值
  - 由于JavaScript采用IEEE754标准,数值存储为64位双精度格式,数值精度最多可以达到53个二进制位(1个隐藏位与52个有效位)
  - 如果数值的精度超过这个限度,第54位及后面的位就会被丢弃,这种情况下,Number.isInteger可能会误判
  - 如果小于5E-324(JS最小值)可能也会被误判
    
### Number.EPSILON
  - ES6在Number对象上面新增了一个极小的常量,Number.EPSILON,它表示1和大于1之间最小浮点数之间的差
  - 对于64位浮点数来说,相当于1后面51个0接1,这个值减去1之后等于2的-52次方
  - 它实际上是JS能表示的最小精度,误差小于这个值就没有意义了,相当于不存在误差

### 安全整数和Number.isSafeInteger()
  - JS能够准确表示的整数范围在-2^53到2^53之间(不含两个端点)超过这个范围,无法精确表示这个值
  - ES6引入了Number.MAX_SAFE_INTEGER和Number.MIN_SAFE_INTEGER来表示最大值和最小值
  - Number.isSafeInteger()是用来查看数值是否在这个范围
  
### Math对象的扩展
#### Math.trunc()
  - 去除小数部分,返回整数部分
  - 对于空值和无法截取整数的值,返回NaN
  
#### Math.sign()
  - 用来判断一个数到底是正数、负数、还是零
  - 对于非数值，会先将其转换为数值。
    1. 正数返回+1
    2. 负数返回-1
    3. 0返回0
    4. -0返回-0
    5. 其他返回NaN
#### Math.cbrt()
  - 用于计算一个数的立方根

#### Math.clz32() 
  - 返回一个32位无符号整数形式有多少个0在前面
  - 与<<左移运算符计算息息相关
  - 对于小数,只考虑整数部分
  
#### Math.imul()
  - 返回两个32位带符号整数形式相乘的结果
  - 适用于超过2的53次方的数,不丢失精度
  
#### Math.fround()
  - 返回一个数的32位单精度浮点数形式
  - 主要作用是将64位双精度浮点数转为32位单精度浮点数,如果小数的精度超过24个二进制位,返回值就会不同于原值,否则返回值不变(即与64位双精度值一致)。
  
#### Math.hypot()
  - 返回所有参数的平方和的平方根
  
#### 对数方法
  - 新增了四个对数方法
  1. Math.expm1()
     - 返回 ex - 1，即Math.exp(x) - 1
  2. Math.log10()
     - 返回以10为底的x的对数,如果x小于0则返回NaN
  3. Math.log2()
     - 返回以2为底的对数
  4. Math.log1p()
     - 返回1+x的自然对数,即Math.log(1 + x),如果x小于-1，返回NaN
     
#### 双曲函数方法
  1. Math.sinh(x) 返回x的双曲正弦
  2. Math.cosh(x) 返回x的双曲余弦
  3. Math.tanh(x) 返回x的双曲正切
  4. Math.asinh(x) 返回x的反双曲正弦
  5. Mat 1. h.acosh(x) 返回x的反双曲余弦
  6. Math.atanh(x) 返回x的反双曲正切
  
#### 指数运算符
  - 新增`**`运算符用于指数运算
  ```
  2 ** 2 // 4
  2 ** 3 // 8
  ```
  - 是右结合
  ```
  // 相当于 2 ** (3 ** 2)
  2 ** 3 ** 2
  // 512
  ```
  - 数运算符可以与等号结合,形成一个新的赋值运算符(**=)
  - V8引擎的指数运算符与Math.pow的实现不相同,对于特别大的运算结果,两者会有细微的差异
  
## 函数的拓展

### 函数参数的默认值
  - ES6之前不能为函数参数设置默认值,只能在函数内部检查
  ```
  function log(x, y) {
    y = y || 'World'; // 无法判定false参数
    console.log(x, y);
  }
  function log(x,y){ // 可以判定false
    if (typeof y === 'undefined') {
      y = 'World';
    }
  }
  ```
  - ES6允许设置默认值
  ```
  function log(x, y = 'World') {
    console.log(x, y);
  }
  ```
  - 提高可读性,让人一目了然哪些参数可以被忽略
  - 参数变量是默认声明,不能被let和const再次声明
  - 不能有同名参数
  - 参数默认值不是传值,每次都要重新计算默认值,也就是惰性求值
  
#### 与解构赋值默认值结合使用
  ```
  function foo({x, y = 5}) {
    console.log(x, y);
  }
  ```
  - 可以为对象参数设置默认值但是不能省略参数
  ```
  function fetch(url, { body = '', method = 'GET', headers = {} }) {
    console.log(method);
  }
  
  fetch('http://example.com', {})
  // "GET"
  
  fetch('http://example.com')
  // 报错
  ```
  - 可以用双重默认值解决这个问题
  ```
  function fetch(url, { body = '', method = 'GET', headers = {} } = {}) {
    console.log(method);
  }
  
  fetch('http://example.com')
  // "GET"
  ```
  
#### 参数默认值的位置
  - 如果不把默认参数放在尾部,是无法省略的
  - 但是可以传入undefined,跳过这个参数
  - 传入null无效
  
### 函数的length属性
  - 函数的length属性返回没有默认值的参数个数,也就是说设置了默认值之后length属性失真
  - length属性的含义是该函数预期传入的参数个数
  
### 作用域
  - 一旦设置了默认值,函数就会形成一个单独的作用域,等初始化结束,就会消失,这种语法行为在没有默认参数的时候是不存在的
  ```
  var x = 1;
  
  function f(x, y = x) {
    console.log(y);
  }
  
  f(2) // 2
  ```
  - 因为设置了默认参数,所以参数形成一个独有作用域,所以内部的y选择了前面定义的参数x作为参数
  - 同时如果此时作用域没有x,就会报错
  - 给x赋值x也会报错
  ```
  var x = 1
  function foo(x = x) {
    // ...
  }
  foo() // ReferenceError: x is not defined
  ```
  - 参数`x = x`形成一个单独作用域,实际执行的是`let x = x`,由于暂时性死区的原因,这行代码会报错"x 未定义"
  - 如果参数默认值是一个函数,也遵循这个规则
  ```
  let foo = 'outer';
  function bar(func = () => foo) {
    let foo = 'inner';
    console.log(func());
  }
  bar(); // outer
  ```
  - 函数bar的参数func的默认值是一个匿名函数,返回值为变量foo，函数参数形成的单独作用域里面,并没有定义变量foo,所以foo指向外层的全局变量foo,因此输出outer
  - 利用参数默认值,可以指定某一个参数不得省略,如果省略就抛出一个错误。
  ```
  function throwIfMissing() {
    throw new Error('Missing parameter');
  }
  
  function foo(mustBeProvided = throwIfMissing()) {
    return mustBeProvided;
  }
  
  foo()
  // Error: Missing parameter
  // 没有参数就会调用默认值throwIfMissing函数,从而抛出一个错误
  ```
  - 另外,可以将参数默认值设为undefined,表明这个参数是可以省略的
  
### reset参数
  - ES6引入reset参数,形式为...参数名,用于引入多余参数
  - reset参数搭配的是一个数组,这样就能避免使用argument
  - argument只是个类数组对象,如果要用数组方法,需要Array.prototype.slice.call转换为数组
  - reset参数之后不能有其他参数,否则会报错
  - 函数的length不包含reset参数
  
### 严格模式
  - ES2016规定了函数内部如果使用了默认参数,解构赋值,或者拓展运算符,函数内部不能再定义为严格模式
  - 函数内部的严格模式,同时适用于函数体和函数参数,执行时先执行函数参数,再执行函数体,但是严格模式定义在函数体中
  - 解决方式
    1. 全局严格模式
    2. 把函数包在一个无参数的立即执行函数里面
    
### name属性
  - 函数的name属性返回函数名
  - 大部分浏览器早就支持,ES6才写入标准,并作出了调整,能返回匿名函数实际执行的函数名
  ```
  let f = function () {};
  // ES5
  f.name // ""
  // ES6
  f.name // "f"
  ```
  - Function构造函数返回的函数实例的name是anonymous
  - bind函数返回的函数,name属性会加上bound属性
  
### 箭头函数
  - ES6允许使用箭头函数定义匿名函数
  - 空参用括号表示参数,单参数可以省略括号
  - 多于一条语句需要用大括号
  - 由于大括号被解释为代码块,所以如果箭头函数返回一个对象,对象外面必须加大括号
  - 如果箭头函数,只有一行语句,且不需要返回值,可以使用void
  ```
  let fn = () => void doesNotReturn()
  ```
  - 箭头函数注意点
    1. 函数体内this对象,就是定义时的对象,而不是使用时的对象
    2. 不可以当作构造函数,也就是不能使用new命令
    3. 不可以使用arguments对象,该对象在函数体内不存在,用reset对象替代
    4. 不可食用yield对象
  - 箭头函数没有this,arguments,super,new.target