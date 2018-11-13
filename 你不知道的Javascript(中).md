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