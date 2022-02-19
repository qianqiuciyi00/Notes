# MessageChannel
MessageChannel允许创建一个新的消息通道，并通过它的两个`MessagePort`属性发送数据。属于**宏任务**。
用法：  
```js
var channel = new MessageChannel()

// 实例属性，只读。两个端口都可以通过postMessage发送数据，而一个端口只要绑定了onmessage回调方法，就可以接收从另一端口传过来的数据
var port1 = channel.port1
var port2 = channel.port2
port1.onmesssge = function(event) {
  console.log('port1接收到来自port2的数据：' + event.data)
}
port2.onmesssge = function(event) {
  console.log('port2接收到来自port1的数据：' + event.data)
}
port1.postMessage('发送给port2')
port2.postMessage('发送给port1')
```
- 用于深拷贝  
  `postMessage`传递的数据也是深拷贝的，也可以深拷贝`undefined`和循环引用的对象，但拷贝有函数时，会报错
  ```js
  // 有undefined + 循环引用
    let obj = {
      a: 1,
      b: {
        c: 2,
        d: 3,
      },
      f: undefined
    }
    obj.c = obj.b;
    obj.e = obj.a
    obj.b.c = obj.c
    obj.b.d = obj.b
    obj.b.e = obj.b.c

    function deepCopy(obj) {
      return new Promise((resolve) => {
        const {port1, port2} = new MessageChannel();
        port2.onmessage = ev => resolve(ev.data);
        port1.postMessage(obj);
      });
    }

    deepCopy(obj).then((copy) => {           // 请记住`MessageChannel`是异步的这个前提！
        let copyObj = copy;
        console.log(copyObj, obj)
        console.log(copyObj == obj)
    });
  ```
- 在web Worker中使用  
  当我们使用多个`web worker`并想要在两个`web worker`之间实现通信的时候，`MessageChannel`也可以派上用场
  ```js
  let worker1 = new Worker('./worker1.js');
let worker2 = new Worker('./worker2.js');
let ms = new MessageChannel();

// 把 port1 分配给 worker1
worker1.postMessage('main', [ms.port1]);
// 把 port2 分配给 worker2
worker2.postMessage('main', [ms.port2]);

//worker1.js
self.onmessage = function(e) {
    console.log('worker1', e.ports);
    if (e.data === 'main') {
        const port = e.ports[0];
        port.postMessage(`worker1: Hi! I'm worker1`);
        port.onmessage = function(ev){
            console.log('reveice: ',ev.data,ev.origin);
        }
    }       
}

//worker2.js
self.onmessage = function(e) {
    if (e.data === 'main') {
        const port = e.ports[0];
        port.onmessage = function(e) {
            console.log('receive: ', e.data);
            port.postMessage('worker2: ' + e.data);
        }
    }
}
  ```


# JSON.stringify
- JSON.stringify的一些特性 
对于undefined、任意函数以及symbol三个特殊的值分别作为对象属性的值、数组元素、单独的值时，JSON.stringify()将返回不同的结果  
  - undefined、任意函数以及symbol作为对象属性值时，JSON.stringify将跳过对他们进行序列化
  ```js
  const data = {
    a: 'a',
    b: undefined,
    c: Symbol('c'),
    fn: function() {
      return 'fn'
    }
  }
  JSON.stringify(data)  // {a: a}
  ```
  - 作为数组元素时，会将他们序列化为null
  ```js
  JSON.stringify([
    'a',
    undefined,
    function fn() {
      return 'fn'
    },
    Symbol('c')
  ])
  // "["a", null, null, null]"
  ```
  - 作为单独的值进行序列化时都会返回undefined
  ```js
  JSON.stringify(function a() {
    console.log('a')
  })
  // undefined
  JSON.stringify(undefined)  // undefined
  JSON.stringify(Symbol('c'))  // undefined
  ```
  非数组对象的属性不能保证以特定的顺序出现在序列化后的字符串中
  ```js
  const data = {
    a: 'a',
    b: undefined,
    c: Symbol("c"),
    fn:function (){
        return "fn";
    },
    d: "d",
};
JSON.stringify(data);
// "{"a": "a", "d":"d"}"
 
