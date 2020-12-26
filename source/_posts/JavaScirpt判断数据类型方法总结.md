---
title: JavaScirpt判断数据类型方法总结
date: 2020-12-26 15:52:07
tags:
- JavaScirpt
categories:
- JavaScript
---

JavaScript经常需要用到数据类型判断，以下是我总结的几种方法。

<!-- more --->

## 方法一：typeof
typeof 是 JavaScript 中最简单的获取数据类型的方法。

缺点：
- 不能区分数组、正则表达式、日期对象等引用类型的数据，引用类型全为 `object`(函数除外)

示例：
```JavaScript
var str = 'string';
var number = 123;
var boolean = true;
var nullValue = null;
var undefinedValue = undefined;
var obj = { a: 1, b: 2, c: 3 };
var func = function () {
  console.log('hello');
}
var array = [1, 2, 3];
var reg = /.*/;
var date = new Date();

console.log('typeof string: ', typeof str);
console.log('typeof number: ', typeof number);
console.log('typeof boolean: ', typeof boolean);
console.log('typeof nullValue: ', typeof nullValue);
console.log('typeof undefinedValue: ', typeof undefinedValue);
console.log('typeof function: ', typeof func);
console.log('typeof obj: ', typeof obj);
console.log('typeof reg: ', typeof reg);
console.log('typeof date: ', typeof date);
```
运行结果:
```sh
"typeof string: " "string"
"typeof number: " "number"
"typeof boolean: " "boolean"
"typeof nullValue: " "object"
"typeof undefinedValue: " "undefined"
"typeof function: " "function"
"typeof obj: " "object"
"typeof reg: " "object"
"typeof date: " "object"
```

## 方法二：instanceof
缺点：
- 这个方法只能判断引用数据类型
- 只能判断是否某个具体的类型，而不能直接获取对应的数据类型

示例:
```JavaScript
var str = 'string';
var number = 123;
var boolean = true;
var nullValue = null;
var undefinedValue = undefined;
var obj = { a: 1, b: 2, c: 3 };
var func = function () {
  console.log('hello');
}
var array = [1, 2, 3];
var reg = /.*/;
var date = new Date();

console.log('func instanceof Function: ', func instanceof Function);

console.log('reg instanceof RegExp: ', reg instanceof RegExp);

console.log('date instanceof Date: ', date instanceof Date);

console.log('arr instanceof Array', array instanceof Array);
```
运行结果:
```sh
"func instanceof Function: " true
"reg instanceof RegExp: " true
"date instanceof Date: " true
"arr instanceof Array" true
```

## 方法三： 使用constructor
缺点：
- 开发者一旦重写 `prototype`，就容易使得数据的 `constructor` 丢失
- 与 `instanceof` 相同，只能判断数据是否某个具体的数据类型，而不能直接获取数据的数据类型

示例：
```JavaScript
var str = 'string';
var number = 123;
var boolean = true;
var nullValue = null;
var undefinedValue = undefined;
var obj = { a: 1, b: 2, c: 3 };
var func = function () {
  console.log('hello');
}
var array = [1, 2, 3];
var reg = /.*/;
var date = new Date();

// 第三种方式 查看constructor
// 确定 null、undefined 没有 constructor且开发者重写prototype后，原有的constructor可能会丢失
console.log('str.countructor === String: ', str.constructor === String);
console.log("number.constructor === Number: ", number.constructor === Number)
console.log("boolean.constructor === Boolean: ", boolean.constructor === Boolean);
```
运行结果：


## 方法四：最准确的方法 - 使用 Object.prototype.toString()
封装的函数：
```JavaScript
// 第四种方式，最准确的方式,使用Object.prototype.toString
function toString(value) {
  var typeString = Object.prototype.toString.apply(value);
  return typeString.slice(8, -1)
}
```

示例：
```JavaScript
var str = 'string';
var number = 123;
var boolean = true;
var nullValue = null;
var undefinedValue = undefined;
var obj = { a: 1, b: 2, c: 3 };
var func = function () {
  console.log('hello');
}
var array = [1, 2, 3];
var reg = /.*/;
var date = new Date();

// 第四种方式，最准确的方式,使用Object.prototype.toString
function toString(value) {
  var typeString = Object.prototype.toString.apply(value);
  return typeString.slice(8, -1)
}
console.log('toString string: ', toString(str));
console.log('toString number: ', toString(number));
console.log('toString boolean: ', toString(boolean));
console.log('toString nullValue: ', toString(nullValue));
console.log('toString undefinedValue: ', toString(undefinedValue));
console.log('toString function: ', toString(func));
console.log('toString obj: ', toString(obj));
console.log('toString reg: ', toString(reg));
console.log('toString date: ', toString(date));
```
运行结果：
```
"toString string: " "String"
"toString number: " "Number"
"toString boolean: " "Boolean"
"toString nullValue: " "Null"
"toString undefinedValue: " "Undefined"
"toString function: " "Function"
"toString obj: " "Object"
"toString reg: " "RegExp"
"toString date: " "Date"
```

## 方法五：使用JS自带的函数判断
例如：`Array.isArray()`
```
Array.isArray([]);
```
运行结果：
```
true
```