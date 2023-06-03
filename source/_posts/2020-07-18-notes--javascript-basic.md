---
title: JavaScript基础
date: 2020-07-18
tags: 
  - notes
  - javascript
categories: technology
keywords: 'basic javascript'
---

# 一、语言基础

## 1.1 语法

### 1.1.1 变量

JavaScript的变量是松散类型的, 也就是说同一个变量可以存储不同数据类型的值(且可在运行期动态改变). 它不像Java等强变量类型语言, 特定数据类型的变量只能存储特定数据类型的值. 变量的声明使用`var`关键字(ES6中新增了`let`和`const`, 应该避免继续使用`var`).

```javascript
// 变量声明关键字为: let, const, var
// let i = 10;
// const i = 10;
var i = 10;

// 同一个变量可以存储不同数据类型的值
var str = 'Hello world';
// 在运行期动态改变变量值的数据类型: 数据类型由Number变为String
i = 'i changed to str';
```

### 1.1.2 数据类型

JavaScript中共有6种数据类型: `Undefined`, `Null`, `Boolean`, `Number`, `String`, `Object`.

- `Undefined`: 只有一个值: `undefined`.
- `Null`: 只有一个值`null`: `let v = null;`
- `Boolean`
- `Number`: 整数和浮点数
- `String`
- `Object`

### 1.1.3 操作符

开发语言中的操作符大都是差不多的, 如自增操作符(`++i`, `i++`), 自减操作符(`--i`, `i--`), 布尔操作符(`!true`, `!false`, `let result = true && false`, `let result = false || true`), 乘性操作符(`let result = a * b`, `let result = a / b`, `let result = a % b`), 加性操作符(`let result = a + b`, `let result = a - b`), 关系操作符(`<`, `>`, `<=`, `>=`), 相等操作符(`==`, `!=`, `===`, `!==`), 条件操作符(`let result = boolean_expression ? true_value : false_value`), 赋值操作符(`let num = 10`).



对于操作符, 需要注意的点有:

- 相等操作符`==`和`===`的区别: 前者在比较的时候会进行类型转换, 比如`let result = ('10' == 10);`的值是`true`, 但是将`==`换成`===`的话结果就为`false`了. 自动类型转换可能会带来很多隐藏问题, 所以在开发的时候一般避免使用`==`操作符.

- 逻辑非操作符`!`可作用于任意数据类型: 在Java中, 逻辑非操作符`!`只能用于布尔类型, 但是JavaScript中它可以用于任意数据类型. **当它作用于空字符串, 0, null, undefined, false**这些表示"没有"的意思的表达式的时候都会返回`true`:

  ```javascript
  // 以下所有输出结果都是true
  let withStr = !'';
  console.log(withStr);
  
  let withZero = !0;
  console.log(withZero);
  
  let withNull = !null;
  console.log(withNull);
  
  let withUndefined = !undefined;
  console.log(withUndefined);
  
  let withFalse = !false;
  console.log(withFalse);
  ```

- 逻辑与操作符`&&`和逻辑或操作符`||`可作用于任意数据类型: 在Java中他们仅用于布尔类型之间的操作, 但是在JavaScript中却能用在不同数据类型之间, 且这些隐藏用法在写React的时候非常有用, 具体内容在2.1中描述.

### 1.1.4 语句

- `if`语句: `if (condition) {...}`. 需要注意的是这里的`condition`可以是任何表达式, 只要不是表示"没有"的, 那么这个条件都成立. JavaScript中表示"没有"的有: `空字符串`, `0`, `null`, `undefined`, `false`.

  ```javascript
  // 下面所有语句执行都没有输出
  if ("") console.log('Hello!');
  if (0) console.log('Hello!');
  if (null) console.log('Hello!');
  if (undefined) console.log('Hello!');
  if (false) console.log('Hello!');
  ```

- `while`语句: `while(true) { ... }`

- `for`语句: `for (let i = 0; i < arr.length; i++) { ... }`

- `for-in`语句: `for (let pro in obj) {...}`

- `for-of`语句:  `for (let pro of obj) {...}`

- `switch`语句

## 1.2 数组

JavaScript中数组和常规开发语言中的数组定义是一样的, 但是需要注意:

- 数组属于JavaScript基本数据类型中的对象类型(`Object`, 它的键名是依次排序的整数`0, 1, 2`), 使用`typeof`运算符返回的是`object`.

- 数组的`length`属性返回的是`最大的键名+1`.

- **`length`属性是可写的, 可以通过手动设置数组的`length`属性来改变数组的大小. 如设置`arr.length=0`来清空数组**.

- 由于JavaScript中变量是松散类型的, 因此数组中元素之间的数据类型可以不同.

- **如果数组中两个逗号之间没有值, 则我们称这个位置为空位**.

  - 使用`delete`命令删除一个元素, 那么被删除元素的位置就形成了空位.
  - 读取空位的值返回`undefined`
  - 空位不影响数组的`length`属性.

  ```javascript
  // typeof 运算符输出object
  let arr = ['a', 'b', 'c', 'd', 'e'];
  console.log(typeof arr);
  
  
  // 可以手动改变length的属性
  console.log(arr.length);
  arr.length = 4;
  // 将length属性设置为4, 删除了最后一个元素, 现在数组变成了['a', 'b', 'c', 'd']
  console.log(arr);
  
  
  // 元素之间数据类型可以不同
  arr[0] = 1;
  // [1, 'b', 'c', 'd']
  console.log(arr);
  
  
  // 使用delete删除一个元素, 这个元素的位置成了空位
  delete arr[1];
  // [ 1, <1 empty item>, 'c', 'd' ]
  console.log(arr);
  // 访问空位输出undefined
  console.log(arr[1]);
  // 空位不影响数组的length属性, 还是输出4
  console.log(arr.length);
  ```

## 1.3 函数

