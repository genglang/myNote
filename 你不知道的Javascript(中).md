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