###变量提升

​		**JavaScript 引擎的工作方式是，先解析代码，获取所有被声明的变量，然后再一行一行地运行**。这造成的结果，就是所有的变量的声明语句，都会被提升到代码的头部，这就叫做变量提升（hoisting）。

```
console.log(a);
var a = 1;
```

上面代码首先使用`console.log`方法，在控制台（console）显示变量`a`的值。这时变量`a`还没有声明和赋值，所以这是一种错误的做法，但是实际上不会报错。因为存在变量提升，真正运行的是下面的代码。

```
var a;
console.log(a);
a = 1;
```

最后的结果是显示`undefined`，表示变量`a`已声明，但还未赋值。

###函数提升

​		JavaScript 引擎将函数名视同变量名，所以采用`function`命令声明函数时，整个函数会像变量声明一样，被提升到代码头部。所以，下面的代码不会报错。

```
f();

function f() {}
```

表面上，上面代码好像在声明之前就调用了函数`f`。但是实际上，由于“变量提升”，函数`f`被提升到了代码头部，也就是在调用之前已经声明了。但是，如果采用赋值语句定义函数，JavaScript 就会报错。

```
f();
var f = function (){};
// TypeError: undefined is not a function
```

上面的代码等同于下面的形式。

```
var f;
f();
f = function () {};
```

上面代码第二行，调用`f`的时候，`f`只是被声明了，还没有被赋值，等于`undefined`，所以会报错。

注意，如果像下面例子那样，采用`function`命令和`var`赋值语句声明同一个函数，由于存在函数提升，最后会采用`var`赋值语句的定义。

```
var f = function () {
  console.log('1');
}

function f() {
  console.log('2');
}

f() // 1
```

上面例子中，表面上后面声明的函数`f`，应该覆盖前面的`var`赋值语句，但是由于存在函数提升，实际上正好反过来。

###函数中的变量提升

与全局作用域一样，函数作用域内部也会产生“变量提升”现象。`var`命令声明的变量，不管在什么位置，变量声明都会被提升到函数体的头部。

```
function foo(x) {
  if (x > 100) {
    var tmp = x - 100;
  }
}

// 等同于
function foo(x) {
  var tmp;
  if (x > 100) {
    tmp = x - 100;
  };
}
```

###布尔值

如果 js 预期某个位置应该是布尔值，会将该位置上现有的值自动转为布尔值。转换规则是除了下面六个值被转为`false`，其他值都视为`true`。注意空字符串是false，空对象和数组是true

- `undefined`
- `null`
- `false`
- `0`
- `NaN`
- `""`或`''`（空字符串）

### 整数和浮点数

JavaScript 内部，所有数字都是以64位浮点数形式储存，即使整数也是如此。所以，`1`与`1.0`是相同的，是同一个数。

```
1 === 1.0 // true
```

这就是说，JavaScript 语言的底层根本没有整数，所有数字都是小数（64位浮点数）。容易造成混淆的是，某些运算只有整数才能完成，此时 JavaScript 会自动把64位浮点数，转成32位整数，然后再进行运算。

**由于浮点数不是精确的值，所以涉及小数的比较和运算要特别小心。**

```js
0.1 + 0.2 === 0.3
// false

0.3 / 0.1
// 2.9999999999999996

(0.3 - 0.2) === (0.2 - 0.1)
// false
```

### 数值范围

如果一个数大于等于2的1024次方，那么就会发生“正向溢出”，即 JavaScript 无法表示这么大的数，这时就会返回`Infinity`。

```
Math.pow(2, 1024) // Infinity
```

如果一个数小于等于2的-1075次方（指数部分最小值-1023，再加上小数部分的52位），那么就会发生为“负向溢出”，即 JavaScript 无法表示这么小的数，这时会直接返回0。

```js
Math.pow(2, -1075) // 0
```

###正0负0

JavaScript 的64位浮点数之中，有一个二进制位是符号位。这意味着，任何一个数都有一个对应的负值，就连`0`也不例外。JavaScript 内部实际上存在2个`0`：一个是`+0`，一个是`-0`，区别就是64位浮点数表示法的符号位不同。它们是等价的。

```js
-0 === +0 // true
0 === -0 // true
0 === +0 // true
```

几乎所有场合，正零和负零都会被当作正常的`0`。

```js
+0 // 0
-0 // 0
(-0).toString() // '0'
(+0).toString() // '0'
```

**唯一有区别的场合是，`+0`或`-0`当作分母，返回的值是不相等的。**

```
(1 / +0) === (1 / -0) // false
```

上面的代码之所以出现这样结果，是因为除以正零得到`+Infinity`，除以负零得到`-Infinity`，这两者是不相等的。

###NAN

**（1）含义**

