# 你不知道的JavaScript笔记(上)
## 一、作用域和闭包
### 传统语言编译步骤
  1. 分词/词法分析 
  2. 解析/语法分析 
  3. 代码生成
### JS保证性能的方法
  - JIT
  - 延迟编译
  - 重编译
### 一些概念
  - 引擎:负责JS编译执行过程
  
  - 编译器:语法分析代码生成
  
  - 作用域:收集维护所有声明的标识符（变量）组成的一系列查询 并实施一套严格的规则 确定当前执行的代码对这些标识符的访问权限
  
  - var赋值流程:分解成词法单元 词法单位生成一个树结构 判断作用域下是否有相同名称变量 新建变量 生成代码
  
### 引擎查询
  
  - LHS:向左查询,试图找到变量容器本身从而进行赋值 赋值操作的目标是谁
  
  - RHS:向右查询,找到变量,非左侧（retrieve his source value）赋值操作的源头
  
    严格模式禁止LHS找不到的时候创建变量

## 二、词法作用域
### 作用域气泡
   - 作用域气泡的结构和相互之间的位置关系给引擎提供了足够的位置信息
   - 引擎用这些信息来查找标识符的位置

### 遮蔽效应
   - 遮蔽效应:作用于差找会在查找到第一个匹配的标识符的时候结束,所以不同作用域可以定义同名变量
   - 函数作用域只和声明时的位置有关
   - 词法作用域只会查找一级标识符,找到变量后对象属性访问规则会分别接管对象内属性的访问
## 欺骗词法
### eval
   - 欺骗作用域,包含声明时,会修改所在位置的作用域
   - 严格模式中eval在运行时有自己的词法作用域
   - JS定时器第一个参数可以是字符串(过时不推荐使用)
   - new Function()也可以传递字符串语句
   - 接受或含有声明代码,就会修改其词法作用域
### with
   - with通常被当作重复引用同一个对象中的多个属性的快捷方式,可以不需要重复引用对象本身,没有完全隔离的语法作用域,因此容易造成作用域泄漏
   ```
      function foo(obj){
        with(obj){
          a = 2
        }
      }
      var o={
        b: 3
      }
      foo(o)
      console.log(o.a) // undefined
      console.log(a) // 2 a泄露到全局作用域
   ```
   - with声明凭空创建了一个词法作用域
     
     JS引擎会在编译阶段进行数项性能优化,其中某些优化依赖于能根据代码的词法进行静态分析,并预先确定函数和变量的定义位置,才能快速找到标识符,因此扰乱作用域会使这些优化白做功
## 三、函数作用域和块作用域
   - 函数作用域是指属于这个函数的全部变量都可以在整个函数的范围内使用及复用（事实上在嵌套的作用域也可以使用）,能充分利用JS变量可以根据需要改变值类型的动态特性
### 函数作用域的好处
   1. 隐藏内部实现
   2. 规避冲突
### 函数作用域的问题
   1. 自身函数命名污染作用域
   2. 解决
   3. 自执行函数
   
   - 如果function是声明第一个词是函数声明,否则就是函数表达式
   
### 匿名函数缺点
   - 在栈追踪中不会显示有意义用户名,调试困难
   - 如果没有函数名,需要引用自身时只能用arguments.callee引用
   - 省略了函数名降低了可读性
 ###自执行函数
   - IIFE(Immediately Invoked Function Expression)
   - 因为函数本身被包裹在()中,所以是一个表达式,第二个()立即执行这个函数
   
#### 垃圾回收
   - let不会变量提升
   - let可以绑定变量到当前作用域
   - let可以避免闭包产生的阻拦垃圾回收的问题
   - for循环头的let不仅将i绑定到for循环块中,事实上它将其重新绑定在循环的每一个迭代中,确保使用上一个迭代结束时的值进行重新赋值
## 三、提升
   - 引擎会在解释JS代码之前首先进行编译,编译的一部分工作就是找到所有的声明,并用合适的作用域关联起来
   - 只有声明本身会被提前,赋值和其他代码不变,每个作用域都会提升操作。
   - 函数和变量都会提升,并且函数会被首先提升,然后才是变量,并且应该避免块作用域内声明函数
   - 函数声明会被提前,函数表达式不会
## 四、作用域闭包
### 闭包的概念
   - 当函数可以记住并访问所在词法作用域时,就产生了闭包,即使函数在当前词法作用域外执行
   - 通过闭包访问引用返回的函数,函数所在的作用域不会被引擎销毁。
   - IE有bug无法回收闭包里引用的变量
   
