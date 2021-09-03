## this 的指向问题

- this 永远指向一个对象；
- this 的指向完全取决于函数调用的位置；

```js
function fun() {
  console.log(this.s);
}

var obj = {
  s: '1',
  fn: fun,
};

var s = '2';

obj.fn(); // 1 this指向obj
fun(); // 2 this指向window
```

`obj.fn()`的调用中，因为运行环境在`obj`对象内，因此函数中的 this 指向对象`obj`;

全局作用域下调用`fun()`，函数中的 this 就会指向全局作用域对象 window；

**问题: this 的指向为啥会发生改变？this 指向改变到底发生在什么时候？**

```js
const A = {
  name: '张三',
  fn: function() {
    console.log('姓名：'， this.name);
  }
}

const B = {
  name: '李四',
}

B.fn = A.fn;

B.fn(); // 姓名：李四
A.fn(); // 姓名：张三
```

上面代码中，`A.fn`属性被赋值给`B.fn`，也就是`A`对象将匿名函数的**地址（指针）**赋值给`B`对象；

那么在调用时，函数分别根据运行环境的不同，指向对象`A`和`B`;

```js
function foo() {
  console.log(this.a);
}

var obj2 = { a: 2, fn: foo };
var obj1 = { a: 1, fn: obj2 };

obj1.fn.fn(); // 2
```

`obj1`对象的`fn`属性值是`obj2`对象的地址，而`obj2`对象的`fn`属性的值是函数`foo`的地址；

函数`foo`的调用环境是在`obj2`中的，因此`this`指向对象`obj2`；

### 对 this 使用最频繁的几种情况的总结，最常见的基本就是一下 5 种：

**对象中的方法、事件绑定、构造函数、定时器、函数对象的 call()、apply()方法**

#### 事件绑定中的 this

事件绑定共有三种方式：行内绑定、动态绑定、事件监听

行内绑定的两种情况：

```html
<input type="button" value="按钮" onclick="clickFun()" />

<script>
  function clickFun() {
    console.log(this); // 此函数的运行环境在全局window对象下，因此this指向window；
  }
</script>

<input type="button" value="按钮" onclick="console.log(this)" />
<!-- 运行环境在节点对象中，因此this指向本节点对象 -->
```

动态绑定与事件监听：

```html
<input type="button" value="按钮" id="btn" />
<script>
  const $btn = document.getElementById('btn');
  $btn.onclick = function () {
    console.log(this); // this指向$btn节点；
  };
  $btn.addEventListener('click', function () {
    console.log(this); // this指向$btn节点；
  });
</script>
```

#### 构造函数中的 this

```js
function Pro() {
  this.x = 1;
  this.y = function () {
    console.log(this.x);
  };
}

var p = new Pro();

p.y(); // 1

/**
 * new 构造函数示意图
 * 1. 创建一个空对象
 * 2. 将本对象的原型指向Pro.prototype(对象继承Pro.prototype)
 * 3. 将构造函数中的this指向本对象
 * 4. 执行构造函数的代码为对象赋值 {x: 1, y: 匿名函数地址}
 * 5. 返回本对象地址
 */
```

#### 定时器中的 this

```js
var obj = {
  fn: function () {
    console.log(this);
  },
};

setInterval(obj.fn, 1000); // this指向window
setInterval(() => {
  obj.fn(); // this指向obj对象
}, 1000);
setInterval('obj.fn()', 1000); // this指向obj对象
```

`setInterval()`是`window`对象下内置的一个方法，接受两个参数，第一个参数允许是一个函数或者是一段可执行的 JS 代码，第二个参数则是执行前面函数或者代码的时间间隔；

#### 函数对象的 call()、apply()、bind()方法

**call()方法**

> call 方法使用的语法
> [函数名称].call(obj, arg1, arg2, ...argN);
> obj：函数内 this 要指向的对象
> arg1, arg2, ...argN：参数列表，参数与参数之间用`,`隔开

```js
var ls = { name: 'ls' };
var zs = { name: 'zs' };
function fun(age) {
  console.log(this.name, age);
}
fun(23); // this指向window

fun.call(zs, 23); // zs 23
```

**apply()方法**

> apply 方法使用的语法
> [函数名称].apply(obj, [arg1, arg2, ...argN]);
> obj：函数内 this 要指向的对象
> [arg1, arg2, ...argN]：参数列表，要求格式为数组

```js
var ls = { name: 'ls' };
var zs = { name: 'zs' };
function fun(age, sex) {
  console.log(this.name, age, sex);
}

fun.apply(zs, [23, 'nan']); // zs 23 nan
```

注意：`call` 和 `apply` 的作用一致，区别仅仅在函数实参参数传递的方式上；

这个两个方法的最大作用基本就是用来强制指定函数调用时 `this` 的指向；

**bind()方法**

> bind 方法使用的语法
> [函数名称].bind(obj[,arg1[, arg2[, ...argN]]]);
> obj：函数内 this 要指向的对象
> arg1, arg2, ...argN：当目标函数被调用时，被预置入绑定函数的参数列表中的参数。
> 返回值：返回一个原函数的拷贝，并拥有指定的 this 值和初始参数

```js
this.x = 9; // 在浏览器中，this 指向全局的 "window" 对象
var module = {
  x: 81,
  getX: function () {
    console.log(this.x);
  },
};

module.getX(); // 81

var retrieveX = module.getX;
retrieveX();
// 返回 9 - 因为函数是在全局作用域中调用的

// 创建一个新函数，把 'this' 绑定到 module 对象
// 新手可能会将全局变量 x 与 module 的属性 x 混淆
var boundGetX = retrieveX.bind(module);
boundGetX(); // 81
```

_偏函数_

> bind() 的另一个最简单的用法是使一个函数拥有预设的初始参数。只要将这些参数（如果有的话）作为 bind() 的参数写在 this 后面。当绑定函数被调用时，这些参数会被插入到目标函数的参数列表的开始位置，传递给绑定函数的参数会跟在它们后面。

```js
function list() {
  return Array.prototype.slice.call(arguments);
}

function addArguments(arg1, arg2) {
  return arg1 + arg2;
}

var list1 = list(1, 2, 3);
console.log(list1); // [1, 2, 3]

var result1 = addArguments(1, 2);
console.log(result1); // 3

// 创建一个函数，它拥有预设参数列表。
var leadingThirtysevenList = list.bind(null, 37);

// 创建一个函数，它拥有预设的第一个参数
var addThirtySeven = addArguments.bind(null, 37);

var list2 = leadingThirtysevenList();
console.log(list2); // [37]

var list3 = leadingThirtysevenList(1, 2, 3);
console.log(list3); // [37, 1, 2, 3]

var result2 = addThirtySeven(5);
console.log(result2); // 37 + 5 = 42

var result3 = addThirtySeven(5, 10);
console.log(result3); // 37 + 5 = 42 ，第二个参数被忽略
```

#### 箭头函数中的 this

箭头函数并没有自己的 this

> 箭头函数体内的 this 对象，就是定义该函数时所在的作用域指向的对象，而不是使用时所在的作用域指向的对象。

```js
var name = 'window';

var A = {
  name: 'A',
  sayHello: () => {
    console.log(this.name);
  },
};

A.sayHello(); // window。 this指向window
```

**最后是使用箭头函数其他几点需要注意的地方**

1. 不可以当作构造函数，也就是说，不可以使用 new 命令，否则会抛出一个错误。
2. 不可以使用 arguments 对象，该对象在函数体内不存在。如果要用，可以用 rest 参数代替。
3. 不可以使用 yield 命令，因此箭头函数不能用作 Generator 函数。