`NaN`是 JavaScript 的特殊值，表示“非数字”（Not a Number），主要出现在将字符串解析成数字出错的场合。

```
5 - 'x' // NaN
```

上面代码运行时，会自动将字符串`x`转为数值，但是由于`x`不是数值，所以最后得到结果为`NaN`，表示它是“非数字”（`NaN`）。

另外，一些数学函数的运算结果会出现`NaN`。

```
Math.acos(2) // NaN
Math.log(-1) // NaN
Math.sqrt(-1) // NaN
```

`0`除以`0`也会得到`NaN`。

```
0 / 0 // NaN
```

需要注意的是，`NaN`不是独立的数据类型，而是一个特殊数值，它的数据类型依然属于`Number`，使用`typeof`运算符可以看得很清楚。

```
typeof NaN // 'number'
```

**（2）运算规则**

`NaN`不等于任何值，包括它本身。

```js
NaN === NaN // false
```

数组的`indexOf`方法内部使用的是严格相等运算符，所以该方法对`NaN`不成立。

```js
[NaN].indexOf(NaN) // -1
```

`NaN`在布尔运算时被当作`false`。

```js
Boolean(NaN) // false
```

`NaN`与任何数（包括它自己）的运算，得到的都是`NaN`。

```js
NaN + 32 // NaN
NaN - 32 // NaN
NaN * 32 // NaN
NaN / 32 // NaN
```

###infinity

**（1）含义**

`Infinity`表示“无穷”，用来表示两种场景。一种是一个正的数值太大，或一个负的数值太小，无法表示；另一种是非0数值除以0，得到`Infinity`。

```
// 场景一
Math.pow(2, 1024)
// Infinity

// 场景二
0 / 0 // NaN
1 / 0 // Infinity
```

上面代码中，第一个场景是一个表达式的计算结果太大，超出了能够表示的范围，因此返回`Infinity`。第二个场景是`0`除以`0`会得到`NaN`，而非0数值除以`0`，会返回`Infinity`。

`Infinity`有正负之分，`Infinity`表示正的无穷，`-Infinity`表示负的无穷。

```
Infinity === -Infinity // false

1 / -0 // -Infinity
-1 / -0 // Infinity
```

上面代码中，非零正数除以`-0`，会得到`-Infinity`，负数除以`-0`，会得到`Infinity`。

由于数值正向溢出（overflow）、负向溢出（underflow）和被`0`除，JavaScript 都不报错，所以单纯的数学运算几乎没有可能抛出错误。

`Infinity`大于一切数值（除了`NaN`），`-Infinity`小于一切数值（除了`NaN`）。

```
Infinity > 1000 // true
-Infinity < -1000 // true
```

`Infinity`与`NaN`比较，总是返回`false`。

```
Infinity > NaN // false
-Infinity > NaN // false

Infinity < NaN // false
-Infinity < NaN // false
```

**（2）运算规则**

`Infinity`的四则运算，**符合无穷的数学计算规则**。

```
5 * Infinity // Infinity
5 - Infinity // -Infinity
Infinity / 5 // Infinity
5 / Infinity // 0
```

0乘以`Infinity`，返回`NaN`；0除以`Infinity`，返回`0`；`Infinity`除以0，返回`Infinity`。

```
0 * Infinity // NaN
0 / Infinity // 0
Infinity / 0 // Infinity
```

`Infinity`加上或乘以`Infinity`，返回的还是`Infinity`。

```
Infinity + Infinity // Infinity
Infinity * Infinity // Infinity
```

`Infinity`减去或除以`Infinity`，得到`NaN`。

```
Infinity - Infinity // NaN
Infinity / Infinity // NaN
```

`Infinity`与`null`计算时，`null`会转成0，等同于与`0`的计算。

```
null * Infinity // NaN
null / Infinity // 0
Infinity / null // Infinity
```

`Infinity`与`undefined`计算，返回的都是`NaN`。

```
undefined + Infinity // NaN
undefined - Infinity // NaN
undefined * Infinity // NaN
undefined / Infinity // NaN
Infinity / undefined // NaN
```

### parseInt()

全局方法：将字符串转换为整数

需要注意：

- 会自动去空格

- 若类型不是字符串，则会先转换成字符串，再转换成整数，例如

  ```js
  parseInt(1.23) // 1
  // 等同于
  parseInt('1.23') // 1
  ```

- 字符串转为整数的时候，是一个个字符依次转换，如果遇到不能转为数字的字符，就不再进行下去，返回已经转好的部分。例如

  ```js
  parseInt('8a') // 8
  parseInt('12**') // 12
  parseInt('12.34') // 12
  parseInt('15e2') // 15
  parseInt('15px') // 15
  ```

