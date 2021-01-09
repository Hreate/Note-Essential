# JavaScript

## 1 ES5 中的严格模式

### 1.1 严格模式的理解

#### 1.1.1 概念

**理解：**除了正常运行模式（混杂模式），ES5 添加了第二种运行模式——“严格模式”（strict mode）。

顾名思义，这种模式使得 JavaScript 在更严格的语法条件下运行。

**目的：**

* 消除 JavaScript 语法的一些不合理、不严谨之处，减少一些怪异行为。
* 消除代码运行 的一些不安全之处，为代码的安全运行报价护航。
* 为未来新版本的 JavaScript 做好铺垫。



#### 1.1.2 使用

* 针对整个脚本文件：将 `use strict` 放在脚本文件的第一行，则整个脚本文件将以严格模式运行。

* 针对单个函数：将 `use strict` 放在函数体的第一行，则整个函数以严格模式运行。

PS：如果浏览器不支持，则这句话只解析为一条简单的语句，没有任何副作用。

脚本文件的变通写法：因为第一种调用方法不利于文件合并，所以更好的做法是，借用第二种方法，将整个脚本文件放在一个立即执行的匿名函数之中。



#### 1.1.3 语法和行为改变

* 必须用 var 声明变量。
* 禁止自定义的函数中的 this 指向 window。
* 创建 eval 作用域。
* 对象不能有重名的属性。



### 1.2 严格模式和普通模式的区别

> 下面列举几条严格模式的内容

#### 1.2.1 全局变量显式声明

在正常模式中，如果一个变量没有声明就赋值，默认是全局变量。严格模式禁止这种用法，全局变量必须显式声明。



#### 1.2.2 禁止 this 关键字指向全局对象

```javascript
var foo = function () {
    console.log(this);
}
foo();
```

上方代码中，普通模式打印的是 window。严格模式下打印的是 undefined。



#### 1.2.3 禁止使用 with 语句

因为 with 语句无法在编译时就确定，属性到底归属哪个对象。



#### 1.2.4 构造函数必须通过 new 实例化对象

构造函数必须通过 new 实例化对象，否则报错。因为 this 为 undefined，此时无法设置属性。

比如说：

```javascript
var Cat = function (name) {
    this.name = name;
}
Cat('haha');
```

上方代码中，如果在严格模式中，则会报错。



#### 1.2.5 属性相关

普通模式下，如果对象有多个重名属性，最后赋值的那个属性会覆盖前面的值。严格模式下，这属于语法错误。

普通模式下，如果函数有多个重名的参数，可以用 arguments[i] 读取。严格模式下，多个重名的参数属于语法错误。

比如下面这样的代码：

```javascript
var obj = {
    username: 'smyh',
    username: 'vae'
}
```

上面的代码，在严格模式下属于语法错误，因为有重名的属性。



#### 1.2.6 函数必须声明在顶层

奖励啊 JavaScript 的新版本会引入“块级作用域”。为了与新版本接轨，严格模式只允许在全局作用域或函数作用域的顶层声明函数。也就是说，不允许在非函数的代码块内声明函数。



#### 1.2.7 新增关键字

为了向将来 JavaScript 的新版本过度，严格模式新增了一些保留字：implements、interface、let、package、private、protected、public、static、void。



## 2 ES5 中的一些扩展

### 2.1 JSON 对象

* js 对象（数组）-> json 对象（数组）：

  ```javascript
  JSON.stringify(obj/srr)
  ```

* json 对象（数组）-> js 对象（数组）：

  ```javascript
  JSON.parse(json)
  ```

上面这两个方法是 ES5 中提供的。

通常说的“json 字符串”只有两种：json 对象、json 数组。

`typeof json字符串` 的返回结果是 string。



### 2.2 Object 的扩展

ES5 给 Object 扩展了一些静态方法，常用的有2个

#### 2.2.1 方法一

```javascript
Object.create(prototype, [descriptors])
```

作用：以指定对象为原型，创建新的对象。同时，第二个参数可以为新的对象添加新的属性，并对此属性进行描述。

**举例1：**（没有第二个参数时）

