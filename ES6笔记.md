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
    - 在代码块内,使用let命令声明变量之前,该变量都是不可用的这在语法上,称为"暂时性死区"
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
    - 函数里面的this,如果函数不是作为对象的方法运行,而是单纯作为函数运行,this会指向顶层对象,但是,严格模式下,这时this会返回undefined
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
  - 解构不仅可以用于数组,还可以用于对象
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
  - 对象的解构赋值,可以很方便地将现有对象的方法,赋值到某个变量
  ```
  let { log, sin, cos } = Math;
  ```
  - 由于数组本质是特殊的对象,因此可以对数组进行对象属性的解构
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
  // 属性类指定匹配所有十进制字符,可以看到各种字型的十进制字符都会匹配成功
  ```
  - `\p{Number}`甚至能匹配罗马数字
  - 其他一些例子
  ```
  // 匹配所有空格
  \p{White_Space}
  
  // 匹配各种文字的所有字母.等同于 Unicode 版的 \w
  [\p{Alphabetic}\p{Mark}\p{Decimal_Number}\p{Connector_Punctuation}\p{Join_Control}]
  
  // 匹配各种文字的所有非字母的字符.等同于 Unicode 版的 \W
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
  - 对于非数值.会先将其转换为数值
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
  - 主要作用是将64位双精度浮点数转为32位单精度浮点数,如果小数的精度超过24个二进制位,返回值就会不同于原值,否则返回值不变(即与64位双精度值一致)
  
#### Math.hypot()
  - 返回所有参数的平方和的平方根
  
#### 对数方法
  - 新增了四个对数方法
  1. Math.expm1()
     - 返回 ex - 1.即Math.exp(x) - 1
  2. Math.log10()
     - 返回以10为底的x的对数,如果x小于0则返回NaN
  3. Math.log2()
     - 返回以2为底的对数
  4. Math.log1p()
     - 返回1+x的自然对数,即Math.log(1 + x),如果x小于-1.返回NaN
     
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
  - 函数bar的参数func的默认值是一个匿名函数,返回值为变量foo.函数参数形成的单独作用域里面,并没有定义变量foo,所以foo指向外层的全局变量foo,因此输出outer
  - 利用参数默认值,可以指定某一个参数不得省略,如果省略就抛出一个错误
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
  
#### 不适用场景
  1. 定义函数的方法,该方法内部包含this
     - 比如说Vue内部钩子函数如果使用箭头函数
  2. 需要使用动态this的时候也不要使用箭头函数
     - 事件绑定调用动态作用域的方法的时候
     
### 双冒号运算符
  - 提案阶段:双冒号函数绑定运算符取代call、apply、bind
  ```
  foo::bar;
  // 等同于
  bar.bind(foo);
  
  foo::bar(...arguments);
  // 等同于
  bar.apply(foo, arguments);
  
  const hasOwnProperty = Object.prototype.hasOwnProperty;
  function hasOwn(obj, key) {
    return obj::hasOwnProperty(key);
  }
  ```
  - 如果双冒号左边为空,右边是一个对象的方法,则等于将该方法绑定在该对象上面
  ```
  var method = obj::obj.foo;
  // 等同于
  var method = ::obj.foo;
  
  let log = ::console.log;
  // 等同于
  var log = console.log.bind(console);
  ```
  
### 尾调用优化
  - 尾调用是函数最后一步调用另一个函数
  - 尾调用调用后不能有赋值等后续操作
  - 函数调用会在内存形成一个调用记录的帧,直到函数运行结束才会消失
  - 尾调是因为函数最后一步操作,不需要保留外层的调用帧,因为调用位置,内部信息等信息都不会再用到了,只要直接使用内存的调用帧
  ```
  function f() {
    let m = 1
    let n = 2
    return g(m + n)
  }
  f()
  // 等同于
  function f() {
    return g(3)
  }
  f()
  // 等同于
  g(3)
  ```
  - 只有不再用到外层函数的内部变量,内层函数的调用帧才会取代外层函数
  
#### 尾递归
  - 函数调用自身,称为递归,如果尾调用自身称为尾递归
  - 递归非常耗费内存,因为同时保存了成百上千个调用帧,很容易发生栈溢出
  ```
  function factorial(n) {
    if (n === 1) return 1;
    return n * factorial(n - 1);
  }
  
  factorial(5) // 120 复杂度O(n)

  // 优化后
  function factorial(n, total) {
    if (n === 1) return total;
    return factorial(n - 1, n * total);
  }
  
  factorial(5, 1) // 120 复杂度O(1)
  ```
  - 非尾递归的Fibonacci数列
  ```
  function Fibonacci (n) {
    if ( n <= 1 ) {return 1};
  
    return Fibonacci(n - 1) + Fibonacci(n - 2);
  }
  
  Fibonacci(10) // 89
  Fibonacci(100) // 堆栈溢出
  Fibonacci(500) // 堆栈溢出
  ```
  - 尾递归的Fibonacci数列
  ```
  function Fibonacci2 (n , ac1 = 1 , ac2 = 1) {
    if( n <= 1 ) {return ac2};
  
    return Fibonacci2 (n - 1, ac2, ac1 + ac2);
  }
  
  Fibonacci2(100) // 573147844013817200000
  Fibonacci2(1000) // 7.0330367711422765e+208
  Fibonacci2(10000) // Infinity
  ```
  - ES6第一次规定,所有ES的实现必须部署尾调用优化,用于节省内存
  - ES6的尾调优化只在严格模式开启,因为正常模式,函数内部有两个变量,可以跟踪函数的调用栈
    1. func.arguments 返回函数参数的个数
    2. func.caller  返回调用当前函数的那个函数
  - 尾调优化发生时,函数的调用栈会被改写,上面两个参数会失真
  - 非严格模式用循环替代递归
  ```
  // 蹦床函数
  function trampoline(f) {
    while (f && f instanceof Function) {
      f = f() // 接收到返回的函数执行
    }
    return f
  }
  ```
  - 具体实现
  ```
  function tco(f) {
    var value;
    var active = false;
    var accumulated = [];
  
    return function accumulator() {
      accumulated.push(arguments);
      if (!active) {
        active = true;
        while (accumulated.length) {
          value = f.apply(this, accumulated.shift());
        }
        active = false;
        return value;
      }
    };
  }
  
  var sum = tco(function(x, y) {
    if (y > 0) {
      return sum(x + 1, y - 1)
    }
    else {
      return x
    }
  });
  
  sum(1, 100000)
  // 100001
  ```

### 函数参数的尾逗号
  - ES2017允许函数最后一个参数有尾逗号
  
## 数组

### 拓展运算符
  - 扩展运算符(spread)是三个点(...),它好比rest参数的逆运算,将一个数组转为用逗号分隔的参数序列
  - 拓展运算符后面甚至可以加表达式
  ```
  const arr = [
    ...(x > 0 ? ['a'] : []),
    'b',
  ];
  ```
  - 如果拓展运算符后面是空数组,则不会有任何效果
  - 扩展运算符如果放在括号中,JavaScript引擎就会认为这是函数调用,然后就会报错
  
### 替代函数apply方法
  - 拓展运算符可以展开数组,不再需要用apply将数组转化为函数的参数