- JavaScript中函数也是一个对象, 他是`Function`类型的实例. 函数名是一个引用类型的变量, 它指向这个函数对象(如果是匿名函数那么指向这个函数的变量其实就是指向这个对象).

  ```javascript
  //标识符func是一个变量(也是这个函数的名字), 他指向代表这个函数的对象.
  function func() {
  
  }
  
  //使用匿名函数也一样, 变量func1 指向这个函数对象.
  let func1 = function() {
  
  };
  ```

- 对于一个函数, 可以认为它存在以下三种角色:

  - 对象角色: 当作为对象角色的时候, 因为他是`Function`类型的实例, 所以可以直接调用`Function.prototype`上面定义的方法.
  - 函数角色: 当作为函数角色的时候, 可以在后面加上小括号, 表示执行这个函数.
  - 构造器角色: 当作为构造器角色的时候, 我们可以在前面加上`new`运算符, 从而生成一个对象.

  ```javascript
  function Student(name) {
  
      this.name = name;
      console.log(this.name);
  }
  
  // 作为对象角色, Student这个标识符是变量标识符, 指向一个函数对象, 这里调用这个函数对象的name属性, 他是从Function中继承来的.
  console.log(Student.name);
  
  // 直接执行这个函数, 作为函数角色, 为this.name赋值, 打印'jeb', 没有任何返回值.
  // 在这种情况下, this的指向就是复杂问题了, 看第七章. 在这里的话指向的是window对象.
  Student('jeb');
  
  // 使用new操作符, 作为构造器角色使用, 生成一个新的对象, 为this.name赋值, 打印this.name 返回新生成的那个对象.
  // 在这种情况下, 函数里面的this指向的是新生成的那个对象.
  let jeb = new Student('jeb');
  ```

- JavaScript中使用`function`来声明函数且函数不能进行重载.

  ```javascript
  // num1和num2前面不能写变量声明关键字 如 var num1, var num2
  // 语法上不需要声明这个函数是否有返回值
  function add(num1, num2) {
      return num1 + num2;
  }
  
  // 不存在重载
  function func() {
      console.log('func without args.');
  }
  
  function func(num) {
      console.log('func with arg, num is: ', num);
  }
  // 由于不存在重载, 后面定义的同名函数会覆盖前面定义的同名函数, 因此这里实际调用的都是后面定义的函数
  // func with arg, num is:  undefined
  // func with arg, num is:  1
  func();
  func(1);
  ```

- **函数的作用域和变量一样, 是声明时所在的作用域**, 与运行时所在的作用域无关.

  ```javascript
  let a = 1;
  function logA() {
      console.log(a);
  }
  
  function logB() {
      let a = 2;
      logA();
  }
  
  // 1
  logB();
  ```

## 1.4 对象

### 1.4.1 对象的构造

JavaScript中对象是`Object`类型的实例. 数组是对象, 函数也是对象. 生成对象的方式一般有如下几种:

- 字面量对象.
- 构造函数.
- `Object()`构造函数.
- ES6中的class.

```javascript
// 字面量对象
let jeb = {
    no: '15230995030',
    name: 'jeb',
    age: 22,

    sayHello: function () {
        console.log('Hello world!');
    }
};


// 构造函数
function Student(no, name, age) {
    this.no = no;
    this.name = name;
    this.age = age;

    this.sayHello = function() {
        console.log('Hello world!');
    }
}
let alpha = new Student('15230995031', 'alpha', 22);


// Object构造函数, 前面1.3函数一节说过当一个函数作为构造函数角色的时候会返回一个对象.
// 因此这里由Object函数充当构造函数角色返回了一个自身没有任何属性的对象.
let alpha2j = new Object();
alpha2j.no = 'Hello world!';
alpha2j.name = 'alpha2j';
alpha2j.age = 22;


// 使用class对类结构进行定义, 然后new这个类的实例. 在ES6.md里面描述, 这里不重复了.
// ...
```

### 1.4.2 对象的访问

可以通过两种不同的方式对对象进行访问:

- 对象.属性名: `let name = jeb.name;`.
- 对象[属性名]**(如果属性名是数字的话, 可以直接输入这个数字, 而不用输入这个数字的字符串(这正是数组对象的获取方式: `arr[index]`). 需要重视这种实际开发中非常有用的语法. 如我们可以根据外界的输入来获取某个对象的属性值, 如果用点号是没有办法获取的)**:
  - 字符串的属性名: `let name = jeb['name'];`
  - 数字的属性名: `let element = arr[1];`



查看对象所有属性的方式: 使用`Object.keys`方法.

```javascript
let person = {
    name: 'jeb',
    age: 12
};
// [ 'name', 'age' ]
console.log(Object.keys(person));
```



删除对象属性的方式: 使用`delete`命令, 属性删除成功之后会返回`true`. 如果删除一个不存在的属性, JavaScript不会报错, 而是返回true.

```javascript
let person = {
    name: 'jeb',
    age: 12
};

let deleteResult = delete person.name;
console.log(`${deleteResult.name}`);
// name属性已经不存在, 这里不会报错, 会返回true.
deleteResult = delete person.name;
// true
console.log(`${deleteResult}`);
```