- 如果字符串的第一个字符不能转化为数字（后面跟着数字的正负号除外），返回`NaN`。所以，**`parseInt`的返回值只有两种可能，要么是一个十进制整数，要么是`NaN`。**

- 如果字符串以`0x`或`0X`开头，`parseInt`会将其按照十六进制数解析；如果字符串以`0`开头，将其按照10进制解析。

- `parseInt`不能识别科学计数法，会按照字符串进行识别，返回错误结果。

- `parseInt`方法还可以接受第二个参数（2到36之间），表示被解析的值的进制，返回该值对应的十进制数，这个用法还有很多细节，因为不太常用，暂时不做记录。

  ### parseFloat()

  全局方法：将字符串转换为浮点数。

  需要注意：

  - 可以识别科学计数法

  - 如果字符串包含不能转为浮点数的字符，则不再进行往后转换，返回已经转好的部分。

  - 可以自动去空格

  - 如果参数不是字符串，则会先转为字符串再转换。

  - 类型不是字符串，会先转换成字符串，再转换成浮点数

    ```js
    parseFloat([1.23]) // 1.23
    // 等同于
    parseFloat(String([1.23])) // 1.23
    ```

  - 如果字符串的第一个字符不能转化为浮点数，则返回`NaN`。尤其值得注意，**`parseFloat`会将空字符串转为`NaN`**。`Number()`函数会将空字符串转换成0，注意区分。

  

  

###isNaN()

  全局方法：判断一个值是否为NaN

  需要注意：

  - `isNaN`只对数值有效，如果传入其他值，会被先转成数值。比如，传入字符串的时候，字符串会被先转成`NaN`，所以最后返回`true`，这一点要特别引起注意。也就是说，`isNaN`为`true`的值，有可能不是`NaN`，而是一个字符串。

  - 对象和数组，`isNaN`也返回`true`。但是，对于空数组和只有一个数值成员的数组，`isNaN`返回`false`，原因是这些数组能被`Number`函数转成数值。因此，使用`isNaN`之前，最好判断一下数据类型。

    ```js
    function myIsNaN(value) {
      return typeof value === 'number' && isNaN(value);
    }
    ```

    判断`NaN`更可靠的方法是，利用`NaN`为唯一不等于自身的值的这个特点，进行判断。

    ```js
    function myIsNaN(value) {
      return value !== value;
    }
    ```

### isFinite()

  `isFinite`方法返回一个布尔值，表示某个值是否为正常的数值。

  ```js
  isFinite(Infinity) // false
  isFinite(-Infinity) // false
  isFinite(NaN) // false
  isFinite(undefined) // false
  isFinite(null) // true
  isFinite(-1) // true
  ```

  除了`Infinity`、`-Infinity`、`NaN`和`undefined`这几个值会返回`false`，`isFinite`对于其他的数值都会返回`true`。

  

### 字符串

-  单引号字符串的内部，可以使用双引号。双引号字符串的内部，可以使用单引号。

- 如果要在单引号字符串的内部，使用单引号，就必须在内部的单引号前面加上反斜杠，用来转义。双引号字符串内部使用双引号，也是如此。

  ```js
  'Did she say \'Hello\'?'
  // "Did she say 'Hello'?"
  
  "Did she say \"Hello\"?"
  // "Did she say "Hello"?"
  ```

- 字符串默认只能写在一行内，分成多行将会报错。如果长字符串必须分成多行，可以在每一行的尾部使用反斜杠。此外，连接运算符（`+`）可以连接多个单行字符串，将长字符串拆成多行书写，输出的时候也是单行。

```js
var longString = 'Long '
  + 'long '
  + 'long '
  + 'string';
```



### 转义符（\）

需要用反斜杠转义的特殊字符，主要有下面这些。

- `\0` ：null（`\u0000`）
- `\b` ：后退键（`\u0008`）
- `\f` ：换页符（`\u000C`）
- `\n` ：换行符（`\u000A`）
- `\r` ：回车键（`\u000D`）
- `\t` ：制表符（`\u0009`）
- `\v` ：垂直制表符（`\u000B`）
- `\'` ：单引号（`\u0027`）
- `\"` ：双引号（`\u0022`）
- `\\` ：反斜杠（`\u005C`）

### 字符串和数组

字符串可以被看作字符数组，可以用下标的方式访问其中的字符，如果下标超过字符串长度或者方括号中不是数字则返回`undefined`.不可以用这种方法修改字符串中的字符，例如

```js
var s = 'hello';

delete s[0];
s // "hello"

s[1] = 'a';
s // "hello"

s[5] = '!';
s // "hello"
```

### Base64转码

有时，文本里面包含一些不可打印的符号，比如 ASCII 码0到31的符号都无法打印出来，这时可以使用 Base64 编码，将它们转成可以打印的字符。另一个场景是，有时需要以文本格式传递二进制数据，那么也可以使用 Base64 编码。