### 拓展运算符的运用
  1. 复制数组
     - ES5用concat
     - ES6直接拓展运算符
     - 两种写法一样,都是浅复制
  2. 合并数组
  3. 与解构赋值结合
  ```
  // ES5
  a = list[0], rest = list.slice(1)
  // ES6
  [a, ...rest] = list
  ```
  4. 字符串
     - 可以让字符串转换为真正的数组,并且能够正确识别四个字节的Unicode字符
  ```
  [...'hello']
  ```
  5. 实现了Iterator接口的对象
     - 任何实现了Iterator接口的对象都可以用拓展运算符转换为真正的数组
     - NodeList实现了Iterator接口,但是自定义的类数组对象没有
  6. Map和Set结构,Generator函数
     - 拓展运算符内部调用的是数据结构的Iterator,因此只要具有Iterator接口的对象,都可以使用拓展运算符,比如说Map结构
     - Generator函数运行后,返回一个遍历器对象,也可以使用拓展运算符

### Array.from()
  - 用于将两类对象转换为数组
    1. 类数组对象
    2. 具有Iterator可遍历的对象
  ```
  // ES5的写法
  var arr1 = [].slice.call(arrayLike); // ['a', 'b', 'c']
  // ES6的写法
  let arr2 = Array.from(arrayLike); // ['a', 'b', 'c']
  ```
  - 如果是一个数组,Array.from()会返回一个一样的数组
  - 类数组对象的本质特征只有一点,具有length属性的对象,任何具有length属性的对象都可以被Array.from()遍历
  ```
  Array.from({ length: 3 });
  // [ undefined, undefined, undefined ]
  ```
  - 对于还没有部署该方法的浏览器,可以用Array.prototype.slice方法替代
  - Array.from还可以接受第二个参数,作用类似于数组的map方法,用来对每个元素进行处理,将处理后的值放入返回的数组
  ```
  Array.from(arrayLike, x => x * x);
  // 等同于
  Array.from(arrayLike).map(x => x * x);
  
  Array.from([1, 2, 3], (x) => x * x)
  // [1, 4, 9]
  ```
  - 使用Array.from()创建任意有内容的数组
  ```
  Array.from({ length: 2 }, () => 'jack')
  // ['jack', 'jack']
  ```
  - Array.from()可以准确地把字符串转换为数组,能正确处理Unicode字符
  
### Array.of() 
  - 用于将一组值转换为数组,用于替代重载不统一的Array构造方法
  - Array()根据参数不同,返回的结果也有所不同
    1. 无参数
       - 返回空数组
    2. 一个参数
       - 返回参数长度的内容为空的数组
    3. 大于两个参数
       - 参数组成的新数组
  - Array.of()固定返回参数组成的数组,如果没有参数就返回空数组
  - Array.of()可以用下面代码模拟
  ```
  function ArrayOf(){
    return [].slice.call(arguments);
  }
  ```
### 数组实例的copyWithin()
  - 在当前数组内部,将指定位置的成员复制到其他位置,会覆盖原有成员,然后返回当前数组
  - 接受三个参数
    1. target 开始替换的位置(必选)
    2. start 从该位置开始读数据
    3. end 从该位置停止读数据

### 数组实例的find()和findIndex()
  - find找出第一个符合条件的成员并返回
  - findIndex找出第一个符合条件的成员并返回下标
  - 与map的参数非常相似,但是第二个参数可以绑定callback执行的时候使用的this
  
### fill
  - 使用给定值.填充一个数组
  - 用于空数组的初始化
  ```
  ['a', 'b', 'c'].fill(7)
  // [7, 7, 7]
  new Array(3).fill(7)
  // [7, 7, 7]
  ```
  - 如果填充的是对象,数组都指向同一个对象,fill只是浅拷贝
  
### entries(),keys()和values()
  - 用于遍历数组,都返回一个遍历器对象,可以用for-of遍历
  - 分别对键值对,键,值进行遍历
  ```
  for (let index of ['a', 'b'].keys()) {
    console.log(index);
  }
  ```

### includes()
  - 返回一个boolean值,数组内是否包含给定值
  - 第二个参数为起始的位置
  - 与字符串的includes相似,为ES2016引入
  - 相比indexOf更加语义化同时可以判定NaN
  
### flat(),flatMap()
  - 数组成员有时候还是数组,flat用于将嵌套的数组拍扁变成一堆的数组
  - flat返回一个新数组,不会修改原数组
  - 接收一个参数表拉平的层数,用Infinity可以设置无论多少层都拉平
  - flat会跳过数组空位
  - flatMap对原数组每个成员执行一个函数,然后对返回的数组执行flat方法,返回一个新的数组,第二个参数可以传入this
  
### 数组的空位
  - 数组的空位指数组某个位置没有任何值
  - 空位不是undefined,空位没有任何值,无法被in,forEach等方法遍历
  - ES5对空位的处理 
    - map会跳过空位,但是会保留这个值
    - join和toString视空位为undefined,undefined和null会被当作字符串处理
  - ES6明确空位转为undefined
    - Array.from方法会将数组的空位,转为undefined
    - 扩展运算符(...)也会将空位转为undefined
    - copyWithin()会连空位一起拷贝
    - fill()会将空位视为正常的数组位置
    - for...of循环也会遍历空位
    
## 对象

### 属性的简介表示法
  - 同名属性和参数可以简写
  ```
  const foo = 'bar';
  const baz = {foo};
  baz // {foo: "bar"}
  // 等同于
  const baz = {foo: foo};
  ```
  - 方法也可以简写
  ```
  const o = {
    method() {
      return "Hello!";
    }
  };
  // 等同于
  const o = {
    method: function() {
      return "Hello!";
    }
  };
  ```
  - getter和setter也是采用这种写法
  ```
  const cart = {
    _wheels: 4,
    get wheels () {
      return this._wheels;
    },
    set wheels (value) {
      if (value < this._wheels) {
        throw new Error('数值太小了！');
      }
      this._wheels = value;
    }
  }
  ```
  - 简介写法的属性名总是字符串,所以不受关键字影响

### 属性名表达式
  - JS定义对象属性有两种方法
    1. obj.a 
    2. obj[a] 
  - ES5如果用字面量定义对象,无法使用方法2
  - ES6允许字面量定义对象时,使用方法2
  ```
  let propKey = 'foo';
  let obj = {
    [propKey]: true,
    ['a' + 'bc']: 123
  };
  ```
  - 属性名如果是对象,会被转成字符串[object Object]
  
### 方法的name属性
  - 函数和方法一样,name属性返回函数名
  - 如果使用了getter和setter,那么无法直接通过函数获取name,name挂载在get和set上,返回值是方法名前加get和set
  ```
  const obj = {
    get foo() {},
    set foo(x) {}
  };
  obj.foo.name
  // TypeError: Cannot read property 'name' of undefined
  
  const descriptor = Object.getOwnPropertyDescriptor(obj, 'foo');
  descriptor.get.name // "get foo"
  descriptor.set.name // "set foo"
  ```
  - bind方法创造的函数,name属性返回bound加上原函数的名字
  - Function构造函数创造的函数,name属性返回anonymous
  - 如果对象的方法是一个Symbol值,那么name属性返回的是这个Symbol值的描述

### 属性的可枚举性和遍历
  - 对象的每个属性都有一个描述对象(属性修饰符)用来控制该属性的行为,可以用Object.getOwnPropertyDescriptor获取
  - 如果属性的enumerable为false,会被以下操作忽略
    1. for-in
    2. Object.keys()
    3. JSON.stringify
    4. Object.assign()
  - ES6规定class的圆形方法都是不可枚举的
  