JSON.stringify([
    "a",
    undefined,
    function fn(){
        return "fn"
    },
    Symbol("c"),
    (d: "d"),
]);
```
z转换值时如果有to.JSON()函数，该函数返回什么值，序列化结果就是什么值，并且忽略其他属性的值
```js
JSON.stringify({
    str: "str",
    toJSON: function () {
        return "strToJson";
    },
});
// "strToJson"
```
JSON.stringify()将会正常序列化Date的值
```js
JSON.stringify({now: new Date()});
```
实际上Date对象自己部署了toJSON()方法，因此Date对象会被当做字符串处理  

NaN和infinity格式的数值及null都会被当作null
```js
JSON.stringify(NaN);
// "null"
JSON.stringify(null);
// "null"
JSON.stringify(Infinity);
// "null"
```
b布尔值、数字、字符串的包装对象在序列化过程中会自动转化成对应的原始值
```js
JSON.stringify([new Number(1), new String("false"), new Boolean(false)]);
// "[1,"false",false]"
```
其它类型的对象，包括Map/Set/WeakMap/WeakSet，仅会序列化可枚举的属性
```js
JSON.stringify(
  Object.create(null, {
    x: { value: 'json', enumerable: false },
    y: { value: 'stringify', enumerable: true },
  })
)
// "{"y": "stringify"}"
```
实现深拷贝最简单粗暴的方法就是序列化：JSON.parse(JSON.stringify(obj))，这个方式实现深拷贝会因为序列化的诸多特性从而导致诸多的坑点：比如循环引用的问题  
BigInt的值会抛出TypeError（不能序列化）  

- JSON.stringify第二个参数replacer  
第二个参数可以是一个函数或者一个数组。作为函数时，它有两个参数，key和value，函数类似就是数组的方法吗，map、filter等方法的回调函数，对每一个属性值都会执行一次该函数。如果是一个数组，数组的值代表就会被序列化成JSON字符串的属性名

# DOMContentLoaded
- 当初始的HTML文档被完全加载和解析完成之后，DOMContentLoaded事件被触发，而无需等待样式表、图像和子框架的完全加载。如果文档中包含脚本，则脚本会阻塞文档的解析，而脚本需要等待CSSDOM构建完成才能执行。在任何情况下，DOMContentLoaded的触发不需要等待图片等其他资源加载完成。
- 使用
  ```js
    document.addEventListener('DOMContentLoaded', function(event) {
        ...
    })
  ```

# DOMContentLoaded和Load
DOM 文档的加载步骤：
1、解析HTML结构
2、加载外部脚本和样式表文件
3、解析并执行脚本代码 // 部分脚本会阻塞页面的加载
4、DOM树构建完成 // DOMContentLoaded事件触发
5、加载图片等外部文件
6、页面加载完毕  // Load事件触发

# 异步脚本
- async 或defer
  当HTML文档被解析时如果遇见同步脚本，则停止解析，先去加载脚本，然后执行，执行结束后继续解析HTML文档。
- defer脚本：
  当HTML文档被解析时如果遇见defer脚本，则在后台加载脚本，文档解析过程不中断，而等文档解析结束之后，defer脚本执行。另外，defer脚本的执行顺序与定义时的位置有关系。
- async脚本：
  当HTML文档被解析时如果遇见async脚本，则在后台加载脚本，文档解析过程不中断。脚本加载完成后，文档停止解析，脚本执行，执行结束之后文档继续解析。

# 异步脚本与DOMContentLoaded
- defer：DOMContentLoaded 只有在defer脚本执行结束之后才会被触发
- async: HTML文档构建不受影响，解析完毕后，DOMContentLoaded触发，而不需要等待adync脚本执行、样式表加载等等。

# MutationObserver
- MutationObserver是一个内建对象，它观察DOM元素，并在检测到更改时触发回调

# Source Map
JavaScript大部分源码都需要经过转换才能投入生产环境。常见的源码转换场景：  
1、压缩，减小体积  
2、多个文件合并，减少HTTP请求数  
3、其他语言编译成JavaScript。  
source map 是一个信息文件，里面储存着位置信息，也就是转换后代码的每一个位置，和所对应转换前的位置。这样，在生产环境报错时将能直接看到原始代码，方便调试

# 事件循环
1、为什么JavaScript是单线程  
与他的用途有关，作为浏览器脚本语言，JavaScript的主要用途是与用户互动，以及操作DOM。假如JavaScript同时有两个线程，一个线程在某个DOM节点上添加内容，另一个线程删除了这个节点，这时浏览器该以哪个线程为准呢？所以，为了避免复杂性，JavaScript就是单线程，以后也不会改变。  
2、任务队列  
单线程意味着，所有任务都需要排队，前一个任务结束，才会执行后一个任务。在等待任务执行完成的时候，有的CPU是闲着的，可以先挂起处于等待中的任务，先运行排在后面的任务，等到前一个任务有了结果，再回过头，把挂起的任务继续执行下去。
于是，所有任务可以分成两种，一种是同步任务，另一种是异步任务。同步任务指的是，在主线程上排队执行的任务，只有前一个任务执行完毕，才能执行后一个任务；异步任务指的是，不进入主线程，而进入“任务队列”的任务，只有“任务队列”通知主线程，某个异步任务可以执行了，该任务才会进入主线程执行  
异步执行的运行机制如下（同步执行也是如此，因此它可以被视为没有异步任务的异步执行）
```
1、所有同步任务都在主线程上执行，形成一个执行栈
2、主线程之外，还存在一个“任务队列”，只要异步任务有了运行结果，就在“任务队列”之中放置一个事件
3、一旦“执行栈”中的所有同步任务执行完毕，系统就会读取“任务队列”，看看里面有哪些事件。那些对应的异步任务，于是结束等待状态，进入执行栈，开始执行
4、主线程不断重复上面的第三步
```
任务队列中，有两种任务：宏任务和微任务  
宏任务：script标签中的整体代码、setTimeout、setInterval、setImmediate、I/O、UI渲染  
微任务：process.nextTick(Node.js)、Promise、Object.observe、MutationObserve(Node.js)  
任务优先级：process.nextTick > promise.then > setTimeout > setTimmediate  
3、Event Loop
主线程从“任务队列”中读取事件，这个过程是不断循环的，所以整个的这个运行机制又称为Event Loop（事件循环）

# 栈和堆的概念和区别，垃圾回收时的区别
- 概念  
  栈：是栈内存的简称，栈是自动分配相对固定大小的内存空间，并由系统自动释放，栈数据结构遵循先进后出的原则  
  堆：是堆内存的简称，堆是动态分配内存，内存大小不固定，也不会自动释放，堆数据结构是一种无序的树状结构，同时它还满足key-value键值对的存储方式  
  栈的特点：速度快，容量小  
  堆的特点：速度稍慢，容量比较大  
- 基本类型和引用类型
![内存 内存](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/7/23/17379552d18f1b91~tplv-t2oaga2asx-watermark.awebp)  
**基本数据类型**直接存放在栈内存中，占用的内存空间的大小是确定的，并由系统自动分配和自动释放。这样的好处就是，内存可以及时得到回收，相对于堆来说，更加容易管理内存空间  
**引用数据类型**：指那些可能由多个值构成的对象；如Object、Array、Function，他们是通过拷贝和new出来的，这样的数据存储在堆中  

- 传值和传址的区别  
**基本类型**：采用的是值传递  
**引用类型**：采用地址传递  
引用类型的数据的地址指针是存储在栈中的，将存放在栈内存中的地址赋值给接收的变量。当我们想要访问引用类型的值的时候，需要先从栈中获得对象的地址指针，然后再通过地址指针找到堆中的所需要的数据（保存在堆内存中，包含引用类型的变量实际上保存的不是变量本身，而是指向该对象的指针）
- 内存分配垃圾回收  
1.内存分配  
  - 栈内存：线性有序存储，容量小，系统分配效率高
  - 堆内存：首先要在顿内存新分配存储区域，之后又要把指针存储到栈内存中，效率相对要低些  
2.垃圾回收
  - 栈内存：变量基本上用完就回收了，相对于堆呢来说存取速度会快，并且栈内存中的数据是可以共享的
  - 堆内存：堆内存中的对象不会随方法的结束而销毁，就算方法结束了，这个对象也可能会被其他引用变量所引用（参数传递）。创建对象是为了反复利用（因为对象的创建成本通常较大），这个对象将被保存到运行时数据区（也就是堆内存）。只有当一个对象没有任何引用变量引用它时，系统的垃圾回收机制才会在核实的时候回收它。

# 内存泄漏
- 不再使用的内存，没有及时释放，就叫做内存泄漏。
- 垃圾回收机制  
  最常用的方法叫做“引用计数”，语言引擎有一张“引用表”，保存了内存里面所有的资源的引用次数。如果一个值的引用次数是0，则表示这个值不再用到了，那么就可以将这块内存释放。  
  如果一个值不再需要了，引用数却不为0，垃圾回收机制无法释放这块内存，从而导致内存泄漏
- WeakMap  
  ES6推出了两种新的数据结构：WeakSet 和 WeakMap。它们对于值的引用都是不计入垃圾回收机制的，Weak表示弱引用

# Object.is()
判断两个值是否为同一个值，返回Boolean类型
```
Object.is(val1, val2)
```
如果满足以下条件则两个值相等：  
- 都是undefined
- 都是null
- 都是true或false
- 都是相同长度的字符串且相同字符按相同顺序排列
- 都是相同对象（意味着每个对象都有同一个引用）
- 都是数字且  
  - 都是+0
  - 都是-0
  - 都是NaN
  - 都是非零而且非NaN且为同一个值

与 == 的区别：==运算符在判断相等前对两边的变量（如果他们不是同一类型）进行强制转换，而Object.is()不会强制转换两边的值  
与 === 的区别：=== 运算符将数字 -0 和 +0 视为相等，而将 Number.NaN 与 NaN 视为不相等

# for in 、for of 和 Object.keys()的区别
- for in ：遍历对象所有可枚举属性，包括原型链上的属性。（数组，对象）
- for of： 主要便利可迭代的对象（Array, Map, Set, argument等）用来获取对象的属性值，返回属性值
- Object.keys()：遍历对象所有可枚举属性，不包括原型链上的属性，返回一个数组（数组，对象）
- Object.getPropertyNames()：返回除原型属性以外的所有属性（包括不可枚举属性）名组成的数组（数组，对象）

# 内部属性[[class]]
所有的typeof 返回值为 object 的对象（如数组）都包含一个内部属性`[[class]]`（我们可以把它看作一个内部的分类，而非传统的面向对象意义上的类）。这个属性无法直接访问，一般通过`Object.prototype.toString(...)`来查看。例如：
```js
Object.prototype.toString.call([1,2,3])
// "[object Array]"