所谓 Base64 就是一种编码方法，可以将任意值转成 0～9、A～Z、a-z、`+`和`/`这64个字符组成的可打印字符。使用它的主要目的，不是为了加密，而是为了不出现特殊字符，简化程序的处理。

JavaScript 原生提供两个 Base64 相关的方法。

- `btoa()`：任意值转为 Base64 编码
- `atob()`：Base64 编码转为原来的值

```
var string = 'Hello World!';
btoa(string) // "SGVsbG8gV29ybGQh"
atob('SGVsbG8gV29ybGQh') // "Hello World!"
```

注意，这两个方法不适合非 ASCII 码的字符，会报错。

```
btoa('你好') // 报错
```

要将非 ASCII 码字符转为 Base64 编码，必须中间插入一个转码环节，再使用这两个方法。

```
function b64Encode(str) {
  return btoa(encodeURIComponent(str));
}

function b64Decode(str) {
  return decodeURIComponent(atob(str));
}

b64Encode('你好') // "JUU0JUJEJUEwJUU1JUE1JUJE"
b64Decode('JUU0JUJEJUEwJUU1JUE1JUJE') // "你好"
```

### 删除对象属性

`delete`命令用于删除对象的属性，删除成功后返回`true`。**注意，删除一个不存在的属性，`delete`不报错，而且返回`true`。**只有一种情况，`delete`命令会返回`false`，那就是该属性存在，且不得删除。

```js
var obj = { p: 1 };
Object.keys(obj) // ["p"]

delete obj.p // true
obj.p // undefined
Object.keys(obj) // []
```

### 判断对象是否存在某个属性- in运算符

`in`运算符用于检查对象是否包含某个属性（注意，检查的是键名，不是键值），如果包含就返回`true`，否则返回`false`。它的左边是一个字符串，表示属性名，右边是一个对象。

```js
var obj = { p: 1 };
'p' in obj // true
'toString' in obj // true
```

`in`运算符的一个问题是，它不能识别哪些属性是对象自身的，哪些属性是继承的。就像上面代码中，对象`obj`本身并没有`toString`属性，但是`in`运算符会返回`true`，因为这个属性是继承的。

这时，可以使用对象的`hasOwnProperty`方法判断一下，是否为对象自身的属性。

```js
var obj = {};
if ('toString' in obj) {
  console.log(obj.hasOwnProperty('toString')) // false
}
```

### 函数参数传递方式

函数参数如果是原始类型的值（数值、字符串、布尔值），传递方式是传值传递（passes by value）。这意味着，在函数体内修改参数值，不会影响到函数外部。

```
var p = 2;

function f(p) {
  p = 3;
}
f(p);

p // 2
```

上面代码中，变量`p`是一个原始类型的值，传入函数`f`的方式是传值传递。因此，在函数内部，`p`的值是原始值的拷贝，无论怎么修改，都不会影响到原始值。

但是，如果函数参数是复合类型的值（数组、对象、其他函数），传递方式是传址传递（pass by reference）。也就是说，传入函数的原始值的地址，因此在函数内部修改参数，将会影响到原始值。

```
var obj = { p: 1 };

function f(o) {
  o.p = 2;
}
f(obj);

obj.p // 2
```

上面代码中，传入函数`f`的是参数对象`obj`的地址。因此，在函数内部修改`obj`的属性`p`，会影响到原始值。

注意，如果函数内部修改的，不是参数对象的某个属性，而是替换掉整个参数，这时不会影响到原始值。

```
var obj = [1, 2, 3];

function f(o) {
  o = [2, 3, 4];
}
f(obj);

obj // [1, 2, 3]
```

上面代码中，在函数`f()`内部，参数对象`obj`被整个替换成另一个值。这时不会影响到原始值。这是因为，形式参数（`o`）的值实际是参数`obj`的地址，重新对`o`赋值导致`o`指向另一个地址，保存在原地址上的值当然不受影响。

### 函数的arguements对象

**（1）定义**

由于 JavaScript 允许函数有不定数目的参数，所以需要一种机制，可以在函数体内部读取所有参数。这就是`arguments`对象的由来。

`arguments`对象包含了函数运行时的所有参数，`arguments[0]`就是第一个参数，`arguments[1]`就是第二个参数，以此类推。这个对象只有在函数体内部，才可以使用。

```
var f = function (one) {
  console.log(arguments[0]);
  console.log(arguments[1]);
  console.log(arguments[2]);
}

f(1, 2, 3)
// 1
// 2
// 3
```

正常模式下，`arguments`对象可以在运行时修改。