#### 属性的遍历
  - ES6总共有5种方法可以遍历对象的属性
    1. for-in
       - 循环遍历自身和继承的可枚举属性(不含Symbol)
    2. Object.keys(obj)
       - 自身的所有可枚举属性(不含Symbol)
    3. Object.getOwnPropertyNames(obj)
       - 自身所有属性的键名(不含Symbol)
    4. Object.getOwnPropertySymbols(obj)
       - 自身所有Symbol属性的键名
    5. Reflect.ownKeys(obj)
       - 遍历自身所有属性
  - 这些方法都遵循同样的遍历次序规则
    - 首先遍历所有数值,数值升序排列  
    - 然后遍历所有字符串,按照加入时间升序
    - 最后遍历Symbol,按照加入时间升序
    
### super关键字
  - ES6新增了关键字super指向当前对象的原型对象
  - super表示原型对象时,只能用在对象方法中,用在其他地方都会报错
  - 目前只有对象的简写方法能被JS引擎确认,定义的是对象方法
  - JavaScript引擎内部,super.foo等同于Object.getPrototypeOf(this).foo(属性)或Object.getPrototypeOf(this).foo.call(this)(方法)
  
### 对象的拓展运算符
  - ES2018将拓展运算符引入了对象
  - 对象的解构赋值相当于将自身的所有可遍历、尚未被读取的属性,分配到指定的对象上
  ```
  let { x, y, ...z } = { x: 1, y: 2, a: 3, b: 4 };
  x // 1
  y // 2
  z // { a: 3, b: 4 }
  ```
  - 无法解构undefined和null
  - 解构赋值必须是最后一个参数,不然会报错
  - 解构赋值是浅拷贝
  - 解构赋值不能复制继承自原型对象的属性
  ```
  const o = Object.create({ x: 1, y: 2 });
  o.z = 3;
  console.log(o) // {z:3}
  let { x, ...newObj } = o;
  let { y, z } = newObj;
  console.log(x) // 1
  console.log(y) // undefined
  console.log(z) // 3
  console.log(newObj)
  ```
  - 变量x是单纯的解构赋值,所以可以读取对象o继承的属性
  - 变量y和z是扩展运算符的解构赋值,只能读取对象o自身的属性,所以变量z可以赋值成功,变量y取不到值
  - ES6 规定,变量声明语句之中,如果使用解构赋值,扩展运算符后面必须是一个变量名,而不能是一个解构赋值表达式,所以上面代码引入了中间变量newObj
  - 如果写成下面这样会报错
  ```
  let { x, ...{ y, z } } = o;
  // SyntaxError: ... must be followed by an identifier in declaration contexts
  ```
  - 拓展运算符还能拓展某个函数的参数,接收剩余值
  
### 拓展运算符
  - 可以用拓展运算符,取出参数对象的所有可遍历属性,拷贝到当前对象之中
  ```
  let z = { a: 3, b: 4 };
  let n = { ...z };
  n // { a: 3, b: 4 }
  ```
  - 数组是特殊对象,因此对象的拓展运算符也可以用在数组上
  ```
  let foo = { ...['a', 'b', 'c'] };
  foo
  // {0: "a", 1: "b", 2: "c"}
  ```
  - 对象的拓展运算符等同于使用Object.assign()
  - 如果想要完整克隆一个对象,可以采用以下写法
  ```
  // 写法一 会有浏览器兼容问题
  const clone1 = {
    __proto__: Object.getPrototypeOf(obj),
    ...obj
  }
  // 写法二
  const clone2 = Object.assign(
    Object.create(Object.getPrototypeOf(obj)),
    obj
  )
  // 写法三
  const clone3 = Object.create(
    Object.getPrototypeOf(obj),
    Object.getOwnPropertyDescriptors(obj)
  )
  ```
  - 拓展运算符也可以合并两个对象
  - 拓展运算符后面是{}或null或undefined不会报错但也不会有任何作用
  - 拓展运算符遇到get会执行函数
  
## 对象新增方法
 
### Object.is()
  -  ===不能比较NaN和+-0,Object.is()可以比较两个值是否严格相等

### Object.assign()
  - 用于对象合并,第一个参数是目标对象
  - 如果只有一个参数,则会返回参数,否则会复制一个合并后的目标对象并返回
  - 无法把null和undefined设置为目标参数,但是可以出现在后面的参数中,会被跳过
  - 其他类型放在非第一个参数也不会报错,但是只有字符串会被拆分成数组拷贝入对象
  - 不拷贝继承属性,不拷贝不可枚举属性,但是拷贝Symbol
  1. 浅拷贝
  2. 同名属性的替换
  3. Object.assign能够处理数组,但是会根据下标进行同名替换
  4. 取值函数的处理
     - Object.assign只能进行值的复制,如果复制的是取值函数(getter),会执行取值后复制
  
### Object.assign()主要用途
  1. 为对象添加属性
  ```
  class Point {
    constructor(x, y) {
      Object.assign(this, {x, y});
    }
  }
  ```
  2. 为对象添加方法
  ```
  Object.assign(SomeClass.prototype, {
    someMethod(arg1, arg2) {
      ···
    },
    anotherMethod() {
      ···
    }
  });
  
  // 等同于下面的写法
  SomeClass.prototype.someMethod = function (arg1, arg2) {
    ···
  };
  SomeClass.prototype.anotherMethod = function () {
    ···
  };
  ```
  3. 克隆对象
  ```
  function clone(origin) {
    return Object.assign({}, origin);
  }
  ```
  - 只能克隆值,不能克隆继承来的值
  - 如果要保持继承链,采用以下代码
  ```
  function clone(origin) {
    let originProto = Object.getPrototypeOf(origin);
    return Object.assign(Object.create(originProto), origin);
  }
  ```
  4. 合并多个对象
  5. 为属性指定默认值
  ```
  const DEFAULTS = {
    logLevel: 0,
    outputFormat: 'html'
  };
  
  function processContent(options) {
    options = Object.assign({}, DEFAULTS, options);
    console.log(options);
    // ...
  }
  ```
  - 由于存在浅拷贝的问题,所以指向引用类型很可能会不起作用
  
