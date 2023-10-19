---
title: JavaScript
date: 2020-02-16 15:38:06
---

## JavaScript
ECMAScript，是几个公司联合ECMA（European Computer Manufacturers Association）组织定制了JavaScript语言的标准。

## 数据类型
Number、字符串、布尔值
>NaN这个特殊的Number与所有其他值都不相等，包括它自己；唯一能判断NaN的方法是通过isNaN()函数
>ES6优化：1.对于字符串需要换行又不想写\n时，可以用反引号"\`" 2.连接字符串觉得`+`麻烦时，可以用`${变量名}`替换
>Number调用`toString()`方法的写法比较特殊，123..toString(); (123).toString();


null，表示空类似于java中的nul，是对象类型但是没有`toString()`方法
undefined，表示未定义，undefined仅仅在判断函数参数是否传递的情况下有用，没有`toString()`方法
数组，js数据可以包含任意数据类型，下标从0开始
> push()向Array的末尾添加若干元素，pop()则把Array的最后一个元素删除掉：
> 如果要往Array的头部添加若干元素，使用unshift()方法，shift()方法则把Array的第一个元素删掉：

对象，键值组成的无序集合
> 属性名包含特殊字符需要用单引号'middle-school',访问时必须用['middle-school']
> 判断对象中是否存在某个属性'name' in person; // true
> 'toString' in person; // true,因为toString定义在object对象中，而所有对象最终都会在原型链上指向object，所以person也拥有toString属性。要判断一个属性是否是person自身拥有的，而不是继承得到的，可以用hasOwnProperty()方法：

```
var person = {
    name: 'Bob',
    age: 20,
    tags: ['js', 'web', 'mobile'],
    city: 'Beijing',
    hasCar: true,
    zipcode: null,
	'middle-school': 'No.1 Middle School'
};
```


## 运算符
**比较运算符：**
1. ==比较，它会自动转换数据类型再比较，很多时候，会得到非常诡异的结果；
2. ===比较，它不会自动转换数据类型，如果数据类型不一致，返回false，如果一致，再比较。

>由于JavaScript这个设计缺陷，不要使用==比较，始终坚持使用===比较。

## 变量
`var a = 123; a = "abc";`
这种变量本身类型不固定的语言称之为动态语言，与之对应的是静态语言比如java。
>js引擎会默认在行的末尾添加";",所以多行时要注意

## strict模式
JavaScript在设计之初，为了方便初学者学习，并不强制要求用var申明变量。这个设计错误带来了严重的后果：如果一个变量没有通过var申明就被使用，那么该变量就自动被申明为全局变量，在同一个页面的不同的JavaScript文件中，如果都不用var申明，恰好都使用了变量i，将造成变量i互相影响，产生难以调试的错误结果。
为了修补JavaScript这一严重设计缺陷，ECMA在后续规范中推出了strict模式，在strict模式下运行的JavaScript代码，强制通过var申明变量，未使用var申明变量就使用的，将导致运行错误。
启用strict模式的方法是在JavaScript代码的第一行写上：`"use strict"`

## 循环
for ... in循环可以遍历对象
> for ... in对Array的遍历得到的是String而不是Number。

```
var o = {
    name: 'Jack',
    age: 20,
    city: 'Beijing'
};
for (var key in o) {
    console.log(key); // 'name', 'age', 'city'
}
```

## Map
js对象的键只能是字符串，所以ES6引入了Map。
```
var m = new Map([['Michael', 95], ['Bob', 75], ['Tracy', 85]]);
```

## iterable
遍历Array可以采用下标循环，遍历Map和Set就无法使用下标。为了统一集合类型，ES6标准引入了新的iterable类型，Array、Map和Set都属于iterable类型。
具有iterable类型的集合可以通过新的`for ... of`循环来遍历，同时修复了`for ... in`循环的一些问题。
iterable内置的forEach方法，它接收一个函数，每次迭代就自动回调该函数。以Array为例：
```
a.forEach(function (element, index, array) {
    // element: 指向当前元素的值
    // index: 指向当前索引
    // array: 指向Array对象本身
    console.log(element + ', index = ' + index);
});
```