```
var f = function(a, b) {
  arguments[0] = 3;
  arguments[1] = 2;
  return a + b;
}

f(1, 1) // 5
```

上面代码中，函数`f()`调用时传入的参数，在函数内部被修改成`3`和`2`。

**严格模式下，`arguments`对象与函数参数不具有联动关系。也就是说，修改`arguments`对象不会影响到实际的函数参数。**

```js
var f = function(a, b) {
  'use strict'; // 开启严格模式
  arguments[0] = 3;
  arguments[1] = 2;
  return a + b;
}

f(1, 1) // 2
```

上面代码中，函数体内是严格模式，这时修改`arguments`对象，不会影响到真实参数`a`和`b`。

通过`arguments`对象的`length`属性，可以判断函数调用时到底带几个参数。

```js
function f() {
  return arguments.length;
}

f(1, 2, 3) // 3
f(1) // 1
f() // 0
```

**（2）与数组的关系**

需要注意的是，虽然`arguments`很像数组，但它是一个对象。数组专有的方法（比如`slice`和`forEach`），不能在`arguments`对象上直接使用。

如果要让`arguments`对象使用数组方法，真正的解决方法是将`arguments`转为真正的数组。下面是两种常用的转换方法：`slice`方法和逐一填入新数组。

```js
var args = Array.prototype.slice.call(arguments);

// 或者
var args = [];
for (var i = 0; i < arguments.length; i++) {
  args.push(arguments[i]);
}
```

**（3）callee 属性**

`arguments`对象带有一个`callee`属性，返回它所对应的原函数。

```js
var f = function () {
  console.log(arguments.callee === f);
}

f() // true
```

可以通过`arguments.callee`，达到调用函数自身的目的。这个属性在严格模式里面是禁用的，因此不建议使用。

### JS闭包

闭包（closure）是 JavaScript 语言的一个难点，也是它的特色，很多高级应用都要依靠闭包实现。

理解闭包，首先必须理解变量作用域。前面提到，JavaScript 有两种作用域：全局作用域和函数作用域。函数内部可以直接读取全局变量。

```
var n = 999;

function f1() {
  console.log(n);
}
f1() // 999
```

上面代码中，函数`f1`可以读取全局变量`n`。

但是，正常情况下，函数外部无法读取函数内部声明的变量。

```
function f1() {
  var n = 999;
}

console.log(n)
// Uncaught ReferenceError: n is not defined(
```

上面代码中，函数`f1`内部声明的变量`n`，函数外是无法读取的。

如果出于种种原因，需要得到函数内的局部变量。正常情况下，这是办不到的，只有通过变通方法才能实现。那就是在函数的内部，再定义一个函数。

```
function f1() {
  var n = 999;
  function f2() {
　　console.log(n); // 999
  }
}
```

上面代码中，函数`f2`就在函数`f1`内部，这时`f1`内部的所有局部变量，对`f2`都是可见的。但是反过来就不行，`f2`内部的局部变量，对`f1`就是不可见的。这就是 JavaScript 语言特有的"链式作用域"结构（chain scope），子对象会一级一级地向上寻找所有父对象的变量。所以，父对象的所有变量，对子对象都是可见的，反之则不成立。

既然`f2`可以读取`f1`的局部变量，那么只要把`f2`作为返回值，我们不就可以在`f1`外部读取它的内部变量了吗！

```
function f1() {
  var n = 999;
  function f2() {
    console.log(n);
  }
  return f2;
}

var result = f1();
result(); // 999
```

上面代码中，函数`f1`的返回值就是函数`f2`，由于`f2`可以读取`f1`的内部变量，所以就可以在外部获得`f1`的内部变量了。

闭包就是函数`f2`，即能够读取其他函数内部变量的函数。由于在 JavaScript 语言中，只有函数内部的子函数才能读取内部变量，因此可以把闭包简单理解成“定义在一个函数内部的函数”。闭包最大的特点，就是它可以“记住”诞生的环境，比如`f2`记住了它诞生的环境`f1`，所以从`f2`可以得到`f1`的内部变量。在本质上，闭包就是将函数内部和函数外部连接起来的一座桥梁。

闭包的最大用处有两个，一个是可以读取外层函数内部的变量，另一个就是让这些变量始终保持在内存中，即闭包可以使得它诞生环境一直存在。请看下面的例子，闭包使得内部变量记住上一次调用时的运算结果。

```js
function createIncrementor(start) {
  return function () {
    return start++;
  };
}

var inc = createIncrementor(5);

inc() // 5
inc() // 6
inc() // 7
```

上面代码中，`start`是函数`createIncrementor`的内部变量。通过闭包，`start`的状态被保留了，每一次调用都是在上一次调用的基础上进行计算。从中可以看到，闭包`inc`使得函数`createIncrementor`的内部环境，一直存在。所以，闭包可以看作是函数内部作用域的一个接口。