操作对象的属性: JavaScript提供了属性描述对象, 可以通过`Object`提供的一些方法和这个属性描述对象, 对一个对象的属性进行操作. [详情链接](https://wangdoc.com/javascript/stdlib/attributes.html).

## 1.5 其他

### 1.5.1 变量提升

​	JavaScript引擎在工作的时候, 会先获取所有被声明的变量, 然后再一行一行地执行. 因此, 所有变量的声明语句都会被提升到代码的头部, 这就叫做变量提升. 由于变量提升的存在, 某些情况下可能代码可以正常运行而不会报错:

```javascript
// 由于变量提升的存在, 下面的代码不会报错.
console.log(a);
var a = 1;

// 变量提升后的代码
var a;
console.log(a);
a = 1;

// 因此控制台得到的输出是undefined
```

## 1.5.2 箭头函数

箭头函数本身没有`this`变量, 箭头函数的`this`都是指向定义它的时候外部函数的`this`.

# 二、易混淆知识点

## 2.1 &&和||

​	在`java`中, 逻辑与`&&`和逻辑或`||`只能用于布尔类型操作数, 且只会返回`true`或者`false`. 但是JavaScript中他们是**可以用于任何类型的操作数**的, 在有一个操作数不是布尔类型的时候返回值就不一定是布尔类型. **在有一个操作数不是布尔类型的情况下**, 他们遵循如下规则:

- `&&`:

  - 如果有一个操作数是表示"没有"的(空字符串, 0, `null`, `undefined`), 那么返回这个操作数.
  - 如果两个操作数都不是表示没有的(空字符串, 0, `null`, `undefined`), 返回第二个操作数.

  ```javascript
  // 有一个表示 "没有" 的操作数, 则返回这个操作数.
  // 分别返回空串, 0, undefined, null
  console.log('' && {name: 'jeb'});
  console.log(0 && {name: 'jeb'});
  console.log({name: 'jeb'} && undefined);
  console.log({name: 'jeb'} && null);
  
  
  // 两个操作数都表示 "有", 返回后面那个操作数.
  // {name: 'jeb'}
  console.log(1 && {name: 'jeb'});
  // 1
  console.log({name: 'jeb'} && 1);
  // {age: 22}
  console.log({name: 'jeb'} && {age: 22});
  ```

- `||`:

  - 如果两个操作数都是表示"没有"的, 那么返回第二个表示没有的操作数.
  - 返回第一个表示"有"的操作数.

  ```javascript
  // 两个操作数都表示 "没有" , 返回第二个表示 "没有" 的操作数
  // undefined
  console.log(0 || undefined);
  // null
  console.log('' || null);
  
  
  // 返回第一个表示 "有" 的操作数
  // {name: 'jeb'}
  console.log('' || {name: 'jeb'});
  // {name: 'jeb'}
  console.log(0 || {name: 'jeb'});
  // name: 'jeb'}
  console.log({name: 'jeb'} || undefined);
  // {name: 'jeb'}
  console.log({name: 'jeb'} || null);
  ```


## 2.2 undefined和null

`undefined`和`null`都是JavaScript的基本数据类型, 它们之间的区别如下:

- `undefined`: 声明一个变量但是没有给它赋过任何值, 或者使用了一个对象上不存在的属性, 这时候都返回`undefined`.

  ```javascript
  // 仅仅声明了但是没有赋值
  let justDeclare;
  // undefined
  console.log(justDeclare);
  
  // obj上根本没有name属性, 所以也是undefined
  let obj = {};
  // undefined
  console.log(obj.name);
  ```

- `null`: 显式地为变量赋值`null`.

  ```javascript
  // 将变量显式赋值为null
  let assignToNull = null;
  // null
  console.log(assignToNull);
  ```


## 2.3 typeof和instanceof

### 2.3.1 typeof

如果要执行的某些操作需要限定变量的数据类型, 这时我们需要能够知道变量数据类型的能力:

```javascript
// 假设定义这个函数的目的是: 我要求两个整型数字相加的结果
// 在Java中可能是这样, 我们可以提供参数类型, 让编译器帮我们检查
// public int add(int num1, int num2) {
//     return num1 + num2;
// }
function add(num1, num2) {
    // 编译器不会帮我们检查, 我们得有种确保num1和num2两个变量里面存的数据的类型是整型的机制
    return num1 + num2;
}

// 由于JavaScript的松散类型变量, 我们在调用的时候传入了非数字,
// 因此尽管 '+' 运算符的类型转换这里不会报错, 但是明显它没有正确地使用这个函数,
// 得出的结果可能也不会如意.
add(true, false);
```



上面的例子中, 假如存在一种机制来对传入变量的数据类型进行合法性判断: 符合条件的处理, 不符合条件的拒绝处理, 那么函数的调用者就没办法以不合法的目的来使用这个函数了. `typeof`运算符就是负责提供一个变量的数据类型的: `let varType = typeof variable`. `typeof`的返回值有如下几种:

- `undefinded`: `undefined`变量.
- `boolean`: 布尔类型变量.
- `string`: 字符串类型变量.
- `number`: 数字类型变量.
- `object`: 对象类型变量. 普通对象, 数组, `null`等.
- `function`: 虽然函数属于基本数据类型中的对象, 但是与`typeof`一起使用时返回值为`function`, 而不是`object`.

### 2.3.2 instanceof

`instanceof`主要是用来判断左边对象是否为右边构造函数的实例. 他的原理是检查能否在左边对象的原型链上找到右边构造函数的`prototype`属性.

```javascript
let Person = function () {};

let alpha = new Person();
// alpha刚被new出来, 他有一个内部指针指向Person.prototype, 所以Person.prototype是存在于alpha的原型链上的
// true
console.log(alpha instanceof Person);

// 这里手动改变Person.prototype后, {} 并不在alpha对象的原型链上, 所以这里返回是false
Person.prototype = {};
// false
console.log(alpha instanceof Person);
```

## 2.4 for...in和for...of

### 2.4.1 for...in

用来循环遍历一个对象的全部属性. 注意:

- 遍历的是对象的所有可遍历(`enumerable`的属性, 会跳过不可遍历的属性).
- 不仅遍历对象自身的属性, 还遍历继承的属性(所以一般和`hasOwnProperty()`方法一起使用, 判断属性是自身属性才进行处理).

```javascript
let person = {
    name: 'jeb',
    age: 18
};

for (const key in person) {
    if (person.hasOwnProperty(key)) {
        // ...
    }
}
```

### 2.4.2 for...of

`for...of`是ES6中新添加的特性, 只要一个对象具有`iterator`接口, 那么这个对象就可以使用该方式来循环. 内建库中可以使用该循环的有数组, Set和Map结构, 字符串等. 与`for...in`不同的是, 它直接获得对象的元素值, 而不是对象的键.

# 三、常用标准库

常用标准库的API有非常多, 一般情况下可以在使用到的时候再查看API文档. 这里记录一些开发时比较常用的特性.

## 3.1 Object

JavaScript中所有对象都是`Object`的实例, **意思是说我们所有对象都自动拥有`Object.prototype`上的属性和方法**.

### 3.1.1 常用API

- `Object.keys(obj)`: 传入一个对象参数, 返回一个数组. **数组的成员是对象自身的属性名(并不包含继承来的)**.
- `Object.getPrototypeOf(obj)`: 我们知道每个对象都有一个内部指针, 指向创建它的构造函数或者类的`prototype`属性指向的对象, 这个方法就是获取对象的内部指针指向的这个对象的.
- `Object.getOwnPropertyDescriptor(obj, propertyKey)`: 获取对象指定的属性的描述对象.
- `Object.defineProperty(obj, propertyKey, descriptor)`: 使用过描述符为对象定义一个属性, 返回定义完属性的对象.
- `Object.prototype.hasOwnProperty(propertyKey)`: 判断当前对象是否拥有某个属性(不包括继承来的).


### 3.1.2 属性描述符对象

JavaScript中有一个数据结构可以用来描述对象的属性, 控制他的行为. 如该属性的值, 属性是否可写, 是否可遍历等. 我们把这个数据结构叫做属性描述符对象:

```javascript
{
    value: 'Hello World!',
    writable: true,
    enumerable: true,
    configurable: true,
    get: undefined,
    set: undefined
}
```



这里是它们的作用:

- `value`: 这个属性的值. 默认为`undefined`.

- `writable`: 表示属性的值是否可改变. 默认为`true`.

  ```javascript
  let obj = Object.defineProperty({}, 'name', {
      value: 'jeb',
      writable: false
  });
  
  // jeb
  console.log(obj.name);
  obj.name = 'alpha';
  // 还是jeb, 因为writable设置为false了, 表示值不能改变.
  console.log(obj.name);
  ```

- `enumerable`: 表示属性是否可遍历. 默认为`true`. 如果设置为`false`, 那么`for...in`, `Object.keys()`, `JSON.stringfy()`操作就会跳过这个属性.

- `configurable`: 阻止对这个属性的一些操作. 如使用`delete`删除这个属性, 更改这个属性的描述符.(如果不显示设置`writable:true`连`value`的值都没办法改变.

  ```javascript
  let obj = Object.defineProperty({}, 'name', {
      value: 'jeb',
      // 如果加上这个设置的话即使是设置了configurable, value的值还是可以变的.
      // writable: true,
      configurable: false
  });
  
  // jeb
  console.log(obj.name);
  obj.name = 'alpha';
  // jeb
  console.log(obj.name);
  
  // 异常, 因为前面没有显示设置writable: true.
  Object.defineProperty(obj, 'name', {
      value: 'alpha2j'
  });
  console.log(obj.name);
  ```

- `get`和`set`: 属性的取值函数和存值函数. 定义了这两个属性后就不能同时将`value`属性和将`writable`设置为`true`了.

  ```javascript
  let obj = Object.defineProperty({_name: 'jeb'}, 'name', {
      get: function () {
          return this._name;
      },
      set: function (value) {
          this._name = 'value by setter: ' + value;
      }
  });
  
  // 其实返回的就是_name的值, 因为我们取值器里面的设置
  console.log(obj._name);
  console.log(obj.name);
  
  // 通过设值器设置值
  obj.name = 'alpha';
  // 等同于console.log(obj._name);
  console.log(obj.name);
  ```


## 3.2 Array

JavaScript中的数组都是`Array`的实例, 常用API如下:

- `Array.isArray(arr)`: 判断一个`arr`变量是否为数组.

- `Array.prototype.push()`, `Array.prototype.pop()`: 压入数组尾部和从尾部弹出

- `Array.prototype.shift()`, `Array.prototype.unshift()`: 压入数组头部和从头部弹出

- `Array.prototype.join(split)`: 使用`split`参数作为分隔符将成员连接成一个字符串返回.

  ```javascript
  let arr = [1, 2, 3];
  // 1-2-3
  console.log(arr.join('-'));
  ```

- `Array.prototype.concat()`: 合并数组并返回, 参数中传入的原数组不变.

- `Array.prototype.reverse()`: 反转数组元素的次序(**会改变原数组顺序**).

- `Array.prototype.slice()`: 提取数组一部分返回, 原数组不变.

  ```javascript
  let arr = [1, 2, 3, 4, 5];
  
  // [ 3, 4 ]  包括start位置元素, 但是不包括end位置的元素
  let result = arr.slice(2, 4);
  console.log(result);
  
  // [ 3, 4, 5 ]  只有start则从start(包括start)开始到结束
  result = arr.slice(2);
  console.log(result);
  ```

- `Array.prototype.sort()`: 没有参数的话按字典顺序排序. 可以传入一个参数自定义排序规则.

- `Array.prototype.map()`: **将数组的所有成员一次传入参数的函数中, 然后将每次的执行结果组成一个新数组返回**.

  ```javascript
  let arr = [1, 2, 3, 4, 5];
  
  let result = arr.map((value, index, array) => {
      return value + 100;
  });
  // [ 101, 102, 103, 104, 105 ]
  console.log(result);
  ```

- `Array.prototype.forEach()`: 和`map`操作相似, 对数组的每个成员进行一次操作, 但是不会将每次操作的结果组成新数组返回.

  ```javascript
  let arr = [1, 2, 3, 4, 5];
  
  arr.forEach((value, index, array) => {
      console.log(`[${index}]=${value}`);
  });
  ```

- `Array.prototype.filter()`: 用于过滤数组成员, 将满足条件的成员合成一个数组返回.

- `Array.prototype.reduce()`: 对于每个数组成员, 将他传入参数函数中进行处理, 处理完后返回一个值, 将这个值当成下个数组成员处理时的参数. 如此循环, 最后一个数组成员处理完后将他的值返回, 作为`reduce`函数的返回值.

  ```javascript
  let arr = [1, 2, 3, 4, 5];
  
  //相当于100 + 1 + 2 + 3 + 4 + 5
  let result = arr.reduce((previousValue, currentValue, currentIndex) => {
      return previousValue + currentValue;
  }, 100);
  //115
  console.log(result);
  ```

## 3.3 String

[文档地址](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String)

## 3.4 Math

[文档地址](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Math)

## 3.5 Date

[文档地址](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date), 下面是一些简单的初始化方式:

```javascript
//初始化方式
// let date = new Date();
// let date = new Date(1546185600000);
let date = new Date(2019, 1, 0, 0, 0, 0, 0);

//返回距离UTC时间的毫秒数
console.log(date.valueOf());
//返回格式化后的本地时间
console.log(date.toLocaleString());
console.log(date.toLocaleDateString());
console.log(date.toLocaleTimeString());

//还有一些get和set方法
//...
```

## 3.6 JSON

对于JSON, 重要的知识点如下:

- 使用`JSON.parse()`将一个JSON字符串解析为JavaScript对象.

- 使用`JSON.stringfy()`将一个JavaScript对象序列化为JSON字符串.

- 如果想使用`JSON.parse()`来将一个字符串解析为JavaScript对象, 那么这个字符串的字段名和字符串类型的值类型一定要使用双引号`""`, 而不能使用单引号`''`, 否则解析引擎无法正常工作.

  ```javascript
  let jsonStr = '{\'name\': \'alpha\'}';
  // 正确的写法, 使用双引号代替字段名和字符串类型值的单引号
  // let jsonStr = '{"name": "alpha"}';
  // 解析的时候抛异常了
  let obj = JSON.parse(jsonStr);
  ```

## 3.7 RegExp

### 3.7.1 创建方式

JavaScript中创建正则表达式的方式有两种:

- 字面量, 斜杆开始和结束: `let regex = /abc/;`

- 使用`RegExp`构造函数: 

  ```javascript
  // 字面量构造方式
  // let regexp = /abc/;
  // RegExp构造函数构造方式
  let regexp = new RegExp('abc');
  
  // 还可以接e受第二个参数, 表示修饰符.  等价于 /abc/i
  let regexp1 = new RegExp('abc', 'i');
  ```


### 3.7.2 实例属性

正则对象的实例属性分为两类: 修饰相关属性和修饰无关属性.

- 修饰相关: 返回一个布尔值, 表示对应的修饰符是否设置.

  ```javascript
  // 这里 i, g, m 三个修饰符都设置上了
  let regexp = /abc/igm;
  
  // true
  console.log(regexp.ignoreCase);
  // true
  console.log(regexp.global);
  // true
  console.log(regexp.multiline);
  ```


- 修饰无关:

  - `RegExp.prototype.lastIndex`: 返回一个整数, 表示下一次开始搜索的位置. 属性值可读写, 但是一般在连续搜索的时候才有意义.
  - `RegExp.prototype.source`: 返回正则表达式的字符串形式(不包括反斜杠).

  ```javascript
  // 这里三个修饰符都设置了: i, g, m
  let regexp = /abc/igm;
  
  // 0
  console.log(regexp.lastIndex);
  // abc
  console.log(regexp.source);
  ```


### 3.7.3 实例方法

对于一个字符串, 使用正则表达式时涉及到的操作无非就两种: 看字符串中是否存在匹配正则表达式的子串, 找出字符串中匹配正则表达式的所有子串.

- `RegExp.prototype.test`: 返回布尔值, 表示参数字符串是否匹配当前正则表达式的模式. 如果使用修饰符`g`, 那么每次执行`test`方法都从上次结束的位置开始向后匹配, 可以使用`lastIndex`属性查看匹配后当前正则对象的指向的位置.

  ```javascript
  let str = 'The harder you work, the more luck you have.';
  
  // 没有使用修饰符g, 每次都是从字符串开始处匹配.
  let reg = /the/;
  // true
  console.log(reg.test(str));
  // 0
  console.log(reg.lastIndex);
  // true
  console.log(reg.test(str));
  // 0
  console.log(reg.lastIndex);
  
  
  // 使用了修饰符g, 表示全局搜索, lastIndex的值会改变,
  // 每次都从lastIndex的位置开始搜(可以手动改变这个值)
  let reg1 = /the/g;
  // 正则对象开始位置在0
  console.log(reg1.lastIndex);
  // 返回true,
  console.log(reg1.test(str));
  // 因为使用了修饰符g, 所以lastIndex会保存上次搜索结束后的第一个位置
  console.log(reg1.lastIndex);
  // false 从上次搜索结束后的第一个位置开始向后搜, 找到字符串尾发现没有就结束
  console.log(reg1.test(str));
  // 0 现在停在了0的位置, 表示下次从0这个位置开始搜索
  console.log(reg1.lastIndex);
  ```

- `RegExp.prototype.exec`: 如果匹配成功, 返回一个数组, 数组成员包括匹配的子串, 匹配子串的开始index, 原字符串, groups. 如果不匹配则返回`null`. 注意:

  - 如果不使用`g`修饰符修饰正则表达式表示进行全局匹配, 那么他总是返回参数字符串的第一个匹配子串.
  - 如果使用`g`修饰符修饰正则表达式表示进行全局匹配, 那么正则对象会从上次匹配成功之后的第一个位置开始匹配子串.

  ```javascript
  let str = 'The harder you work, the more luck you have.';
  
  let reg = /the/i;
  let result;
  // [ 'The',
  //    index: 0,
  //    input: 'The harder you work, the more luck you have.',
  //    groups: undefined ]
  result = reg.exec(str);
  console.log(result);
  
  // 不是全局匹配, 每次都是从字符串开始进行匹配, 只匹配第一个.
  // [ 'The',
  //    index: 0,
  //    input: 'The harder you work, the more luck you have.',
  //    groups: undefined ]
  result = reg.exec(str);
  console.log(result);
  
  
  let reg1 = /the/ig;
  let result1;
  
  // reg1.lastIndex为0
  console.log(reg1.lastIndex);
  result1 = reg1.exec(str);
  console.log(result1);
  
  // reg1.lastIndex为3
  console.log(reg1.lastIndex);
  result1 = reg1.exec(str);
  console.log(result1);
  
  // reg1.lastIndex为24
  console.log(reg1.lastIndex);
  result1 = reg1.exec(str);
  // result 为null 重置lastIndex
  console.log(result1);
  
  // lastIndex变为0
  console.log(reg1.lastIndex);
  ```

### 3.7.4 修饰符

- `g`修饰符: 对于一个正则表达式, 如果不使用`g`修饰符, 那么使用这个正则表达式对应的对象来对字符串进行匹配操作的话, 每次都是从字符串的`0`这个位置开始匹配. 但是如果使用`g`修饰符的话, 正则对象的`lastIndex`属性就会改变, 每次对相同或者不同字符串的匹配操作都是根据这个`lastIndex`属性来确定开始位置的.

- `i`操作符: 匹配的时候忽略大小写.

- `m`修饰符表示多行模式(multiline), 会修改`^`和`$`的行为. 默认情况下(即不加`m`修饰符时), `^`和`$`匹配字符串的开始处和结尾处, 加上`m`修饰符以后, `^`和`$`还会匹配行首和行尾, 即`^`和`$`会识别换行符(`\n`).

  ```javascript
  console.log(/world$/.test('hello world\n')); // false
  console.log(/world$/m.test('hello world\n')); // true
  ```

# 四、执行环境及作用域

## 4.1 执行环境

JavaScript中执行环境指的是当前代码执行时所在的上下文, 分为全局环境和局部环境. 如果一段代码是在函数中执行的那么我们称这段代码的执行环境为局部环境, 如果是直接在最外部执行的则称这些代码的执行环境为全局环境.

```javascript
// 处于全局执行环境
let globalVariable = 'global variable';

// 处于全局执行环境
function sayHelloToMale(age) {

    // 处于局部执行环境
    console.log('I am going to say hello.');
    if (age < 18) {
        // 处于局部执行环境
        console.log('Hello boy!');
    } else {
        // 处于局部执行环境
        console.log('Hello man!')
    }
}

// 处于全局执行环境
sayHelloToMale(11);
```



​	全局执行环境是最外围的一个执行环境, 运行代码的宿主环境不同那么表示全局执行环境的变量对象就不同. 假如我们的代码是运行在浏览器环境中, 那么与这个全局执行环境对应的变量对象就是`window`对象, **所有全局变量和函数都是作为`window`对象的属性和方法创建**. 如果我们代码运行在node环境中, 那么这个全局环境变量对象就是`global`对象. 



> 执行环境定义了变量或函数有权访问的其他数据, 决定了他们各自的行为. 每个执行环境都有一个与之关联的**变量对象**, 环境中定义的所有变量和函数都保存在这个对象中.

## 4.2 作用域

​	变量能够在什么范围内被访问, 这里的范围我们就叫做作用域. 在JavaScript中有三种作用域: 全局作用域, 函数作用域, 块级作用域(ES6新增).

- 如果一个变量在整个应用中都能被访问到, 那么我们就说这个变量处于全局作用域中, 是一个全局变量.
- 如果一个变量只能在当前的函数中访问到, 那么我们就说这个变量处于函数作用域中, 是一个局部变量.
- 如果一个变量只能在当前代码块中访问到, 那么我们就说这个变量处于块级作用域中, 是一个局部变量.

### 4.2.1 全局作用域

​	如果一个变量是在全局执行环境中声明的, 那么这个变量处于全局作用域中, 是一个全局变量, 在任何地方都能访问到. 比如我们在浏览器的`console`中直接声明变量, 那么这些变量就是全局变量, 因为声明变量的代码没有被任何函数包裹起来, 是执行在全局环境中的. 或者我们新建一个`test.js`文件, 然后在里面输入前面控制台相同的内容, 那执行这个文件的代码的时候和控制台直接输入是一样的, 都是跑在全局环境中. 

​	需要注意的一点是: 如果一个变量并不是在全局环境中声明的, 而是在局部环境中, 但是声明变量的时候并没有使用变量声明符号,  那这个变量也是全局变量(**这个直接忽略吧, 我们声明变量肯定要带上`var`, `let`或者`const`这些声明符的. 而且一定要带, 不带的话也是自己挖坑**).

```javascript
// f12 打开控制台
// 或者新建一个js文件
// 直接输入 
// 因为是在全局环境中, 所以是全局变量
let globalVariable = 'hello world!';
function sayHello() {
    // 访问的是全局变量
    console.log(globalVariable);
}
// 'hello world!'
sayHello();



// 不使用声明符号, 也是全局变量
function globalVariable() {
    // 没有使用变量声明符号, 虽然他是在globalVariable函数的局部环境中声明的
    // 但是它还是变成了全局变量
    glo = 'hello world!';
}

globalVariable();
// glo是一个全局变量, 这里正常输出值
// 'hello world!'
console.log(glo);
```

### 4.2.2 函数作用域

​	如果一个变量是处于函数作用域中的, 那么这个变量只能在这个函数的内部执行环境中被访问到. 在JavaScript中声明一个变量的时候如果想让他处于函数作用域, 那么使用`var`关键字.

```javascript
function funcScope() {

    if (true) {
        // 在局部环境中使用var声明的变量属于函数作用域
        var variable = 'hello world!';
    }

    // 正常输出
    // 'hello world!'
    console.log(variable);
}

funcScope();
```

### 4.2.3 块级作用域

如果一个变量是属于块级作用域的, 那么只有在当前代码块中`{}`才有能力访问到这个变量, 在JavaScript中使用`let`和`const`声明的变量都属于块级作用域.

```javascript
function funcScope() {

    if (true) {
        // 在局部环境中使用var声明的变量属于函数作用域
        let variable = 'hello world!';
        const constant = 'hello jeb!';

        {
            // 还是在变量定义的代码块中, 可以访问到
            console.log('In sub block, variable: ', variable);
            console.log('In sub block, constant: ', constant);
        }
    }

    // 已经不在变量声明的代码块中, 无法访问到这些块级作用域变量.
    // 编译报错, ReferenceError
    // console.log(variable);
    // console.log(constant);
}

funcScope();
```

# 五、闭包

**闭包是指有权访问另一个函数作用域中的变量的函数**. 创建闭包的方式就是在一个函数内部创建另一个函数, 这个被创建的函数引用创建它的函数内部的变量.

```javascript
function createCounter(initial) {

    return function (value) {
        // 保存了对外面函数的局部变量initial的引用
        return initial + value;
    }
}

let counter = createCounter(10);
// 12
console.log(counter(2));
// 20
console.log(counter(10));
```



闭包的最大用处有两个: 一个是可以读取函数内部的变量, 另一个就是让这些变量始终保持在内存中, 即闭包可以使得它诞生环境一直存在.

```javascript
function createIncrementer(start) {
    return function() {
        return start++;
    }
}

// 通过闭包, start的状态被保留了.
let incrementer = createIncrementer(5);
console.log(incrementer()); // 5
console.log(incrementer()); // 6
console.log(incrementer()); // 7
```

# 六、原型

## 6.1 基本概念, 原理及重要特性

​	原型这块是比较重要的, 理解这些知识点对实际开发的帮助非常大. 关于JavaScript中的原型这部分内容, 需要重点理解原型概念, 原型链原理, 还有JavaScript在对象上搜寻属性的工作方式.

- 原型的概念: JavaScript中构造函数有一个属性`prototype`, 他指向一个对象. 当使用构造函数实例化一个对象的时候, 实例化出来的对象会有一个内部指针指向构造函数的`prototype`属性指向的对象, 这时候我们把这个`prototype`属性指向的对象称为构造函数实例的原型. 

- 原型链原理: 因为原型也是一个对象, 他也会有一个内部指针, 指向实例化它的构造函数的`prototype`属性指向的原型对象, 这个原型对象又有一个内部指针指向另一个原型对象, 层层递进直到`Object.prototype`, 而`Object.prototype`指向的对象的原型对象为`null`, 到这里就不会再有原型的指向了, 我们把这个链中涉及到的原型对象称为原型链.
- 在对象上搜寻属性的工作方式: 当我们调用某个对象的属性或方法时, 如`person.sayHello()`, JavaScript引擎会先在查找对象自身的属性, 如果找不到, 那么会到这个对象的原型上面去找, 如果还是找不到就继续到原型的原型上去找, 沿着原型链一直往上走, 直到最顶层的`Object.prototype`, 如果还是没有找到就返回`undefined`.

```javascript
let MyArray = function () { };

MyArray.prototype = new Array();
MyArray.prototype.constructor = MyArray;

let myArray = new MyArray();
myArray.push(1, 2, 3);
console.log(myArray.length);

// 上面的原型链为:
// 1. myArray的原型为MyArray.prototype, MyArray.prototype = new Array(), 因此第一个原型指向:
// myArray._pro -> new Array();
// 2. 因为new Array()出来的对象有一个内部指针指向Array.prototype: (new Array())._pro -> Array.prototype, 所以:
// myArray._pro -> new Array(), (new Array())._pro -> Array.prototype
// 3. Array.prototype也是一个对象, 这个对象也有原型对象, 它有一个内部指针指向Object.prototype: Array.prototype._pro -> Object.prototype
// myArray._pro -> new Array(), (new Array())._pro -> Array.prototype, Array.prototype._pro -> Object.prototype
// 4. Object.prototype原型也是一个对象, 但是这个原型对象的内部指针指向了null: Object.prototype._pro -> null, 所以原型链终止.

// 因此整个原型链为
// myArray._pro -> new Array(), (new Array())._pro -> Array.prototype, Array.prototype._pro -> Object.prototype, Object.prototype._pro -> null
// 当在myArray上调用一个属性或者方法的时候, 在整个原型链上的搜索顺序为:
// 现在myArray这个对象上找, 如果没有在myArray._pro上找, 也就是new Array()上, 沿着链走, 当走到最尾发现原型对象为null的时候就停止搜索.
```

## 6.2 constructor属性

`prototype`对象有一个`constructor`属性, 他默认指向`prototype`对象所在的构造函数:

```javascript
function Person() {}
// true
console.log(Person.prototype.constructor === Person);
```



对于`constructor`属性, 有一下几个需要注意的:

- 可以使用`constructor`属性从一个实例对象创建另一个实例.

  ```javascript
  let Person = function () {
      
  };
  
  let alpha = new Person();
  let jeb = new alpha.constructor();
  ```

- 如果手动修改原型对象, 那么一般会同时修改`constructor`属性, 防止引用的时候出错.

  ```javascript
  function Person(name) {
    this.name = name;
  }
  
  Person.prototype.constructor === Person // true
  
  Person.prototype = {
    method: function () {}
  };
  
  // false
  console.log(Person.prototype.constructor === Person);
  // true
  console.log(Person.prototype.constructor === Object);
  ```

# 七、this关键字

## 7.1 this详解

在JavaScript中`this`关键字是非常重要的, 而且也是非常容易搞混的. 如果没有深入去学习, 那么在实际开发的时候一不小心就`undefined`, 一不小心就`ReferenceError`, 很容易掉坑. 要深入理解`tihs`的作用, 那么需要总结`this`能在哪些地方使用:

- 在全局环境中使用`this`: 如果是浏览器环境, 那么他指向的是`window`对象. 如果是在node环境, 那么他指向的是一个新的对象(注意并不是`global`对象).

  ```javascript
  this.name = 'alpha';
  
  // 浏览器环境: window
  // alpha
  // console.log(this.name);
  
  // node环境: 一个新的对象
  // { name: 'alpha' }
  console.log(this);
  ```

- 在函数中使用`this`:

  - 当这个函数被当做构造函数来用时(使用`new`运算符时), `this`指向的是即将在构造函数中返回的那个对象.

    ```javascript
    function Person(name) {
    
        this.name = name;
        this.sayName = function () {
            console.log(this.name);
        }
    }
    
    let person = new Person('jeb');
    // jeb
    console.log(person.name);
    // jeb
    person.sayName();
    ```

  - 当一个函数被当做对象的方法来使用时(`obj.method`), 函数里面的`this`就指向调用他的`obj`对象. 但是**注意方法是可以被赋值的, 如果一个方法被赋给了一个变量, 那这时候`this`的指向就改变了**.

    ```javascript
    function Person(name) {
    
        this.name = name;
        this.sayName = function () {
            console.log(this.name);
        }
    }
    
    let person = new Person('jeb');
    let sayNameGlobal = person.sayName;
    // 因为在全局环境执行这个函数, this并不是指向person对象
    // undefined
    sayNameGlobal();
    
    let newPerson = {name: 'alpha'};
    newPerson.sayName = person.sayName;
    // 在newPerson上调用这个函数, 所以现在在sayName里面的this指向的是newPerson这个对象
    // alpha
    newPerson.sayName();
    ```

- 在多层嵌套的函数中使用`this`: 

  ```javascript
  function Person(name) {
  
      this.name = name;
      this.sayNameAndSayNameAgain = function () {
  
          // this指向调用这个方法的对象
          console.log(this.name);
  
          (function () {
  
              // this指向全局对象
              // undefined
              // console.log(this.name);
              console.log(this);
  
              (function() {
  
                  // this指向全局对象
                  // undefined
                  // console.log(this.name);
                  console.log(this);
              }());
          }());
      }
  }
  
  let jeb = new Person('jeb');
  jeb.sayNameAndSayNameAgain();
  ```

- 在ES6类的方法上使用`this`: 看`ES6.md`.

## 7.2 绑定this方法

`this`可以动态进行切换, 但是提供灵活的同时给编程带来了困难和模糊. JavaScript提供了`call`, `apply`, `bind`三个方法, 来切换/固定`this`的指向.

```javascript
// call
let n = 123;
let obj = { n: 456 };

function a(var1, var2) {
    console.log(this.n);
}

a.call(obj, 1, 2); // 456


// apply: 和call一样, 但是参数的接收方式为数组.
// a.apply(obj, [1, 2]);

// bind: 用于将函数体内的this绑定到某个对象, 然后返回一个新函数.
let d = new Date();
d.getTime();

let print = d.getTime.bind({});
// 执行绑定this之后的新函数.
print();
```

# 八、错误处理

## 8.1 错误类型

​	JavaScript中`Error`是最基本的错误类型, 在它的基础上, 还定义了其他几种错误类型. 在开发中可能会偶尔看到运行环境抛出的错误, 如果对它们不熟悉则很多时候是一头雾水.

- `SyntaxError`: 表示解析代码时发生语法错误.

  ```javascript
  // 变量名错误
  var 1a;
  // Uncaught SyntaxError: Invalid or unexpected token
  
  // 缺少括号
  console.log 'hello');
  // Uncaught SyntaxError: Unexpected string
  ```

- `ReferenceError`: 引用一个不存在的变量.

  ```javascript
  // name根本没有定义就引用了
  // ReferenceError: name is not defined
  console.log(name);
  ```

- `RangeError`: 值超出了有效范围: 为数组赋负数值等.

- `TypeError`: 变量或参数不是预期类型时发生的错误, 比如对字符串, 布尔值, 数值等原始类型使用`new`命令. `let obj = {}; obj.unknownMethod()` 这样也会抛出这个异常. 因为`obj.unknownMethod`的值是`undefined`, 而不是一个函数.

- `URIError`: uri相关函数的参数不正确时抛出的错误, 主要涉及`encodeURI()`, `decodeURI()`, `encodeURIComponent()`, `decodeURIComponent()`, `escape()`和`unescape()`这六个函数.

- `EvalError`: `eval`函数没有被正确执行时抛出的错误.

## 8.2 try...catch和throw

和Java一样, 在JavaScript中也是使用`try...catch`来捕获错误, 使用`throw`来抛出错误的.

```javascript
try {
    let obj = {};
    // 这里会由执行引擎抛出异常
    obj.unknownMethod();
} catch (e) {
    // 这里捕获了异常并输出
    console.log(e.message);
    
    // 这里使用throw抛出我们自己的异常
    throw new Error('error throw by myself.');
}
```

## 8.3 自定义错误

JavaScript中是使用`throw`来抛出错误的, 他不像Java那样抛出的异常必须是继承自某个类, 而是可以`throw`任意类型, 但是强迫症晚期是不会抛出任意类型的. 约定抛出的错误都继承自`Error`.

```javascript
function EntityNotFoundException(message) {
    this.message = message;
}

EntityNotFoundException.prototype = new Error();
EntityNotFoundException.prototype.constructor = EntityNotFoundException;

try {
    throw new EntityNotFoundException('Id为5的实体找不到.');
} catch (e) {

    console.log(e.message);
    console.log(e);
}
```

# 九、其他

## 9.1 官方文档地址

一般查看API或者看一些相关的guide都是在MDN上看, [地址](https://developer.mozilla.org/en-US/docs/Web/JavaScript). 在左边bar的References里面能够查看许多API.

## 9.2 console

在代码中我们可以使用`console`对象来向控制台输出信息, 常用的是`console.log()`. 在这个api中可以传入多个参数, 参数参数之间用逗号隔开, 当参数是对象的时候, 他会将对象的属性也打印出来. 当接盘侠的时候这是个非常有用的东西. 像一个Node项目如果不支持IDE的debug的话只能一点一点地`console.log()`.

```javascript
let jeb = {
    name: 'jeb',
    age: 22
};

// the detail of jeb:  { name: 'jeb', age: 22 }
console.log('the detail of jeb: ', jeb);

//如果使用+号, 那么直接调用jeb对象的toString() 方法, 这个方法定义在Object.prototype上
//the detail of jeb: [object Object]
console.log('the detail of jeb: ' + jeb);
```
