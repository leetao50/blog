---
title: JavaScript中检测对象中是否存在某个属性
date: 2023-07-12 11:37:17
tags:
---

检测对象中属性的存在与否可以通过几种方法来判断。

# 使用in关键字

如果指定的属性在指定的对象或其原型链中，则 in 运算符返回 true。

```javascript
const car = { make: 'Honda', model: 'Accord', year: 1998 };

console.log('make' in car);
// Expected output: true

delete car.make;

console.log('make' in car);
// Expected output: false

if ('make' in car === false) {
  car.make = 'Suzuki';
}

console.log(car.make);
// Expected output: "Suzuki"


console.log('make' in car);
// Expected output: true
```

## 语法

prop in object

**参数**

+ prop：一个字符串类型或者 symbol 类型的属性名或者数组索引（非 symbol 类型将会强制转为字符串）。

## 用法

下面的例子演示了一些 in 运算符的用法。
```javascript
// 数组
var trees = new Array("redwood", "bay", "cedar", "oak", "maple");
0 in trees        // 返回 true
3 in trees        // 返回 true
6 in trees        // 返回 false
"bay" in trees    // 返回 false (必须使用索引号，而不是数组元素的值)

"length" in trees // 返回 true (length 是一个数组属性)

Symbol.iterator in trees // 返回 true (数组可迭代，只在 ES2015+ 上有效)

// 内置对象
"PI" in Math          // 返回 true

// 自定义对象
var mycar = {make: "Honda", model: "Accord", year: 1998};
"make" in mycar  // 返回 true
"model" in mycar // 返回 true
```

in右操作数必须是一个对象值。例如，你可以指定使用String构造函数创建的字符串，但不能指定字符串文字。

```javascript
var color1 = new String("green");
"length" in color1 // 返回 true
var color2 = "coral";
"length" in color2 // 报错 (color2 不是对象)
```

### 对被删除或值为 undefined 的属性使用in

如果你使用 delete 运算符删除了一个属性，则 in 运算符对所删除属性返回 false。
```javascript
var mycar = {make: "Honda", model: "Accord", year: 1998};
delete mycar.make;
"make" in mycar;  // 返回 false

var trees = new Array("redwood", "bay", "cedar", "oak", "maple");
delete trees[3];
3 in trees; // 返回 false
```

如果你只是将一个属性的值赋值为undefined，而没有删除它，则 in 运算仍然会返回true。
```javascript
var mycar = {make: "Honda", model: "Accord", year: 1998};
mycar.make = undefined;
"make" in mycar;  // 返回 true

var trees = new Array("redwood", "bay", "cedar", "oak", "maple");
trees[3] = undefined;
3 in trees; // 返回 true
```

### 继承属性
如果一个属性是从原型链上继承来的，in 运算符也会返回 true。

```javascript
"toString" in {}; // 返回 true
```


# 使用对象的hasOwn()方法
如果指定的对象自身有指定的属性，则静态方法 Object.hasOwn() 返回 true。如果属性是继承的或者不存在，该方法返回 false。

> 备注： Object.hasOwn() 旨在取代 Object.prototype.hasOwnProperty()。

```javascript
var o={x:1};
o.hasOwnProperty("x");    　　 //true，自有属性中有x
o.hasOwnProperty("y");    　　 //false，自有属性中不存在y
o.hasOwnProperty("toString"); //false，这是一个继承属性，但不是自有属性

const object1 = {
  prop: 'exists'
};

console.log(Object.hasOwn(object1, 'prop'));
// Expected output: true

console.log(Object.hasOwn(object1, 'toString'));
// Expected output: false

console.log(Object.hasOwn(object1, 'undeclaredPropertyValue'));
// Expected output: false

```

## 语法

Object.hasOwn(obj, prop)

+ obj：要测试的 JavaScript 实例对象。
+ prop：要测试属性的 String 类型的名称或者 Symbol。
+ 返回值：如果指定的对象中直接定义了指定的属性，则返回 true；否则返回 false。