为什么闭包能够返回外层函数的内部变量？原因是闭包（上例的`inc`）用到了外层变量（`start`），导致外层函数（`createIncrementor`）不能从内存释放。只要闭包没有被垃圾回收机制清除，外层函数提供的运行环境也不会被清除，它的内部变量就始终保存着当前值，供闭包读取。

闭包的另一个用处，是封装对象的私有属性和私有方法。

```js
function Person(name) {
  var _age;
  function setAge(n) {
    _age = n;
  }
  function getAge() {
    return _age;
  }

  return {
    name: name,
    getAge: getAge,
    setAge: setAge
  };
}

var p1 = Person('张三');
p1.setAge(25);
p1.getAge() // 25
```

上面代码中，函数`Person`的内部变量`_age`，通过闭包`getAge`和`setAge`，变成了返回对象`p1`的私有变量。

注意，外层函数每次运行，都会生成一个新的闭包，而这个闭包又会保留外层函数的内部变量，所以内存消耗很大。因此不能滥用闭包，否则会造成网页的性能问题。

### 立即调用的函数表达式

```
(function(){ /* code */ }());
// 或者
(function(){ /* code */ })();
```

上面两种写法都是以圆括号开头，引擎就会认为后面跟的是一个表达式，而不是函数定义语句，所以就避免了错误。这就叫做“立即调用的函数表达式”（Immediately-Invoked Function Expression），简称 IIFE。即在函数定义之后加一对小括号，然后整体再用小括号包起来，意思为立即调用该函数。具体细节用到时查询，这里不做详细说明。

### eval命令

`eval`命令接受一个字符串作为参数，并将这个字符串当作语句执行。这个语句需要有自身独立的意义，不能和外部语句配合使用。如果接受参数不是字符串，会原样返回。`eval`没有自己的作用域，可能会造成安全问题，并且不利于引擎执行速度，慎用。

### 数组的本质

本质上，数组属于一种特殊的对象。`typeof`运算符会返回数组的类型是`object`。

```js
typeof [1, 2, 3] // "object"
```

上面代码表明，`typeof`运算符认为数组的类型就是对象。

数组的特殊性体现在，它的键名是按次序排列的一组整数（0，1，2...）。

```js
var arr = ['a', 'b', 'c'];

Object.keys(arr)
// ["0", "1", "2"]
```

上面代码中，`Object.keys`方法返回数组的所有键名。可以看到数组的键名就是整数0、1、2。

由于数组成员的键名是固定的（默认总是0、1、2...），因此数组不用为每个元素指定键名，而对象的每个成员都必须指定键名。JavaScript 语言规定，对象的键名一律为字符串，所以，数组的键名其实也是字符串。之所以可以用数值读取，是因为非字符串的键名会被转为字符串。

```js
var arr = ['a', 'b', 'c'];
arr['0'] // 'a'
arr[0] // 'a'
```

上面代码分别用数值和字符串作为键名，结果都能读取数组。原因是数值键名被自动转为了字符串。

注意，这点在赋值时也成立。一个值总是先转成字符串，再作为键名进行赋值。

```
var a = [];
a[1.00] = 6;
a[1] // 6
```

上面代码中，由于`1.00`转成字符串是`1`，所以通过数字键`1`可以读取值。

对象有两种读取成员的方法：点结构（`object.key`）和方括号结构（`object[key]`）。但是，**对于数值的键名，不能使用点结构。**

```
var arr = [1, 2, 3];
arr.0 // SyntaxError
```

上面代码中，`arr.0`的写法不合法，因为单独的数值不能作为标识符（identifier）。所以，数组成员只能用方括号`arr[0]`表示（方括号是运算符，可以接受数值）。

### 手动设置数组的length属性

`length`属性是可写的。如果人为设置一个小于当前成员个数的值，该数组的成员数量会自动减少到`length`设置的值。

```js
var arr = [ 'a', 'b', 'c' ];
arr.length // 3

arr.length = 2;
arr // ["a", "b"]
```

上面代码表示，当数组的`length`属性设为2（即最大的整数键只能是1）那么整数键2（值为`c`）就已经不在数组中了，被自动删除了。

清空数组的一个有效方法，就是将`length`属性设为0。

```js
var arr = [ 'a', 'b', 'c' ];
arr.length = 0;
arr // []
```

如果人为设置`length`大于当前元素个数，则数组的成员数量会增加到这个值，新增的位置都是空位。

```js
var a = ['a'];

a.length = 3;
a[1] // undefined
```

上面代码表示，当`length`属性设为大于数组个数时，读取新增的位置都会返回`undefined`。