### 模块模式的必要特点
   - 必须要有外部的封闭函数,该函数至少要被调用一次(每次调用都会创建一个新的模块)
   - 封闭函数必须要返回至少一个内部函数,这样内部函数才能在私有作用域中形成闭包,并可以访问或者修改内部状态
   - 一个具有函数属性的对象本身并不是真正的模块
   - 从方便观察角度看,一个函数调用所返回的,只有数据属性没有闭包函数的对象并不是真正的模块
   - 模块化的特征：为创建内部作用域而了一个包装函数、包装函数的返回值必须至少包括一个对函数内部的引用
     
  
### ES6模块机制
   - 函数并不是一个可以被稳定实别的模式
   - ES6模块在编译期就对导入模块进行了检测判断是否真实存在,然后抛出一个早期错误
    
   - import可以将一个模块中的一个或多个API倒入当前作用域中,并分别绑定在一个变量上
   - module会将整个模块的API倒入并绑定在一个变量上
   - export会将当前模块的一个标识符导出为公共API
   
### 隐式作用域 
   - let声明会创建一个显式的作用域并于其进行绑定
   
   ```
    // 伪代码
    let (a = 2) {
      console.log(a) // 2
    }
    console.log(a) // ReferenceError
  ```
### 性能
   - try-catch来实现块级作用域性能很糟糕
   - 浏览器因此对其进行了优化
   - try-catch与自调函数并不能完全等价

## 六、this
  - this提供了一种更优雅的方式来隐式传递一个对象引用,因此可以将API设计得更加简洁并易于复用
  - arguments.callee可以引用当前运行的函数对象(已弃用)
  - this是运行时绑定的,上下文取决于函数调用的条件
  - this声明和函数定义没有任何关系,只取决于函数调用方式,传入参数等信息,
  - this就是记录其中一个属性,会在函数执行中调用
  - 严格模式不会进行默认绑定
### this的四种绑定方式
  1. 默认绑定:直接使用不带修饰的函数调用
  2. 隐式绑定:调用时有上下文对象或者被某个对象拥有或包含
     - 隐式丢失:在this调用的时候因为上下文变化导致的this绑定丢失
  3. 显式绑定：call apply bind 第一个参数如果传入了原始值,那么会被转换成对象形式（new String（））
     - 许多内置函数提供了一个可选参数可以绑定this
     - 软绑定:call apply
     - 硬绑定:bind
      ```
          // 手写bind
          function bind(fn,obj){
               return function(){
                 return fn.apply(obj,arguments)
               }
          }
      ```   
      API调用的上下文:api通过传入参数实现回调函数指定this
  4. new绑定:JS特殊的构造函数只是一些使用new操作符时调用的函数,不属于某个类,也不会实例化一个类,实际上甚至不能说是一种特殊的函数类型,只是被new调用的普通函数。被称为构造函数调用（不存在构造函数 只有构造调用）
 ### new绑定的流程
  1. 创建(或者构造)一个全新的对象
  2. 这个对象会被进行原型链接
  3. 这个新对象会绑定到函数调用this
  4. 如果函数没有返回其他对象,那么new表达式中的函数调用会自动返回这个新对象
  
  ### this绑定优先级
    new > 硬绑定(显式绑定) > 隐式绑定 > 默认绑定
  ### Polyfill
  - 处理旧浏览器的兼容问题,比如bind
  - 但是因为不是内置函数无法创建不含.prototype的函数,会有一些副作用
   ```
      if (!Function.prototype.bind) {
      		Function.prototype.bind = function (oThis) {
      			if (typeof this !== 'function') {
      				throw new TypeError('bind尝试绑定this但是this类型不符')
      			}
      			var aArgs = Array.prototype.slice.call(arguments, 1),
      				fToBind = this,
      				fNOP = function(){},
      				fBound = function() {
      					return fToBind.apply(
      						(
      							this instanceof fNOP &&
      							oThis ? this : oThis
      						),
      						aArgs.concat(Array.prototype.slice.call(arguments))
      					)
      				}
      			fNOP.prototype=this.prototype
      			fBound.prototype = new fNOP()
      			return fBound
      		}
      	}
   ```
  - 这段代码会判断函数是否被new调用,如果是就会用新创建的this替代已有的this
  - 使用new的原因:主要目的是预先设置函数的一些参数,这样使用new进行初始化就可以只传入剩余参数
  - bind的功能之一就是把第一个参数this外的其他参数都传给下层函数（柯里化的一种）
  