Object.prototype.toString.call( /regex-literal/i )
// "[object RegExp]"

// 我们自己创建的类就不会有这份特殊待遇，因为toString()找不到toStringTag属性时只好返回默认的Object标签
// 默认情况类的[[class]]返回[object Object]
class Class1 {}
Object.prototype.toString.call(new Class1())  // "[object Object]"
// 需要定制[[class]]
class Class2 {
  get [Symbol.toStringTag]() {
    return "class2"
  }
}
Object.prototype.toString.call(new Class2())  // "[object Class2]"
```

# 面向对象
- 三大特征：
  - 封装：  
    所谓封装，就是把客观事物封装成抽象的类，并且类可以把自己的数据和方法只让可信的类或对象操作，对不可信的进行信息隐藏。封装是面向对象的特征之一，是对象和类概念的主要特征。简单地说，一个类就是一个封装了数据以及操作这些数据的代码的逻辑实体。在一个对象内部，某些代码或某些数据可以是私有的，不能被外界访问。通过这种方式，对象对内部数据提供了不同级别的防护，以防止程序中无关的部分意外的改变或错误使用了对象的私有部分。
  - 继承：  
    继承是指这样一种能力：它可以使用现有类的所有功能，并在无需重写原来的类的情况下对这些功能进行扩展。通过继承创建的新类称为“子类”或“派生类”，被继承的类称为“基类”、“父类”或“超类”。继承的过程，就是从一般到特殊的过程。要实现继承，可以通过“继承”和“组合”来实现。继承概念的实现方式有两类：实现继承和接口继承。实现继承是指直接使用基类的属性和方法而无需额外编码的能力；接口继承是指仅使用属性和方法的名称，但是子类必须提供实现的能力。
  - 多态：  
    所谓多态是指一个类实例的相同方法在不同情形有不同表现形式。多态机制使具有不同内部结构的对象可以共享相同的外部接口。意味着，虽然针对不同对象的具体操作不同，但通过一个公共的类，它们（那些操作）可以通过相同的方式予以调用。  
    最常见的多态就是将子类传入父类参数中，运行时调用父类方法时通过传入的子类决定具体的内部结构或行为。
- 五大原则：
  - 单一职责原则：**一个类应该仅有一个引起它变化的原因**
  - 开放封闭原则：**对扩展是开放的，对更改是封闭的**
  - 里氏替换原则：**子类可以替换父类并且出现在父类能够出现的任何地方，贯彻GOF倡导的面向接口编程**
  - 依赖倒置原则：**传统的结构化编程中，最上层的模块通常都要依赖下面的子模块来实现，也成为最高层依赖低层**，所以依赖倒置原则就是要逆转这种依赖，让高层模块不要依赖低层模块
  - ISP接口隔离原则：**使用多个专门的接口比使用单个接口要好得多**

# Symbol.toPrimitive
Symbol.toPrimitive 是一个内置的 Symbol 值，它是作为对象的函数值属性存在的，当一个对象转换为对应的原始值时，会调用此函数。  
该函数被调用时，会被传递一个字符串参数 hint ，表示要转换到的原始值的预期类型。 hint 参数的取值是 "number"、"string" 和 "default" 中的任意一个。  
此⽅法在转基本类型时调⽤优先级最⾼
```js
let a = {
valueOf() {
return 0;
 },
toString() {
return '1';
 },
 [Symbol.toPrimitive]() {
return 2;
 }
}1 + a // => 3
'1' + a // => '12'
```