```javascript
var obj1 = {username: 'smyhvae', age: 26};
var obj2 = {address: 'shenzhen'};

obj2 = Object.create(obj1);
console.log(obj2);
```

obj1 成为了 obj2 的原型

**举例2：**（有第二个参数时）

第二个参数可以给新的对象添加新的属性：

```javascript
var obj1 = {username: 'smyhvae', age: 26};
var obj2 = {address: 'shenzhen'};

obj2 = Object.create(obj1, {
    sex: { // 给 obj2 添加新的属性 sex
        value: '男', // 通过 value 关键字设置 sex 的属性值
        writable: false,
        configurable: true,
        enumerable: true
    }
});

console.log(obj2);
```

上方代码，通过第5行给 obj2 设置了一个新的属性 sex，但是要通过 value 来设置属性值（第6行）。设置玩属性值后，这个属性值默认是不可修改的，要通过 writable 来设置。总而言之，这几个关键字的解释如下：

* value：设置属性值。
* writable：标识当前属性值是否可修改。如果不指定，默认为 false，不可修改。
* configurable：标识当前属性是否可以被删除。默认为 false，不可删除。
* enumerable：标识当前属性是否能用 for in 枚举。默认为 false，不可。



单独设置属性

```javascript
Object.defineProperty(obj2, 'sex', {
    value: 'cc',
    writable: true,
    configurable: true,
    enumerable: true
});
```



### 2.2.2 方法二

```javascript
Object.defineProperties(object, descriptors)
```

**作用：**为指定对象定义扩展多个属性。

代码举例：

```javascript
var obj2 = {
    firstName: 'smyh',
    lastName: 'vae'
};
Object.defineProperties(obj2, {
    fullName: {
        get: function () {
            return this.firstName + '·' + this.lastName;
        },
        set: function (data) { // 监听扩展属性，当扩展属性发生变化的时候自动调用，自动调用后将变化的值作为实参注入到 set 函数
            var names = data.split('·');
            this.firstName = name[0];
            this.lastName = name[1];
        }
    }
});
```

* get：用来获取当前属性值的回调函数
* set：修改当前属性值触发的回调函数，并且实参即为修改后的值

存取器属性：setter、getter 一个用来存值，一个用来取值



### 2.3 Object 的扩展（二）

obj 对象本身就自带了两个方法。格式如下：

> get 属性名() {} 用来得到当前属性值的回调函数
>
> set 属性名() {} 用来监听当前属性值变化的回调函数

举例如下：

```javascript
var obj = {
    firstName: 'kobe',
    lastName: 'bryant',
    get fullName() {
        return this.firstName + ' ' + this.lastName;
    },
    set fullName(data) {
        var names = data.split(' ');
        this.firstName = names[0];
        this.lastName = names[1];
    }
};
console.log(obj.fullName);
obj.fullName = "curry stephen";
console.log(obj.fullName);
```



### 2.4 数组的扩展

**方法1：**

```javascript
Array.prototype.indexOf(value)
```

作用：获取 value 在数组中的第一个下标。

**方法2：**

```javascript
Array.prototype.lastIndexOf(value)
```

作用：获取 value 在数组中的最后一个下标。

**方法3：**遍历数组

```javascript
Array.prototype.forEach(function(item, index) {})
```

**方法4：**

```javascript
Array.prototype.map(function(item, index) {})
```

作用：遍历数组返回一个新的数组，返回的是加工之后的新数组。

**方法5：**

```javascript
Array.prototype.filter(function(item, index) {})
```

作用：遍历过滤出一个新的数组，返回条件为 true 的值。



### 2.5 函数 functiong 的扩展：bind()

> ES5 中新增了 bind() 函数来改变 this 的指向

```javascript
Function.prototype.bind(obj)
```

作用：将函数内的 this 绑定为 obj，并将函数返回。

**面试题：**call()、apply() 和 bind() 的区别：

* 都能改变 this 的指向。
* call()/apply() 是立即调用函数。
* bind()：绑定完 this 后，不会立即调用当前函数，而是将函数返回，因此后面还需要再加 () 才能调用。