## 函数
js函数可以接受比定义的形参更多的实参，关键字arguments，可以获取函数传入的所有参数，它只在函数内部起作用。
为了方便，ES6引入了rest参数，可以将多余的参数存到数组中
```
function abs(x) {
    if (x >= 0) {
        return x;
    } else {
        return -x;
    }
}
var abs = function (x) {
    if (x >= 0) {
        return x;
    } else {
        return -x;
    }
};
```

```
function foo(a, b, ...rest) {
    console.log('a = ' + a);
    console.log('b = ' + b);
    console.log(rest);
}

foo(1, 2, 3, 4, 5);
// 结果:
// a = 1
// b = 2
// Array [ 3, 4, 5 ]

foo(1);
// 结果:
// a = 1
// b = undefined
// Array []
```
### 变量提升
JavaScript的函数定义有个特点，它会先扫描整个函数体的语句，把所有申明的变量“提升”到函数顶部，所以我们在函数内部定义变量时，请严格遵守“在函数内部首先申明所有变量”这一规则。

### 全局作用域
不在任何函数内定义的变量就具有全局作用域。实际上，JavaScript默认有一个全局对象window，全局作用域的变量实际上被绑定到window的一个属性：
```
'use strict';

var course = 'Learn JavaScript';
alert(course); // 'Learn JavaScript'
alert(window.course); // 'Learn JavaScript'
```
### 命名空间
全局变量会绑定到window上，不同的JavaScript文件如果使用了相同的全局变量，或者定义了相同名字的顶层函数，都会造成命名冲突，并且很难被发现。
减少冲突的一个方法是把自己的所有变量和函数全部绑定到一个全局变量中。例如：
```
// 唯一的全局变量MYAPP:
var MYAPP = {};

// 其他变量:
MYAPP.name = 'myapp';
MYAPP.version = 1.0;

// 其他函数:
MYAPP.foo = function () {
    return 'foo';
};
```
把自己的代码全部放入唯一的名字空间MYAPP中，会大大减少全局变量冲突的可能。许多著名的JavaScript库都是这么干的：jQuery，YUI，underscore等等。

### 局部作用域
为了解决块级作用域，ES6引入了新的关键字let，用let替代var可以申明一个块级作用域的变量：
ES6标准引入了新的关键字const来定义常量，const与let都具有块级作用域：
```
'use strict';

function foo() {
    var sum = 0;
    for (let i=0; i<100; i++) {
        sum += i;
    }
    // SyntaxError:
    i += 1;
}
```
### 对象的方法
```
var xiaoming = {
    name: '小明',
    birth: 1990,
    age: function () {
        var y = new Date().getFullYear();
        return y - this.birth;
    }
};

xiaoming.age; // function xiaoming.age()
xiaoming.age(); // 今年调用是25,明年调用就变成26了
```
在一个方法内部，this是一个特殊变量，它始终指向当前对象，也就是xiaoming这个变量。所以，this.birth可以拿到xiaoming的birth属性。
如果以对象的方法形式调用，比如xiaoming.age()，该函数的this指向被调用的对象，也就是xiaoming，这是符合我们预期的。
但是如果单独调用函数，比如getAge()，此时，该函数的this指向全局对象，也就是window。
由于这是一个巨大的设计错误，要想纠正可没那么简单。ECMA决定，在strict模式下让函数的this指向undefined




## 对象
```
typeof 123; // 'number'
typeof NaN; // 'number'
typeof 'str'; // 'string'
typeof true; // 'boolean'
typeof undefined; // 'undefined'
typeof Math.abs; // 'function'
typeof null; // 'object'
typeof []; // 'object'
typeof {}; // 'object'
```
* 不要使用new Number()、new Boolean()、new String()创建包装对象；
* 用parseInt()或parseFloat()来转换任意类型到number；
* 用String()来转换任意类型到string，或者直接调用某个对象的toString()方法；
* 通常不必把任意类型转换为boolean再判断，因为可以直接写if (myVar) {...}；
* typeof操作符可以判断出number、boolean、string、function和undefined；
* 判断Array要使用Array.isArray(arr)；
* 判断null请使用myVar === null；
* 判断某个全局变量是否存在用typeof window.myVar === 'undefined'；
* 函数内部判断某个变量是否存在用typeof myVar === 'undefined'。