### Object.getOwnPropertyDescriptors()
  - ES5的Object.getOwnPropertyDescriptor()会返回某个对象属性的描述对象(属性修饰符)
  - ES2017引入ES5的Object.getOwnPropertyDescriptors()返回指定对象的所有属性的描述对象
  ```
  const obj = {
    foo: 123,
    get bar() { return 'abc' }
  };
  
  Object.getOwnPropertyDescriptors(obj)
  // { foo:
  //    { value: 123,
  //      writable: true,
  //      enumerable: true,
  //      configurable: true },
  //   bar:
  //    { get: [Function: get bar],
  //      set: undefined,
  //      enumerable: true,
  //      configurable: true } }
  ```
  - 返回一个对象,所有原对象的属性名都是该对象的属性名,对应的属性值就是该属性的描述对象
  - 该方法的引入主要是为了解决Object.assign无法获取get属性和set属性的问题
  - Object.getOwnPropertyDescriptors()方法配合Object.defineProperties()方法可以实现正确的拷贝
  ```
  const source = {
    set foo(value) {
      console.log(value);
    }
  };
  
  const target2 = {};
  Object.defineProperties(target2, Object.getOwnPropertyDescriptors(source));
  Object.getOwnPropertyDescriptor(target2, 'foo')
  // { get: undefined,
  //   set: [Function: set foo],
  //   enumerable: true,
  //   configurable: true }
  ```
  - Object.getOwnPropertyDescriptors()方法的另一个用处,是配合Object.create()方法将属性克隆到一个新对象(浅拷贝)
  ```
  const clone = Object.create(Object.getPrototypeOf(obj),
    Object.getOwnPropertyDescriptors(obj));
  
  // 或者
  
  const shallowClone = (obj) => Object.create(
    Object.getPrototypeOf(obj),
    Object.getOwnPropertyDescriptors(obj)
  );
  ```
  - 还可以实现一个对象继承另一个对象
  ```
  // 以前
  const obj = {
    __proto__: prot,
    foo: 123,
  };
  
  // ES6规定__proto__只有浏览器要部署,其他环境不用部署
  // 去掉__proto__
  const obj = Object.create(prot);
  obj.foo = 123;
  
  // 或者
  const obj = Object.assign(
    Object.create(prot),
    {
      foo: 123,
    }
  );
  ```
  - Object.getOwnPropertyDescriptors()写法
  ```
  const obj = Object.create(
    prot,
    Object.getOwnPropertyDescriptors({
      foo: 123,
    })
  ```
  - Object.getOwnPropertyDescriptors()也可以用来实现 Mixin(混入)模式
  ```
  let mix = (object) => ({
    with: (...mixins) => mixins.reduce(
      (c, mixin) => Object.create(
        c, Object.getOwnPropertyDescriptors(mixin)
      ), object)
  });
  
  // multiple mixins example
  let a = {a: 'a'};
  let b = {b: 'b'};
  let c = {c: 'c'};
  let d = mix(c).with(a, b);
  
  d.c // "c"
  d.b // "b"
  d.a // "a"
  ```
  - 处于完整性的考虑Object.getOwnPropertyDescriptors()进入标准以后,以后还会新增Reflect.getOwnPropertyDescriptors()方法
  
### __proto__属性,Object.setPrototypeOf(),Object.getPrototypeOf()

### `__proto__`
  - 用于读取当前对象的prototype对象,IE11以上浏览器都支持了这个属性
  ```
  // es5 的写法
  const obj = {
    method: function() { ... }
  };
  obj.__proto__ = someOtherObj;
  
  // es6 的写法
  var obj = Object.create(someOtherObj);
  obj.method = function() { ... };
  ```
  - 该属性没有引入ES6正文,而是写进了附录,原因是因为双下划线说明本质上是一个内部属性,不是一个正式API,因为浏览器广泛支持,才写进了标准
  - 只有浏览器必须部署这个属性,其他运行环境不一定需要部署
  - 使用Object.setPrototypeOf()(写操作)、Object.getPrototypeOf()(读操作)、Object.create()(生成操作)代替直接访问
  - __proto__其实是调用的`Object.prototype.__proto__`,实现如下
  ```
  Object.defineProperty(Object.prototype, '__proto__', {
    get() {
      let _thisObj = Object(this);
      return Object.getPrototypeOf(_thisObj);
    },
    set(proto) {
      if (this === undefined || this === null) {
        throw new TypeError();
      }
      if (!isObject(this)) {
        return undefined;
      }
      if (!isObject(proto)) {
        return undefined;
      }
      let status = Reflect.setPrototypeOf(this, proto);
      if (!status) {
        throw new TypeError();
      }
    },
  });
  
  function isObject(value) {
    return Object(value) === value;
  }
  ```
### Object.setPrototypeOf()
  - 与__proto__相同,用于设置一个对象的prototype对象,是ES6正式推荐的设置原型对象的方法
  ```
  // 格式
  Object.setPrototypeOf(object, prototype)
  
  // 用法
  const o = Object.setPrototypeOf({}, null);
  
  // 等同于
  function setPrototypeOf(obj, proto) {
    obj.__proto__ = proto;
    return obj;
  }
  ```
  - 第一个参数不能为null或者undefined

### Object.getPrototypeOf()
  - 读取一个对象的原型对象
  - 不能获取null或者undefined
  
### Object.keys(),Object.values(),Object.entries()

#### Object.keys()
  - ES5引入了Object.keys()方法
  - ES2017引入了配套的Object.values(),Object.entries()方法

#### Object.values()
  - 只返回对象自身的可遍历属性
  - 会过滤属性名为Symbol的值
  - 如果参数是字符串,会返回字符串数组
  
#### Object.entries()
  - 返回键值对数组
  - 除了返回值不一样,其他跟Object.values()都一样
  - Object.entries方法的另一个用处是,将对象转为真正的Map结构
  - 自己实现
  ```
  // Generator函数的版本
  function* entries(obj) {
    for (let key of Object.keys(obj)) {
      yield [key, obj[key]];
    }
  }
  
  // 非Generator函数的版本
  function entries(obj) {
    let arr = [];
    for (let key of Object.keys(obj)) {
      arr.push([key, obj[key]]);
    }
    return arr;
  }
  ```

### Object.fromEntries()
  - Object.entries()的逆操作,把键值对转换为对象
  - 适合将Map结构转为对象
  - 配合URLSearchParams对象,将查询字符串转为对象
  
## Symbol
  - ES5的对象属性名都是字符串,这容易造成属性名的冲突
  - Symbol值通过Symbol函数生成
  - Symbol函数前不能用new命令,否则会报错,这是因为生成的Symbol是一个原始类型的值,不是对象
  - 接受一个字符串作为参数,作为Symbol实例的描述,方便区分
  - 相同参数的Symbol值不相等
  - Symbol不能与其他值进行运算,会报错
  - 可以显示转换成字符串,或布尔值
  ```
  let sym = Symbol('My symbol');
  
  String(sym) // 'Symbol(My symbol)'
  sym.toString() // 'Symbol(My symbol)'
  Boolean(sym) // true
  !sym  // false
  ```
  
### 作为属性名的Symbol
  - 因为每个Symbol值都是不相等的,因此Symbol可以作为标示符,用于对象的属性名,可以保证不会出现同名属性
  ```
  let mySymbol = Symbol();
  
  // 第一种写法
  let a = {};
  a[mySymbol] = 'Hello!';
  
  // 第二种写法
  let a = {
    [mySymbol]: 'Hello!'
  };
  
  // 第三种写法
  let a = {};
  Object.defineProperty(a, mySymbol, { value: 'Hello!' });
  
  // 以上写法都得到同样结果
  a[mySymbol] // "Hello!"
  ```
  - Symbol类型还可以用于定义一组常量,保证这组常量的值都是不相等的
  - 可以保证switch正确运行
  ```
  const COLOR_RED    = Symbol();
  const COLOR_GREEN  = Symbol();
  
  function getComplement(color) {
    switch (color) {
      case COLOR_RED:
        return COLOR_GREEN;
      case COLOR_GREEN:
        return COLOR_RED;
      default:
        throw new Error('Undefined color');
      }
  }
  ```
  - Symbol作为属性名的时候是公开属性而不是私有属性

### 属性名的遍历
  - Symbol作为属性名不会被for-in、for-of、Object.key、Object.getOwnPropertyNames、JSON.stringify返回
  - 但是Symbol不是私有属性,用Object.getOwnPropertySymbols可以遍历,返回当前对象所有有作用的属性名Symbol
  - Reflect.ownKeys方法可以返回所有类型的键名,包括常规键名和Symbol键名
  - Symbol可以定义一些非私有,但是只希望用于内部的方法
  