### 判断this
  可以通过优先级来判断函数在某个调用位置应用的是哪条规则,按照下面的顺序进行判断
  
  1. 函数是否在new中调用(new绑定)?如果是的话this绑定的是新创建的对象
  2. 函数是否通过apply/call(显式绑定)或者硬绑定调用?如果是的话,this绑定的是指定的对象
  3. 函数是否在某个上下文对象中调用(隐式绑定)?如果是的话,this绑定的是那个上下文对象
  4. 如果都不是的话,使用默认绑定,如果在严格模式下,就绑定undefined,否则绑定全局对象

### 例外
  - 如果把null或者undefined传入call,会被忽略,并把this默认绑定(用apply展开数组用...替代)
### 更安全的this
  - Object.create(null)创建一个空对象(不会创建prototype)然后作为this传入call/apply/bind
### 间接引用
  - 在某些情况下,间接引用一个函数,触发的默认绑定可能会影响结果
  ```
     function foo(){
       console.log(this.a)
     }
     var a = 2
     var o = { a: 3, foo: foo }
     var p = { a: 4 }
     o.foo() // 3
     (p.foo = o.foo)() // 2
  ```
### 软绑定
  ```
      if(!Function.prototype.softBind){
        Function.prototype.softBind = function( obj ) {
          var fn = this
          // 捕获所有 curried参数
          var curried = [].slice.call(arguments, 1)
          var bound = function() {
            return fn.apply(
              (!this||this === (windows || global)) ? obj : this ,
              curried.concat.apply( curried, argument)
            )
          }
          bound.prototype = Object.create( fn.prototype )
          return bound
        }
      }
  ```
  - 对指定函数进行封装,首先检查调用时的this
  - 如果this绑定全局变量或者undefined,就把指定的默认对象obj绑定到this
  - 否则不会修改this
  - 此外,这段代码还支持柯里化
### this词法
  - 箭头函数不适用this的四种标准规则,而是根据外层(函数或者全局)作用域决定
  
## 七.对象
### 对象定义的两种方式
  - 字面量
  - 构造函数
  唯一的区别:字面量你可以添加多个键值对,构造函数需要逐个添加属性
### 对象的类型
  - JS万物皆对象是错误的
  - null有时候会被当作对象,这是JS的bug,即对null执行typeof会返回object
  - 不同的对象底层都会被表示为二进制,JS二进制前三位为0会被判为object类型,null的二进制全都是0
  
### 复杂基本类型
  - JS有许多特殊的对象子类型,我们称之为复杂基本类型
  1. 函数是对象的子类型,是JS中的一等公民,本质上和普通对象一样,可以像一个对象一样操作
  2. 数组也是对象的一种类型,具备一些额外的行为,数组比一般的对象要稍微复杂
  
### 内置对象
  1. String
  2. Nubmer
  3. Boolean
  4. Object
  5. Function
  6. Array
  7. Data
  8. RegExp
  9. Error
  - 可以当作构造函数来使用,从而可以构造一个对应子类型的新对象
  ```
    let str1 = 'Hello'
    console.log(typeof str1) // 'string'
    console.log(str1 instanceof String) // false
   
    let str2 = new String('hello')
    console.log(typeof str2) // 'object'
    console.log(str2 instanceof String) // true
  ```
  - 字符串调用字符串对象方法时引擎会自动把字符串字面量转换为String对象(尽量不要用构造方法)
  - null,undefined没有构造形式
  - Date只有构造形式
  - Object,Array,Function,RegExp来说,无论字面量还是构造函数都是对象
  - Error很少在代码中显式创建,一般是抛出异常时自动创建

###  内容
  - 对象的内容是一些存储在特定命名位置(任意类型)值组成的,我们称之为属性
  - 语言上内容暗指这些值被存储在对象内部,但这只是表现形式,实际在引擎内部,存储方式多种多样
  - 一般不会存在对象内部,而是通过名称引用指针指向真正的存储位置

### .和[]访问对象值的区别
  - 属性访问和键访问,访问同一个位置,主要区别在于
  - .操作符要求属性名满足标识符命名规范
  - []则接受所有UTF8/Unicode字符串作为属性名,比如说Super-Fun!只能通过[]访问(赋值时无论属性名是啥都会转成字符串)
  - 可以在声明时通过[a+b计算属性名]
  - 给数组通过[]添加属性不会改变length值
  - 数组会自动把key的数字字符串转化为数字

### 复制对象
  - 旧方法
  ```
    var newObject = JSON.parse(JSON.stringify(someObj))
  ```
  - 新方法
  ```
    let obj1 = Object.assign({},obj2)
  ```
 
### 属性描述符
  ES5后可以直接通过getOwnPropertyDescriptor检测对象属性特征
  通过Object.defineProperty(obj,prop,descriptor)定义属性描述符
  - writable 可修改
    - 如果修改,非严格模式下会出现静默失败,严格模式会报错
  - enumerable 可枚举
    - 是否出现在属性枚举中(for...in)
  - configurable 可配置
    - 不管是否是严格模式,只要修改一个不可配置的属性都会报错
    - 不可配置属性不可删除