PS：bind() 传参的方式和 call() 一样

**分析：**

为什么 ES5 中要加入 bind() 方法来改变 this 的指向呢？

因为 bind() 不会立即调用当前函数。bind() 通常在回调函数中使用，因为回调函数并不会立即调用。如果希望在回调函数中改变 this，不妨使用 bind()。



## 3 ES6 的变量声明

ES6 中新增了 let 和 const 来定义变量：

* var：ES5 和 ES6 中，定义**全局变量**（是 variable 的简写）。
* let：定义**局部变量**，替代 var。
* const：定义**常量**（定义后不可修改）。



### 3.1 var：全局变量

看下面的代码：

```javascript
{
    var a = 1;
}
console.log(a); // 这里的 a，指的是代码块中的 a
```

上方代码是可以输出结果的，输出结果为1。因为 var 是全局声明的，所以，即使实在代码块中声明，仍然在全局起作用。

再来看下面这段代码：

```javascript
var a = 1;
{
    var a = 2;
}
console.log(a); // 这里的 a，指的是代码块中的 a
```

上方代码的输出结果为2，因为 var 是全局声明的。

**总结：**

用 var 定义的全局变量，有时候会污染整个 js 的作用域。



### 3.2 let：定义局部变量

```javascript
var a = 2;
{
    let a = 3;
}
console.log(a);
```

上方代码的输出结果为2.用 let 声明的变量，只在局部（块级作用域内）起作用。

let 是防止数据污染，来看下面这个 for 循环的例子，很经典。

1. 用 var 声明变量：

   ```javascript
   for (var i = 0; i < 10; i++) {
      console.log('循环体中：' + i); // 每循环一次，就会在 {} 所在的块级作用域中，重新定义一个新的 i 
   }
   console.log('循环体外' + i)；
   ```

   上方代码可以正常打印结果，且最后一行的打印结果是10。说明循环体外定义的变量 i，是在全局起作用的。

2. 用 let 声明变量：

   ```javascript
   for (let i = 0; i < 10; i++) {
       console.log('循环体中：' + i);
   }
   console.log('循环体外：' + i);
   ```

   上方代码的最后一行无法打印结果，也就是说打印会报错。因为用 let 定义的变量 i，只在 {} 这个块级作用域里生效。

   **总结：**用 let 声明，减少 var 声明带来的污染全局空间。

   为了进一步说明 let 不会带来污染，需要说明的是：当我们定义了 let a = 1 时，如果我们在同一个作用域内继续定义 let a = 2 是会报错的



### 3.3 const：定义常量

在程序开发中，有些变量是希望声明后，在业务层就不再发生变化，此时可以用 const 来定义。

举例：

```javascript
const name = 'smyhvae'; // 定义常量
```

用 const 声明的常量，只在局部（块级作用域内）起作用。



## 4 变量的解构赋值

ES6 允许我们，通过数组或者对象的方式，对一组变量进行赋值，这被称为解构。

解构赋值在实际开发中可以大量减少代码量，并且让程序解构更清晰。



### 4.1 数组的解构赋值

**举例：**

通常情况下，我们在为一组变量赋值时，一般是这样写：

```javascript
let a = 0;
let b = 1;
let c = 2;
```

现在我们可以通过数组解构的方式进行赋值：

```javascript
let [a, b, c] = [1, 2, 3];
```

二者效果是一样的。

**解构的默认值：**

在解构赋值时，是允许使用默认值的，举例如下：

```javascript
{
    // 一个变量时
    let [foo = true] = [];
    console.log(foo); // 输出结果，true
}

{
    // 两个变量时
    let [a, b] = ['骗局']; // a 赋值为：骗局，b 没有赋值
    console.log(a + '，' + b); // 输出结果：骗局，undefined
}

{
    // 两个变量时
    let [a, b = 'smyhvae'] = ['骗局']; // a 赋值为：骗局，b 没有赋值
    console.log(a + '，' + b); // 输出结果：骗局，smyhvae
}
```

undefined 和 null 区别：