### Symbol.for(),Symbol.keyFor()
  - Symbol.for()可以检测让多个变量使用同一个Symbol值,检测是否存在与参数同名Symbol值,有就返回这个值,否则就创建一个参数名的Symbol值
  ```
  let s1 = Symbol.for('foo');
  let s2 = Symbol.for('foo');
  
  s1 === s2 // true
  ```
  - Symbol调用多次返回不同的Symbol,Symbol.for()返回同一个
  - Symbol.keyFor方法返回一个已登记的Symbol类型值的key
  ```
  let s1 = Symbol.for("foo");
  Symbol.keyFor(s1) // "foo"
  
  let s2 = Symbol("foo");
  Symbol.keyFor(s2) // undefined
  ```
  - Symbol.for()登记在全局环境,可以在不同iframe和service worker
  
### 实例:模块的Singleton模式
  - Singleton模式指的是调用一个类,任何时候返回的都是同一个实例
  - Node的模块文件可以看作一个类,怎么保证每次执行模块文件返回的都是同一个实例?
  - 可以把实例放到顶层对象global
  ```
  // mod.js
  function A() {
    this.foo = 'hello'
  }
  if (!global._foo) {
    global._foo = new A()
  }
  module.exports = global._foo
  
  const a = require('./mod.js')
  console.log(a.foo)
  ```
  - 但是任何文件都可以修改globel对象,因此容易使代码失真
  - 使用Symbol实现单例模式
  ```
  // mod.js
  const FOO_KEY = Symbol.for('foo')
  
  function A() {
    this.foo = 'hello'
  }
  
  if (!global[FOO_KEY]) {
    global[FOO_KEY] = new A()
  }
  
  module.exports = global[FOO_KEY];
  
  global[Symbol.for('foo')] = { foo: 'world' }
  const a = require('./mod.js')
  // 保证 global[FOO_KEY] 不会被无意间覆盖,但是还是可以被改写
  ```
  - 键名使用Symbol方法生成,无法被改写,但也导致其他脚本都无法引用FOO_KEY
  ```
  // mod.js
  const FOO_KEY = Symbol('foo')
  ... ...
  ```
  
### 内置的Symbol值
  - ES6提供了11个内置的Symbol值,指向语言内部使用的方法
  
#### Symbol.hasInstance
  - 对象的Symbol.hasInstance属性,指向一个内部方法
  - 当其他对象使用instanceof运算符,判断是否为该对象的实例时,会调用这个方法
  - instaceof在语言内部调用`Foo[Symbol.hasInstance](foo)`
  
#### Symbol.isConcatSpreadable
  - 对象的Symbol.isConcatSpreadable属性等于一个布尔值
  - 表示该对象用于Array.prototype.concat()时,是否可以展开
  ```
  let arr1 = ['c', 'd'];
  ['a', 'b'].concat(arr1, 'e') // ['a', 'b', 'c', 'd', 'e']
  arr1[Symbol.isConcatSpreadable] // undefined
  
  let arr2 = ['c', 'd'];
  arr2[Symbol.isConcatSpreadable] = false;
  ['a', 'b'].concat(arr2, 'e') // ['a', 'b', ['c','d'], 'e']
  ```
  - Symbol.isConcatSpreadable对于数组,等于true或者undefined时可以展开
  - 但是对于类数组对象正好相反,只有属性为true才能展开
  ```
  let obj = {length: 2, 0: 'c', 1: 'd'};
  ['a', 'b'].concat(obj, 'e') // ['a', 'b', obj, 'e']
  
  obj[Symbol.isConcatSpreadable] = true;
  ['a', 'b'].concat(obj, 'e') // ['a', 'b', 'c', 'd', 'e']
  ```
  - 可以定义在类里

#### Symbol.species
  - 指向一个构造函数,创建衍生对象时,会使用该属性
  ```
  class MyArray extends Array {
  }
  
  const a = new MyArray(1, 2, 3);
  const b = a.map(x => x);
  const c = a.filter(x => x > 1);
  
  b instanceof MyArray // true
  c instanceof MyArray // true
  ```
  - 子类MyArray继承了父类Array,a是MyArray的实例,b和c是a的衍生对象
  - b和c虽然是数组方法产生的,但也是MyArray的实例
  - 可以为MyArray设置Symbol.species属性,用于指定被new时的构造函数
  ```
  class MyArray extends Array {
    static get [Symbol.species]() { return Array; }
  }
  
  const a = new MyArray();
  const b = a.map(x => x);
  
  b instanceof MyArray // false
  b instanceof Array // true
  ```

#### Symbol.match
  - 对象的Symbol.match属性,指向一个函数
  - 当执行str.match(myObject)时,如果该属性存在,会调用它,返回该方法的返回值
  ```
  String.prototype.match(regexp)
  // 等同于
  regexp[Symbol.match](this)
  
  class MyMatcher {
    [Symbol.match](string) {
      return 'hello world'.indexOf(string);
    }
  }
  
  'e'.match(new MyMatcher()) // 1
  ```
  
#### Symbol.replace
  - 对象的Symbol.replace属性,指向一个方法
  - 当该对象被String.prototype.replace方法调用时,会返回该方法的返回值
  ```
  String.prototype.replace(searchValue, replaceValue)
  // 等同于
  searchValue[Symbol.replace](this, replaceValue)
  ```
  - Symbol.replace方法会收到两个参数
    1. replace方法正在作用的对象
    2. 替换后的值
  ```
  const x = {};
  x[Symbol.replace] = (...s) => console.log(s);
  
  'Hello'.replace(x, 'World') // ["Hello", "World"]
  ```

#### Symbol.search
   - 对象的Symbol.search属性,指向一个方法
   - 当该对象被String.prototype.search方法调用时,会返回该方法的返回值
   ```
   String.prototype.search(regexp)
   // 等同于
   regexp[Symbol.search](this)
   
   class MySearch {
     constructor(value) {
       this.value = value;
     }
     [Symbol.search](string) {
       return string.indexOf(this.value);
     }
   }
   'foobar'.search(new MySearch('foo')) // 0
   ```
   
#### Symbol.split
  - 对象的Symbol.split属性,指向一个方法
  - 当该对象被String.prototype.split方法调用时,会返回该方法的返回值
  ```
  String.prototype.split(separator, limit)
  // 等同于
  separator[Symbol.split](this, limit)
  ```
  
#### Symbol.iterator
  - 对象的Symbol.iterator指向对象默认遍历器方法
  - 对对象的for-of循环会调用Symbol.iterator方法,返回对象默认的遍历器
  
#### Symbol.toPrimitive
  - Symbol.toPrimitive指向一个方法,当对象被转化成原始类型时调用这个方法
  - 被调用时会接受一个字符串参数表示当前运算模式
    1. Number 转化成数值
    2. String 转化成字符串
    3. Default 既可以转化为数值也可以转化为字符串
  ```
  let obj = {
    [Symbol.toPrimitive](hint) {
      switch (hint) {
        case 'number':
          return 123;
        case 'string':
          return 'str';
        case 'default':
          return 'default';
        default:
          throw new Error();
       }
     }
  };
  
  2 * obj // 246
  3 + obj // '3default'
  obj == 'default' // true
  String(obj) // 'str'
  ```