如果人为设置`length`为不合法的值，JavaScript 会报错。

```js
// 设置负值
[].length = -1
// RangeError: Invalid array length

// 数组元素个数大于等于2的32次方
[].length = Math.pow(2, 32)
// RangeError: Invalid array length

// 设置字符串
[].length = 'abc'
// RangeError: Invalid array length
```

值得注意的是，由于数组本质上是一种对象，所以可以为数组添加属性，但是这不影响`length`属性的值。

```js
var a = [];

a['p'] = 'abc';
a.length // 0

a[2.1] = 'abc';
a.length // 0
```

上面代码将数组的键分别设为字符串和小数，结果都不影响`length`属性。因为，`length`属性的值就是等于最大的数字键加1，而这个数组没有整数键，所以`length`属性保持为`0`。

###数组使用in运算符

**检查某个键名是否存在**的运算符`in`，适用于对象，也适用于数组。

```js
var arr = [ 'a', 'b', 'c' ];
2 in arr  // true
'2' in arr // true
4 in arr // false
```

上面代码表明，数组存在键名为`2`的键。由于键名都是字符串，所以数值`2`会自动转成字符串。

注意，如果数组的某个位置是空位，`in`运算符返回`false`。

```js
var arr = [];
arr[100] = 'a';

100 in arr // true
1 in arr // false
```

上面代码中，数组`arr`只有一个成员`arr[100]`，其他位置的键名都会返回`false`。

### 数组的空位

当数组的某个位置是空元素，即两个逗号之间没有任何值，我们称该数组存在空位（hole）。

```
var a = [1, , 1];
a.length // 3
```

上面代码表明，**数组的空位不影响`length`属性**。虽然这个位置没有值，引擎依然认为这个位置是有效的。

需要注意的是，如果最后一个元素后面有逗号，并不会产生空位。也就是说，有没有这个逗号，结果都是一样的。

```js
var a = [1, 2, 3,];

a.length // 3
a // [1, 2, 3]
```

上面代码中，数组最后一个成员后面有一个逗号，这不影响`length`属性的值，与没有这个逗号时效果一样。

**数组的空位是可以读取的，返回`undefined`。**

```js
var a = [, , ,];
a[1] // undefined
```

使用`delete`命令删除一个数组成员，会形成空位，并且不会影响`length`属性。

```js
var a = [1, 2, 3];
delete a[1];

a[1] // undefined
a.length // 3
```

上面代码用`delete`命令删除了数组的第二个元素，这个位置就形成了空位，但是对`length`属性没有影响。也就是说，`length`属性不过滤空位。所以，**使用`length`属性进行数组遍历，一定要非常小心。**

**数组的某个位置是空位，与某个位置是`undefined`，是不一样的。如果是空位，使用数组的`forEach`方法、`for...in`结构、以及`Object.keys`方法进行遍历，空位都会被跳过。**

```js
var a = [, , ,];

a.forEach(function (x, i) {
  console.log(i + '. ' + x);
})
// 不产生任何输出

for (var i in a) {
  console.log(i);
}
// 不产生任何输出

Object.keys(a)
// []
```

**如果某个位置是`undefined`，遍历的时候就不会被跳过。**

```js
var a = [undefined, undefined, undefined];

a.forEach(function (x, i) {
  console.log(i + '. ' + x);
});
// 0. undefined
// 1. undefined
// 2. undefined

for (var i in a) {
  console.log(i);
}
// 0
// 1
// 2

Object.keys(a)
// ['0', '1', '2']
```

这就是说，空位就是数组没有这个元素，所以不会被遍历到，而`undefined`则表示数组有这个元素，值是`undefined`，所以遍历不会跳过。

### 类似数组的对象

如果一个对象的所有键名都是正整数或零，并且有`length`属性，那么这个对象就很像数组，语法上称为“类似数组的对象”（array-like object）。

```js
var obj = {
  0: 'a',
  1: 'b',
  2: 'c',
  length: 3
};

obj[0] // 'a'
obj[1] // 'b'
obj.length // 3
obj.push('d') // TypeError: obj.push is not a function
```

上面代码中，对象`obj`就是一个类似数组的对象。但是，“类似数组的对象”并不是数组，因为它们不具备数组特有的方法。对象`obj`没有数组的`push`方法，使用该方法就会报错。

“类似数组的对象”的根本特征，就是具有`length`属性。只要有`length`属性，就可以认为这个对象类似于数组。但是有一个问题，这种`length`属性不是动态值，不会随着成员的变化而变化。

```js
var obj = {
  length: 0
};
obj[3] = 'd';
obj.length // 0
```

上面代码为对象`obj`添加了一个数字键，但是`length`属性没变。这就说明了`obj`不是数组。