### 不变性
  - 所有的方法创建的都是浅不变形,只会影响目标对象和直接属性,对引用对象属性无效,需要对引用再次进行不可变操作
  - JS很少需要深不可变性,建议重新思考程序设计,慎用

### 不可变性操作
  1. 对象常量
    - 结合writable和configurable就可以创建一个真正的常量属性(不可修改、重定义和删除)
  2. 禁止拓展
    - 通过Object.praventExtensions(obj)禁止添加属性
  3. 密封
    - Object.seal()会创建一个密封对象
    - 相当于调用Object.praventExtensions(),并把configurable设置为false
  4. 冻结
    - Object.freeze()创建一个冻结对象
    - 相当于对现有对象调用Object.seal()并把writable设置为false(引用对象不受影响)
    
### [[Get]]
  - 访问对象属性的时候其实是执行了[[Get]]操作
  - 对象默认内置[[Get]]操作首先在对象中差找是否有同名属性,找到就返回这个值
  - 如果没有找到就去原型链上查找
  - 如果都没有就返回undefined
  
### [[Put]]
  - 执行的时候取决于许多因素,包括对象中是否已经有这个属性了
    1. 属性是否是访问描述符?如果是并且存在setter就调用setter
    2. 属性的数据描述符writable是否为false,是就根据是否是严格模式失败
    3. 如果都不是,将该值设置为属性的值

### Getter和Setter
  - ES5中可以使用getter和setter部分改写默认操作,但是只能应用在单个属性上
  - 只能应用在单个属性上,无法应用在整个对象上
  - 分别在获取和设置的时候调用
  - 二者都会在对象中创建一个不包含值的属性,这个属性会自动调用一个隐藏函数,返回值会被当成当前属性的返回值
  - 当getter和setter两者都有时,这个属性会被定义为"访问描述符"
  - JS会忽略访问描述符的value和writable特性,取而代之的是get和set(和configurable和enumerable)特性
  
### 存在性
  判断对象是否有某属性
  1. (prop in obj)
  2. obj.hasOwnProperty('prop')
  3. obj.propertyIsEnumerable('prop'')
  
  区别
  - in 会检测prototype原型链
  - hasOwnProperty 只检测当前对象(排除原型链)
  - propertyIsEnumerable 遍历满足hasOwnProperty的可枚举元素

### 枚举
  - enumerable设置为false之后,属性就不会出现在for-in循环中
  - 在数组上应用for..in循环有时会产生出人意料的结果,因为这种枚举不仅会包含所有数值索引,还会包含所有可枚举属性
  - 最好只在对象上应用for..in 循环,如果要遍历数组就使用传统的for循环来遍历数值索引
  
### 判断属性是否可以被枚举的方法
  1. obj.propertyIsEnumerable
      - 会检查给定的属性名是否直接存在于对象中(而不是在原型链上)并且满足enumerable:true
  2. Object.keys
      - 会返回一个数组，包含所有可枚举属性
  3. Object.getOwnPropertyNames
      - 会返回一个数组,包含所有属性,无论它们是否可枚举

### 循环
  - 普通for循环并不是遍历值,而是遍历下标来指向值
  - ES5中增加了一些数组的辅助迭代器,每种辅助迭代器都能接受一个回调函数并把它应用到数组的每个元素上,唯一区别就是它们对于回调函数的返回值处理方式不同
      1. forEach 忽略返回值
      2. some 一直运行到true
      3. every 一直运行到false
### for-of
  - ES6新增了的遍历数组的语法(如果对象本身定义了迭代器也可以遍历对象)
  1. for-of循环首先会向被访问的对象请求一个迭代器对象
  2. 然后通过迭代器对象的next()方法来遍历所有返回值
  
  1. 数组for-of
      - 数组有内置的@@iterator,因此for-of能直接应用在数组上
  
  2. 对象for-of
      - 使用ES6中的符号Symbol.iterator来获取对象的@@iterator
      - 引用类似iterator的特殊属性时要使用符号名,而不是符号包含的值
      - 此外，虽然看起来很像一个对象,但是@@iterator本身并不是一个迭代器对象,而是一个返回迭代器对象的函数——这点非常精妙并且非常重要
  - 可以通过Object.defineProperty给对象定义迭代器属性
  - for-of循环每次调用myObject迭代器的next()方法时,内部的指针都会向前移动并返回对象列表的下一个值(提醒:需要注意遍历对象属性/值的顺序)