#### Symbol.toStringTag
  - 指向一个方法,在该对象上调用toString方法时,如果这个属性存在,它的返回值会出现在toString方法返回的字符串中,表示对象类型
  - 这个属性可以用于定制`[object object]`后面的字符串
  - ES6中新增的toStringTag属性值
    - `JSON[Symbol.toStringTag]`:'JSON'
    - `Math[Symbol.toStringTag]`:'Math'
    - `Module 对象M[Symbol.toStringTag]`:'Module'
    - `ArrayBuffer.prototype[Symbol.toStringTag]`:'ArrayBuffer'
    - `DataView.prototype[Symbol.toStringTag]`:'DataView'
    - `Map.prototype[Symbol.toStringTag]`:'Map'
    - `Promise.prototype[Symbol.toStringTag]`:'Promise'
    - `Set.prototype[Symbol.toStringTag]`:'Set'
    - `%TypedArray%.prototype[Symbol.toStringTag]`:'Uint8Array'等
    - `WeakMap.prototype[Symbol.toStringTag]`:'WeakMap'
    - `WeakSet.prototype[Symbol.toStringTag]`:'WeakSet'
    - `%MapIteratorPrototype%[Symbol.toStringTag]`:'Map Iterator'
    - `%SetIteratorPrototype%[Symbol.toStringTag]`:'Set Iterator'
    - `%StringIteratorPrototype%[Symbol.toStringTag]`:'String Iterator'
    - `Symbol.prototype[Symbol.toStringTag]`:'Symbol'
    - `Generator.prototype[Symbol.toStringTag]`:'Generator'
    - `GeneratorFunction.prototype[Symbol.toStringTag]`:'GeneratorFunction'

#### Symbol.unscopables
  - Symbol.unscopables属性,指向一个对象
  - 该对象指定了使用with关键字时,哪些属性会被with环境排除
  - 可以通过指定Symbol.unscopables属性,让with语法块不会在当前作用域寻找foo属性
  ```
  // 没有 unscopables 时
  class MyClass {
    foo() { return 1; }
  }
  
  var foo = function () { return 2; };
  
  with (MyClass.prototype) {
    foo(); // 1
  }
  
  // 有 unscopables 时
  class MyClass {
    foo() { return 1; }
    get [Symbol.unscopables]() {
      return { foo: true };
    }
  }
  
  var foo = function () { return 2; };
  
  with (MyClass.prototype) {
    foo(); // 2
  }
  ```

## Set和Map数据结构

### Set
  - ES6提供了新的数据结构Set,类似于数组,但是成员值都是唯一的,没有重复值
  - Set本身是个构造函数,用来生成Set数据结构
  ```
  const s = new Set();
  [2, 3, 5, 4, 5, 2, 2].forEach(x => s.add(x));
  for (let i of s) {
    console.log(i);
  }
  // 2 3 5 4
  ```
  - Set可以用add方法加入成员
  - Set可以接受一个数组或者具有iterable接口的其他数据结构
  - 可以用Set去除数组重复成员
  ```
  // 去除数组的重复成员
  [...new Set(array)]
  ```
  - Set接受参数去重的规则是严格相等,包括NaN,但是两个对象总是不相等的
  
### Set实例的属性和方法
  - Set.prototype.constructor 构造函数.默认就是Set函数
  - Set.prototype.size:返回Set实例的成员总数
  - Set实例的方法分为两大类
    1. 操作方法(用于操作数据)
    2. 遍历方法(用于遍历成员)
  - 四个操作方法
    1. add(value):添加某个值.返回 Set 结构本身
    2. delete(value):删除某个值.返回一个布尔值.表示删除是否成功
    3. has(value):返回一个布尔值.表示该值是否为Set的成员
    4. clear():清除所有成员.没有返回值
  - Array.from方法可以将Set结构转为数组
  ```
  // 另一种数组去重操作
  Array.from(new Set(array))
  ```
  - 四个遍历方法
    1. keys():返回键名的遍历器
    2. values():返回键值的遍历器
    3. entries():返回键值对的遍历器
    4. forEach():使用回调函数遍历每个成员
    - Set键名就是键值,forEach第二个参数是this
  - Set可以非常方便地求交集并集和差集
  ```
  let a = new Set([1, 2, 3]);
  let b = new Set([4, 3, 2]);
  
  // 并集
  let union = new Set([...a, ...b]);
  // Set {1, 2, 3, 4}
  
  // 交集
  let intersect = new Set([...a].filter(x => b.has(x)));
  // set {2, 3}
  
  // 差集
  let difference = new Set([...a].filter(x => !b.has(x)));
  ```
  - 无法在遍历的时候直接修改原Set结构
  ```
  // 方法一
  let set = new Set([1, 2, 3]);
  set = new Set([...set].map(val => val * 2));
  // set的值是2, 4, 6
  
  // 方法二
  let set = new Set([1, 2, 3]);
  set = new Set(Array.from(set, val => val * 2));
  // set的值是2, 4, 6
  ```

### WeakSet
  - WeakSet结构与Set类似,也是不重复的值的集合
  - WeakSet成员只能是对象
  - WeakSet中的对象都是弱引用,垃圾回收机制不会考虑WeakSet对于对象的引用,对象资源被回收之后WeakSet内部的引用会自动消失
  - WeakSet不可遍历
  - WeakSet是一个构造函数,可以使用new命令,创建WeakSet数据结构
  - 参数是数组时,是数组成员变成WeakSet成员而不是数组本身
  ```
  const a = [[1, 2], [3, 4]];
  const ws = new WeakSet(a);
  // WeakSet {[1, 2], [3, 4]}
  
  const b = [3, 4];
  const ws = new WeakSet(b); // b成员不是对象
  // Uncaught TypeError: Invalid value used in weak set(…)
  ```
  - WeakSet结构有以下三个方法。
    1. WeakSet.prototype.add(value):向WeakSet实例添加一个新成员
    2. WeakSet.prototype.delete(value):清除 WeakSet 实例的指定成员
    3. WeakSet.prototype.has(value):返回一个布尔值,表示某个值是否在WeakSet实例之中
  - WeakSet没有size属性,没有办法遍历其成员
  
### Map
  - JS对象本质上是键值对集合,但是传统上的对象只能用字符串当键
  - 如果键不是字符串,会被自动转化为字符串,会变成不希望变成的样子
  - ES6提供了Map数据结构,类似与对象,也是键值对的集合,但是键的范围不限于字符串,所有类型都能成为键
  ```
  const m = new Map();
  const o = {p: 'Hello World'};
  
  m.set(o, 'content')
  m.get(o) // "content"
  
  m.has(o) // true
  m.delete(o) // true
  m.has(o) // false
  ```
  - 只要具有Iterator接口,且每一个成员都是双元素的数组结构,都可以作为Map构造函数的参数
  - Set和Map都能用来生成新的Map
  - 同一个键的多次赋值,后面覆盖前面
  - 获取一个未知的键返回undefined
  - 值相同的实例(引用类型),在Map中被视为两个键

#### Map实例与操作方法
  - 操作方法
  1. size:返回Map结构的成员总数
  2. set(key, value)
  3. get(key)
  4. has(key)
  5. delete(key)
  6. clear()
  
  - 遍历方法
  1. keys():返回键名的遍历器
  2. values():返回键值的遍历器
  3. entries():返回所有成员的遍历器
  4. forEach():遍历Map的所有成员
  - 遍历顺序就是插入顺序
  - map实例可以用...转换为数组
  