在赋值时，如果赋值的是 undefined 或者 null，会有什么区别呢？

```javascript
{
    // 两个变量时
    let [a, b = 'smyhvae'] = ['骗局', undefined]; // a 赋值为：骗局，b 没有赋值
    console.log(a + '，' + b); // 输出结果：骗局，smyhvae
}

{
    // 两个变量时
    let [a, b = 'smyhvae'] = ['骗局', null]; // a 赋值为：骗局，b 没有赋值
    console.log(a + '，' + b); // 输出结果：骗局，null
}
```

上方代码分析：

* undefined：相当于什么都没有，此时 b 采用默认值。
* null：相当于有值，但值为 null。



### 4.2 对象的解构赋值

通常情况下，从接口拿到 json 数据后，一般这么赋值：

```javascript
var a = json.a;
var b = json.b;
var c = json.c;
```

上面这样写，过于麻烦了。

现在，同样可以针对对象，进行解构赋值。

举例如下：

```javascript
let { foo, bar } = { bar: '我是 bar 的值', foo: '我是 foo 的值' };
console.log(foo + '，' + bar); // 输出结果：我是 bar 的值，我是 foo 的值
```

上方代码可以看出，对象的解构与数组的解构，有一个重要的区别：数组的元素是按次序排列的，变量的取值由它的位置决定；而对象的属性没有次序，是根据键来取值的。

**圆括号的使用：**

如果变量 foo 在解构之前就已经定义了，此时你再去解构，就会出现问题。下面是错误的代码，编译会报错：

```javascript
let foo = 'haha';
{ foo } = { foo: 'ymyhvae' };
console.log(foo);
```

要解决报错，只要在解构的语句外边，加一个圆括号即可：

```javascript
let foo = 'haha';
({ foo } = { foo: 'smyhvae' });
console.log(foo); // 输出结果：smyhvae
```



### 4.3 字符串解构

字符串也可以解构，这是因为，此时字符串被转换成了一个类似数组的对象。举例如下：

```javascript
const [a, b, c, d] = 'smyhvae';
console.log(a);
console.log(b);
console.log(c);
console.log(d);
```



## 5 for ... of 循环

ES6 中，如果要遍历一个数组，可以这样做：

```javascript
let arr1 = [1, 2, 3, 4, 5];
for (let value of arr1) {
    console.log(value)
}
```

for ... of 的循环可以避免开拓内存空间，增加代码运行效率，所以建议在工作中使用 for ... of 循环。

注意，上面的数组中，for ... of 获取的是数组里的值；for ... in 获取的是 index 索引值。



### 5.1 Map对象的遍历

for ... of 既可以遍历数组，也可以遍历 Map 对象。



## 6 模板字符串

```javascript
var name = 'symhvae';
var age = '26';
console.log('name:' + name + ',age:' + age); // 传统写法
console.log(`name:${name},age;${age}`); // ES6 写法
```

注意：