建议使用此方法替代 Object.hasOwnProperty()，因为它适用于使用 Object.create(null) 创建的对象且重写了继承的 hasOwnProperty() 方法的对象。尽管可以通过在外部对象上调用 Object.prototype.hasOwnProperty() 解决这些问题，但是 Object.hasOwn() 更加直观。

## 用法

### 使用 hasOwn 去测试属性是否存在
以下代码展示了如何确定 example 对象中是否包含名为 prop 的属性。

```javascript
const example = {};
Object.hasOwn(example, 'prop');   // false——目标对象的属性 'prop' 未被定义

example.prop = 'exists';
Object.hasOwn(example, 'prop');   // true——目标对象的属性 'prop' 已被定义

example.prop = null;
Object.hasOwn(example, 'prop');   // true——目标对象本身的属性存在，值为 null

example.prop = undefined;
Object.hasOwn(example, 'prop');   // true——目标对象本身的属性存在，值为 undefined

```

## 直接属性和继承属性

const example = {};
example.prop = 'exists';
```javascript

// `hasOwn` 静态方法只会对目标对象的直接属性返回 true：
Object.hasOwn(example, 'prop');             // 返回 true
Object.hasOwn(example, 'toString');         // 返回 false
Object.hasOwn(example, 'hasOwnProperty');   // 返回 false

// `in` 运算符对目标对象的直接属性或继承属性均会返回 true：
'prop' in example;                          // 返回 true
'toString' in example;                      // 返回 true
'hasOwnProperty' in example;                // 返回 true

```

## 迭代对象的属性

要迭代对象的可枚举属性，你应该这样使用：
```javascript

const example = { foo: true, bar: true };
for (const name of Object.keys(example)) {
  // …
}

```
但是如果你使用 for...in，你应该使用 Object.hasOwn() 跳过继承属性：

```javascript
const example = { foo: true, bar: true };
for (const name in example) {
  if (Object.hasOwn(example, name)) {
    // …
  }
}

```
## 检查数组索引是否存在
Array 中的元素被定义为直接属性，所以你可以使用 hasOwn() 方法去检查一个指定的索引是否存在：

```javascript
const fruits = ['Apple', 'Banana','Watermelon', 'Orange'];
Object.hasOwn(fruits, 3);   // true ('Orange')
Object.hasOwn(fruits, 4);   // false——没有定义的

```
## hasOwnProperty 的问题案例
本部分证明了影响 hasOwnProperty 的问题对 hasOwn() 是免疫的。首先，它可以与重新实现的 hasOwnProperty() 一起使用：
```javascript

const foo = {
  hasOwnProperty() {
    return false;
  },
  bar: 'The dragons be out of office',
};

if (Object.hasOwn(foo, 'bar')) {
  console.log(foo.bar); //true——重新实现 hasOwnProperty() 不会影响 Object
}

```
它也可以用于测试使用 Object.create(null) 创建的对象。这些对象不会继承自 Object.prototype，因此 hasOwnProperty() 方法是无法访问的。
```javascript

const foo = Object.create(null);
foo.prop = 'exists';
if (Object.hasOwn(foo, 'prop')) {
  console.log(foo.prop); //true——无论对象是如何创建的，它都可以运行。
}

```


# 用undefined判断

自有属性和继承属性均可判断。

```javascript
var o={x:1};
o.x!==undefined;        //true
o.y!==undefined;        //false
o.toString!==undefined  //true
```

该方法存在一个问题，如果属性的值就是undefined的话，该方法不能返回想要的结果，如下

```javascript
var o={x:undefined};
o.x!==undefined;        //false，属性存在，但值是undefined
o.y!==undefined;        //false
o.toString!==undefined  //true
```

# 在条件语句中直接判断

```javascript
var o={};
if(o.x) {   //如果x是undefine,null,false," ",0或NaN,它将保持不变
o.x+=1;  
}
```