#### Map与其他数据结构的互相转换 
  1. Map转数组
     - 使用拓展运算符(...)
  2. 数组转Map
     - 只能传入特定结构的数组进构造函数
  3. Map转对象
     - 所有Map键都是字符串,可以无损地转对象(非字符串会被转为字符串)
  4. 对象转Map
     - Map的set方法
  5. Map转JSON
     - 如果键名都是字符串,这个时候可以选择转为对象JSON
     ```
     // '{"yes":true,"no":false}'
     ```
     - 如果键名有非字符串,可以转为数组JSON
     ```
     // '[[true,7],[{"foo":3},["abc"]]]'
     ```
  6. JSON转Map
     - 正常情况所有键名都是字符串
     - JSON数组可以一一转为Map
  - 涉及对象和Map转换都需要手动遍历
  
#### WeaKMap
  - WeakMap结构与Map结构类似,但是WeakMap只接受对象作为键名(null除外)
  - 不能接受Symbol值为键名
  - WeakMap的键名所指向的对象,不计入垃圾回收机制
  - 一个典型的使用场景就是给DOM元素添加数据,当DOM元素被清除,WeakMap保存的记录也会被清除
  - 只有键名是弱引用
  - WeakMap与Map主要区别在两个方面
    1. 没有遍历操作与size属性
    2. 无法清空,无法使用clear方法
    
## Proxy
  - 用于修改某些操作的默认行为,等同于在语言层面进行修改,因此属于一种(元编程:对编程语言本身进行编程)
  - 相当于在目标对象之前假设一层拦截,外界访问对象前都必须先通过这层拦截
  - 因此提供了一种机制,可以对外界的访问进行过滤和改写
  - Proxy这个词的原意是代理,用在这里表示由它来"代理"某些操作,可以译为"代理器"
  - Proxy实际上重载(overload)了点运算符,即用自己的定义覆盖了语言的原始定义
  
### Proxy语法
  - ES6原生提供Proxy构造函数,用来生成Proxy实例
  - new Proxy()表示生成一个Proxy实例,target参数表示所要拦截的目标对象,handler参数也是一个对象,用来定制拦截行为
  ```
  var proxy = new Proxy(target, handler)
  ```
  - 要使得Proxy起作用,必须针对Proxy实例(上例是proxy对象)进行操作,而不是针对目标对象(上例是空对象)进行操作
  - 如果Handler没有进行任何拦截,那就等同于直接通向原对象
  - 可以将Proxy实例设置到对象上或者对象的原型对象上,从而在对象中调用
  ```
  var proxy = new Proxy({}, {
    get: function(target, property) {
      return 35
    }
  })
  let obj = Object.create(proxy)
  obj.time // 
  ```
  - 同一个拦截器函数,可以设置拦截多个操作
  ```
  let handler = {
      get: function (target, name) {
          if (name === 'prototype') {
              return Object.prototype
          }
          return 'Hello' + name
      },
      apply: function (target, thisBinding, args) {
          return args[0]
      },
      construct: function (target, args) {
          return {value: args[1]}
      }
  }
  let fproxy = new Proxy(function (x, y) {
      return x + y
  }, handler)
  console.log(fproxy(1, 2)) // 1new fproxy(1, 2) // {value: 2}
  console.log(new fproxy(1, 2)) // {value: 2}
  console.log(fproxy.prototype === Object.prototype)// true
  console.log(fproxy.foo === 'Hello, foo')// true
  ```
#### Proxy支持的13种拦截方式
  1. get(target,propKey,receiver)
  2. set(target,propKey,value,receiver)
  3. has(target,propKey) 
     - 拦截`propKey in proxy`操作
  4. deleteProperty(target, propKey)
     - 拦截`delete proxy[propKey]`
  5. ownKeys(target)
     - 拦截`Object.getOwnPropertyNames(proxy)、Object.getOwnPropertySymbols(proxy)、Object.keys(proxy)、for...in`
     - 返回一个数组
  6. getOwnPropertyDescriptor(target, propKey)
     - 拦截`Object.getOwnPropertyDescriptor(proxy, propKey)`
     - 返回属性的描述对象
  7. defineProperty(target, propKey, propDesc)
     - 拦截`Object.defineProperty(proxy, propKey, propDesc）、Object.defineProperties(proxy, propDescs)`
     - 返回一个布尔值
  8. preventExtensions(target)
     - 拦截`Object.preventExtensions(proxy)`
     - 返回一个布尔值
  9. getPrototypeOf(target)
     - 拦截`Object.getPrototypeOf(proxy)`
     - 返回一个对象
  10. isExtensible(target)
     - 拦截`Object.isExtensible(proxy)`
     - 返回一个布尔值
  11. setPrototypeOf(target, proto)
     - 拦截`Object.setPrototypeOf(proxy, proto)`
     - 返回一个布尔值,如果目标对象是函数,还有两种操作可以
  12. apply
     - 拦截Proxy实例作为函数调用的操作,比如`proxy(...args)、proxy.call(object, ...args)、proxy.apply(...)`
  13. construct(target, args)
     - 拦截Proxy实例的构造函数调用的操作,比如new proxy(...args)

### Proxy 实例的方法

#### get
  - 用于拦截某个属性的读取操作,接收三个参数
    1. 目标对象
    2. 属性名
    3. proxy实例本身(操作行为针对的对象,可选)
  ```
  var person = {
    name: "张三"
  }
  
  var proxy = new Proxy(person, {
    get: function(target, property) {
      if (property in target) {
        return target[property]
      } else { // 如果访问对象不存在的属性,抛出一个错误
        throw new ReferenceError("Property \"" + property + "\" does not exist.")
      }
    }
  })
  
  proxy.name // "张三"
  proxy.age // 抛出一个错误
  ```
  - get方法可以继承
  - 利用get可以把读取属性的操作转变为执行某个函数,从而实现属性的链式操作
  ```
  let pipe = (function () {
      return function (value) {
          let funcStack = []
          let oproxy = new Proxy({}, {
              get: function (pipeObject, fnName) {
                  if (fnName === 'get') {
                      return funcStack.reduce(function (val, fn) {
                          return fn(val)
                      }, value)
                  }
                  funcStack.push(window[fnName])
                  return oproxy
              }
          }
          return oproxy
      }
  }())
  // 必须要用var才能赋值到window对象上
  var double = n => n * 2
  var pow = n => n * n
  var reverseInt = n => n.toString().split('').reverse().join('')
  console.log(pipe(3).double.pow.reverseInt.get)// 63
  ```
  - 可以用get拦截,实现一个生成各种DOM节点的通用函数dom
  ```
  const dom = new Proxy({}, {
      get(target, property) {
          return function (attrs = {}, ...children) {
              console.log('开始执行', 'attrs', attrs, 'children', children)
              const el = document.createElement(property)
              for (let prop of Object.keys(attrs)) {
                  el.setAttribute(prop, attrs[prop])
              }
              for (let child of children) {
                  if (typeof child === 'string') {
                      child = document.createTextNode(child)
                  }
                  el.appendChild(child)
              }
              return el
          }
      }
  })
  const el = dom.div({}, 'Hello, my name is ', dom.a({href: '//example.com'}, 'Mark'), '. I like:',
      dom.ul({},
          dom.li({}, 'The web'),
          dom.li({}, 'Food'),
          dom.li({}, '…actually that\'s it')
      )
  )

  document.body.appendChild(el)
  ```
  - 如果一个属性不可配置(configurable)且不可写(writable),proxy就不能修改该属性