典型的“类似数组的对象”是函数的`arguments`对象，以及大多数 DOM 元素集，还有字符串。

```js
// arguments对象
function args() { return arguments }
var arrayLike = args('a', 'b');

arrayLike[0] // 'a'
arrayLike.length // 2
arrayLike instanceof Array // false

// DOM元素集
var elts = document.getElementsByTagName('h3');
elts.length // 3
elts instanceof Array // false

// 字符串
'abc'[1] // 'b'
'abc'.length // 3
'abc' instanceof Array // false
```

上面代码包含三个例子，它们都不是数组（`instanceof`运算符返回`false`），但是看上去都非常像数组。

数组的`slice`方法可以将“类似数组的对象”变成真正的数组。

```js
var arr = Array.prototype.slice.call(arrayLike);
```

除了转为真正的数组，“类似数组的对象”还有一个办法可以使用数组的方法，就是通过`call()`把数组的方法放到对象上面。

```js
function print(value, index) {
  console.log(index + ' : ' + value);
}

Array.prototype.forEach.call(arrayLike, print);
```

上面代码中，`arrayLike`代表一个类似数组的对象，本来是不可以使用数组的`forEach()`方法的，但是通过`call()`，可以把`forEach()`嫁接到`arrayLike`上面调用。

下面的例子就是通过这种方法，在`arguments`对象上面调用`forEach`方法。

```js
// forEach 方法
function logArgs() {
  Array.prototype.forEach.call(arguments, function (elem, i) {
    console.log(i + '. ' + elem);
  });
}

// 等同于 for 循环
function logArgs() {
  for (var i = 0; i < arguments.length; i++) {
    console.log(i + '. ' + arguments[i]);
  }
}
```

字符串也是类似数组的对象，所以也可以用`Array.prototype.forEach.call`遍历。

```js
Array.prototype.forEach.call('abc', function (chr) {
  console.log(chr);
});
// a
// b
// c
```

注意，这种方法比直接使用数组原生的`forEach`要慢，所以最好还是先将“类似数组的对象”转为真正的数组，然后再直接调用数组的`forEach`方法。

```js
var arr = Array.prototype.slice.call('abc');
arr.forEach(function (chr) {
  console.log(chr);
});
// a
// b
// c
```

### 严格相等运算符

JavaScript 提供两种相等运算符：`==`和`===`。

简单说，它们的区别是相等运算符（`==`）比较两个值是否相等，严格相等运算符（`===`）比较它们是否为“同一个值”。如果两个值不是同一类型，严格相等运算符（`===`）直接返回`false`，而相等运算符（`==`）会将它们转换成同一个类型，再用严格相等运算符进行比较。

下面介绍严格相等运算符的算法。

**（1）不同类型的值**

如果两个值的类型不同，直接返回`false`。

```
1 === "1" // false
true === "true" // false
```

上面代码比较数值的`1`与字符串的“1”、布尔值的`true`与字符串`"true"`，因为类型不同，结果都是`false`。

**（2）同一类的原始类型值**

同一类型的原始类型的值（数值、字符串、布尔值）比较时，值相同就返回`true`，值不同就返回`false`。

```
1 === 0x1 // true
```

上面代码比较十进制的`1`与十六进制的`1`，因为类型和值都相同，返回`true`。

需要注意的是，**`NaN`与任何值都不相等（包括自身）**。另外，正`0`等于负`0`。

```
NaN === NaN  // false
+0 === -0 // true
```

**（3）复合类型值**

两个复合类型（对象、数组、函数）的数据比较时，不是比较它们的值是否相等，而是**比较它们是否指向同一个地址**。

```js
{} === {} // false
[] === [] // false
(function () {} === function () {}) // false
```

上面代码分别比较两个空对象、两个空数组、两个空函数，结果都是不相等。原因是对于复合类型的值，严格相等运算比较的是，它们是否引用同一个内存地址，而运算符两边的空对象、空数组、空函数的值，都存放在不同的内存地址，结果当然是`false`。

如果两个变量引用同一个对象，则它们相等。

```js
var v1 = {};
var v2 = v1;
v1 === v2 // true
```

注意，对于两个对象的比较，严格相等运算符比较的是地址，而大于或小于运算符比较的是值。

```js
var obj1 = {};
var obj2 = {};

obj1 > obj2 // false
obj1 < obj2 // false
obj1 === obj2 // false
```

上面的三个比较，前两个比较的是值，最后一个比较的是地址，所以都返回`false`。

**（4）undefined 和 null**

**`undefined`和`null`与自身严格相等。**

```js
undefined === undefined // true
null === null // true
```

由于变量声明后默认值是`undefined`，因此两个只声明未赋值的变量是相等的。

```js
var v1;
var v2;
v1 === v2 // true
```