* 模板字符串使用是反引号\`。
* 可以使用表达式。
* 可以调用函数。



## 7 ES6：函数扩展

### 7.1 前言

ES6 在函数扩展方面，新增了很多特性，例如：

* 箭头函数
* 参数默认值
* 参数解构赋值
* 扩展运算符
* rest 参数
* this 绑定
* 尾调用



### 7.2 箭头函数

定义和调用函数（传统写法）：

```javascript
function fn1(a, b) {
    return a + b;
}
console.log(fn1(1, 2)); // 输出结果：3
```

定义和调用函数（ES6 中的写法）：

```javascript
var fn2 = (a, b) => a + b;
console.log(fn2(1, 2)); // 输出结果：3
```

二者的效果是一样的。

在箭头函数中，如果方法体内有两行代码，那就需要在方法体外边加上{}，如下：

```javascript
var fn2 = (a, b) => {
    console.log('haha');
    return a + b;
}
console.log(fn2(1, 2)); // 输出结果：3
```

上方代码中：

* 如果有且仅有1个参数，则()可以省略。
* 如果方法体内有且仅有1行代码，则{}可以省略，但前提是，这条语句必须是 return 或者没有返回值。



#### 7.2.1 this 的指向（https://www.cnblogs.com/lfri/p/11872696.html）

ES5 中，this 指向的是函数被调用的对象；而 ES6 的箭头函数中，this 指向的是函数被定义时。

简单来说，箭头函数中的 this，是不会变的，是永远绑定在当前的环境下。



### 7.3 参数默认值

**传统写法：**

```javascript
function fn(param) {
    let p = param || 'hello';
    console.log(p);
}
```

上方代码中，函数体内的写法是：如果 param 不存在，就用 hello 字符串做兜底。

**ES6 写法：**（参数默认值的写法）

```javascript
function fn(param = 'hello') {
    console.log(param);
}
```

在 ES6 中定义方法时，可以给方法里的参数加一个默认值（缺省值）：

* 方法被调用时，如果没有给参数赋值，那就是用默认值。
* 方法被调用时，如果给参数复制了新的值，那就用新的值。

如下：

```javascript
var fn2 = (a, b = 5) => {
    console.log('haha');
    return a + b;
}
console.log(fn2(1)); // 第二个参数使用默认值5，输出结果：6
console.log(fn2(1, 8)); // 输出结果：9
```

**提醒1：**默认值的后面，不能再有没有默认值的变量。比如 (a, b, c) 这三个参数，如果给 b 设置了默认值，那么就一定要给 c 设置默认值。

**提醒2：**

看下面这段代码：

```javascript
let x = 'smyh';
function fn(x,y = x){
    console.log(x, y);
}
fn('vae');
```

注意第二行代码，给 y 赋值为 x，这里的 x 是括号里的第一个参数，并不是第一行代码声明的 x。打印结果为：vae vae。

如果把第一个参数改一下：

```javascript
let x = 'smyh';
function fn(z, y = x) {
    console.log(z, y);
}
fn('vae');
```

此时打印结果是：vae smyh



### 7.4 扩展运算符（https://www.cnblogs.com/jin-zhe/p/10034259.html）

注意区分：

* 扩展运算符的格式为 `...`
* rest 运算符的格式为 `...变量名`

拆分数组

举例：

```javascript
var arr = {1, 2, 3, 4};
function fn(a, b, c, d) { // 当不确定方法的参数时，可以使用扩展运算符
    console.log(a);
    console.log(b);
    console.log(c);
    console.log(d);
}
fn(...arr);
```

上方代码中注意，arg 参数之后，不能再添加别的参数，否则编译报错。

其他用法：

```javascript
var arr1 = {1, 2, 3, 4, 5};
var arr2 = arr1; // 传递的是引用
var arr3 = [...arr1]; // 深拷贝
```



### 7.5 rest 运算符

rest 指的是剩余部分，可变参，合并成数组举例：

```javascript
function fn(first, second, ...arg) {
    console.log(arg.length);
}
fn(0, 1, 2, 3, 4, 5, 6); // 调用函数后，输出结果：5
```



## 8 Promise

### 8.1 概述

Promise 对象：代表了未来某个将要发生的事件（通常是一个异步操作）。

ES6 中的 Promise 对象，可以将异步操作以同步的流程表达出来，很好地解决了回调地狱的问题（避免了层层嵌套的回调函数）。在使用 ES5 的时候，在多层嵌套回调时，写完的代码层次过多，很难进行维护和二次开发。



### 8.2 回调地狱的举例

假设买菜、做饭、洗碗都是异步的。

现在的流程是：买菜成功之后，才能开始做饭。做饭成功后，才能开始洗碗。这里面就涉及了回调的嵌套。

ES6 的 Promise 是一个构造函数，用来生成 Promise 实例。



### 8.3 Promise 对象的3个状态

* 初始化状态（等待状态）：pending
* 成功状态：fullfilled
* 失败状态：rejected



### 8.4 使用 Promise 的基本步骤

1. 创建 Promise 对象
2. 调用 Promise 的回调函数 then()

代码格式如下：

```javascript
let promise = new Promise((resolve, reject) => {
    // 进来之后，状态为 pending
    console.log('111'); // 这一行代码是同步的
});
```