### js面向对象编程
js面向对象区分于java，没有类的概念，而是通过原型（prototype）来实现面向对象编程。
例如：创建一个Array对象`var arr = [1, 2, 3];`其原型链是`arr ----> Array.prototype ----> Object.prototype ----> null`,Array.prototype定义了indexOf()、shift()等方法，因此你可以在所有的Array对象上直接调用这些方法。
两个对象之间可以通过原型来指定继承关系：
```
var Student = {
    name: 'Robot',
    height: 1.2,
    run: function () {
        console.log(this.name + ' is running...');
    }
};
var xiaoming = {
    name: '小明'
};
xiaoming.__proto__ = Student;
```
一般编程使用方式：
```
// 原型对象:
var Student = {
    name: 'Robot',
    height: 1.2,
    run: function () {
        console.log(this.name + ' is running...');
    }
};

function createStudent(name) {
    // 基于Student原型创建一个新对象:
    var s = Object.create(Student);
    // 初始化新对象:
    s.name = name;
    return s;
}

var xiaoming = createStudent('小明');
xiaoming.run(); // 小明 is running...
xiaoming.__proto__ === Student; // true
```
### 构造函数创建对象
js可以定义一个普通函数，然后通过关键字`new`创建对象：
```
function Student(name) {
    this.name = name;
    this.hello = function () {
        alert('Hello, ' + this.name + '!');
    }
};
var xiaoming = new Student('小明');
xiaoming.name; // '小明'
xiaoming.hello(); // Hello, 小明!
```
### class继承
传统的原型链继承实现起来是比较麻烦的，所以新的关键字class从ES6开始正式被引入到JavaScript中，class的目的就是让定义类更简单。



---
tip1: 本地运行得html和js会因为浏览器的安全限制以file://开头的地址无法执行联网等js代码。
tip2: js引擎会在每个语句的结尾加上`;`，可能会导致如下错误。
	function foo() {
		return
			{ name: 'foo' };
	}

	foo(); // undefined
tip3: 未用var声明的变量都是全局变量，同一个页面的不同js文件中的变量会互相影响； strict模式强制使用var
tip4: 实际上JavaScript对象的所有属性都是字符串，不过属性对应的值可以是任意数据类型。
tip5: 判断一个js对象是否有改属性（'name' in xiaoming; // true）而不是继承的可以用函数hasOwnProperty()
tip6: js对象可以作为Map来用，但是为了规避键值只能是字符串的问题所以ES6引入了Map
tip7: Array Map 和 Set都属于iterable类型所以可以用ES6的for of遍历，interable内置的forEach方法遍历更好
tip8: 由于JavaScript的变量提升（js的函数定义有个特点，它会先扫描整个函数体的语句，把所有申明的变量“提升”到函数顶部）这一怪异的“特性”，我们在函数内部定义变量时，请严格遵守“在函数内部首先申明所有变量”这一规则
tip9: 不在任何函数内定义的变量就具有全局作用域,被绑定到window对象上。
tip10: 为了解决块级作用域，ES6引入了新的关键字let，用let替代var可以申明一个块级作用域的变量，这样后面的语句就访问不到i，for (let i=0; i<100; i++) { sum += i; }
tip11: js可以使用apply改变函数的this指向， fuc1.apply(null, [])
tip12: 箭头函数看上去是匿名函数的一种简写，但实际上，箭头函数和匿名函数有个明显的区别：箭头函数内部的this是词法作用域，由上下文确定。可以修复this指向的问题

diffrence with java:
1. 12.000===12, js中整型和浮点型都是Number类型，可以直接进行比较
2. '' 和 "", js中都表示String，java中前者表示char，后者表示String
3. js数组可以直接越界赋值而不报错，var arr = [1, 2, 3]; arr[5] = 'x'; arr; // arr变为[1, 2, 3, undefined, undefined]
4. 遍历一个js对象可以用for in循环， java需要用反射
5. js的函数中可以嵌套函数