#### set
  - 用于拦截某个属性的赋值操作,可以接受四个参数
    1. 目标对象
    2. 属性名
    3. 属性值
    4. proxy实例本身
  ```
  let validator = {
    set: function(obj, prop, value) {
      if (prop === 'age') {
        if (!Number.isInteger(value)) {
          throw new TypeError('The age is not an integer')
        }
        if (value > 200) {
          throw new RangeError('The age seems invalid')
        }
      }
  
      // 对于满足条件的 age 属性以及其他属性，直接保存
      obj[prop] = value
    }
  }
  
  let person = new Proxy({}, validator)
  
  person.age = 100
  
  person.age // 100
  person.age = 'young' // 报错
  person.age = 300 // 报错
  ```
  - 设置了存值函数set,任何不符合要求的age属性赋值,都会抛出一个错误,这是数据验证的一种实现方法
  - 利用set方法,还可以数据绑定,即每当对象发生变化时,会自动更新 DOM
  - 可以通过get和set方法控制数据的访问,比如说禁止访问下划线开头的私有变量
  ```
  const handler = {
    get (target, key) {
      invariant(key, 'get');
      return target[key];
    },
    set (target, key, value) {
      invariant(key, 'set');
      target[key] = value;
      return true;
    }
  };
  function invariant (key, action) {
    if (key[0] === '_') {
      throw new Error(`Invalid attempt to ${action} private "${key}" property`);
    }
  }
  const target = {};
  const proxy = new Proxy(target, handler);
  proxy._prop
  // Error: Invalid attempt to get private "_prop" property
  proxy._prop = 'c'
  // Error: Invalid attempt to set private "_prop" property
  ```
  - 第四个参数receiver指的是原始行为所操纵的对象,一般情况是proxy对象本身(原型链上除外)
  - 如果目标对象自身某个属性,不可写不可配置,set方法就不起作用
  - 严格模式下set方法不返回true就报错
#### apply
  - apply方法拦截函数的调用,call,apply操作,接收三个参数
    1. 目标对象
    2. 目标对象的上下文(this)
    3. 目标对象的参数数组
  - 函数调用会被拦截
  ```
  var target = function () { return 'I am the target'; };
  var handler = {
    apply: function () {
      return 'I am the proxy';
    }
  };
  
  var p = new Proxy(target, handler);
  
  p()
  // "I am the proxy"
  ```
  - 无论是直接调用还是call,apply调用都会被拦截
  ```
  var twice = {
    apply (target, ctx, args) {
      return Reflect.apply(...arguments) * 2;
    }
  };
  function sum (left, right) {
    return left + right;
  };
  var proxy = new Proxy(sum, twice);
  proxy(1, 2) // 6
  proxy.call(null, 5, 6) // 22
  proxy.apply(null, [7, 8]) // 30
  ```
  - 直接调用Reflect.apply方法,也会被拦截

#### has
  - has用于拦截HasProperty操作,即判断对象是否具有某个属性时,会被拦截,接收两个参数
    1. 目标对象
    2. 需要查询的属性名
  - 可以使用has隐藏某些属性,不被in运算符发现
  ```
  var handler = {
    has (target, key) {
      if (key[0] === '_') {
        return false;
      }
      return key in target;
    }
  };
  var target = { _prop: 'foo', prop: 'foo' }
  var proxy = new Proxy(target, handler)
  console.log('_prop' in proxy) // false
  ```
  - 如果对象被设置了不可配置或者禁止拓展(Object.preventExtensions),使用has拦截会报错
  ```
  var obj = { a: 10 };
  Object.preventExtensions(obj)
  
  var p = new Proxy(obj, {
    has: function(target, prop) {
      return false;
    }
  })
  'a' in p // TypeError is thrown
  ```
  - 如果对象属性被配置了不可拓展,has方法就不得隐藏该属性
  - has方法拦截HasProperty操作,而不是HasOwnProperty操作,即has方法不判断一个属性是对象自身的属性,还是继承的属性
  - has拦截对for-in不生效

#### construct
  - 用于拦截new命令,接收两个参数
    1. target:目标对象
    2. args:构造函数参数对象
  - 必须返回一个对象否则会报错

#### deleteProperty
  - 用于拦截delete操作,如果这个方法抛出错误或者返回false,当前属性就无法被删除,接收两个参数
    1. 目标对象
    2. 删除的属性名
  - 如果对象自身具有不可配置的属性,不能被deleteProperty方法删除,否则报错
  
#### defineProperty
  - defineProperty拦截了Object.defineProperty操作
  - 如果默认返回false,那么定义的新属性总是无效
  ```
  var handler = {
    defineProperty (target, key, descriptor) {
      return false;
    }
  };
  var target = {};
  var proxy = new Proxy(target, handler);
  proxy.foo = 'bar' // 不会生效
  ```
  - 如果对象是不可拓展的,则defineProperty不能添加对象上不存在的属性,否则报错
  - 如果目标对象的某个属性不可写或者不可配置,defineProperty方法不得改变这两个设置
  
#### getOwnPropertyDescriptor
  - 该方法拦截Object.getOwnPropertyDescriptor(),返回一个属性描述对象或者undefined
  ```
  var handler = {
    getOwnPropertyDescriptor (target, key) {
      if (key[0] === '_') {
        return;
      }
      return Object.getOwnPropertyDescriptor(target, key);
    }
  };
  var target = { _foo: 'bar', baz: 'tar' };
  var proxy = new Proxy(target, handler);
  Object.getOwnPropertyDescriptor(proxy, 'wat')
  // undefined
  Object.getOwnPropertyDescriptor(proxy, '_foo')
  // undefined
  Object.getOwnPropertyDescriptor(proxy, 'baz')
  // { value: 'tar', writable: true, enumerable: true, configurable: true }
  ```
#### getPrototypeOf
  - getPrototypeOf方法主要用来拦截获取对象原型,拦截下面这些操作
    - `Object.prototype.__proto__`
    - `Object.prototype.isPrototypeOf()`
    - `Object.getPrototypeOf()`
    - `Reflect.getPrototypeOf()`
    - `instanceof`
  - 返回值必须是对象或者null,否则报错
  - 如果目标对象被设置了不可拓展,必须返回目标对象的原型对象
  ```
  var proto = {};
  var p = new Proxy({}, {
    getPrototypeOf(target) {
      return proto;
    }
  });
  Object.getPrototypeOf(p) === proto // true
  ```
  
#### isExtensible
  - 拦截Object.isExtensible操作
  ```
  var p = new Proxy({}, {
    isExtensible: function(target) {
      console.log("called");
      return true;
    }
  });
  
  Object.isExtensible(p)
  // "called"
  // true
  ```
  - 只能返回布尔值,否则会被强转为布尔值
  - 返回值必须与目标对象的isExtensible属性保持一致,否则会抛出错误
  
#### ownKeys
  - 用于拦截自身对象属性的读取操作
    - Object.getOwnPropertyNames()
    - Object.getOwnPropertySymbols()
    - Object.keys()
    - for...in
  - 使用Object.keys()时,会忽略以下属性
    1. 不可遍历属性
    2. Symbol
    3. 对象上不存在的属性(对象上没有proxy拦截的返回值)
  - ownKeys方法返回的数组成员只能是字符串或者Symbol,其他类型或者返回的不是数组,会报错
  - 如果目标对象自身包含不可配置的属性,则该属性必须被ownKeys方法返回,否则报错
  - 如果目标对象是不可拓展的,返回的数组必须包含原对象的所有属性,不能包含多余属性,否则报错
    