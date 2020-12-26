---
title: 常用的JavaScript算法
date: 2020-12-23 23:48:35
tags:
  - 算法
  - JavaScript
---

常用的 JavaScript 方法。

<!-- more --->

## Dom 相关

### 事件代理

```JavaScript
document.getElementById("father-id").onClick = function(event) {
  event = event || window.event;
  const target = event.target || event.srcElement;
  if (target.nodeName.toLowerCase() === 'xxx') {
    // event handler
    // ...
  }
}
```

## 数组相关

### 数组扁平化

```JavaScript
function flatten(arr) {
  if (!Array.isArray(arr)) return [];
  let _array = [];
  arr.forEach(function(elem) {
    if (Array.isArray(elem)) {
      _array = _array.concat(flatten(elem));
      // arr = [...arr, ...flatten(elem)]
    } else {
      // 不是数组元素
      _array.push(elem);
    }
  })
  return _array;
}

const array = [1, [2], [[3]], [[[4]]]];
const arrayFlatten= flatten(array);
console.log('array before flatten', array);
console.log('array after flatten', arrayFlatten);
```

运行结果:

```
array before flatten (4) [1, Array(1), Array(1), Array(1)]
array after flatten: [1, 2, 3, 4]
```

## 去重

方法 1

```JavaScript
function unique(array) {
  const set = new Set();
  const arrayUnique = [];
  array.forEach(function (elem) {
    if (set.has(elem)) return;

    // 元素不存在，添加到数组中
    arrayUnique.push(elem);
    set.add(elem);
  })
  return arrayUnique;
}

console.log(unique([1,2,2,2,3,4,5]))
```

方法 2

```JavaScript
function unique(array) {
  return [...new Set(array)];
}

console.log(unique([1,2,2,2,3,4,5]));
```

运行结果：

```
[ 1, 2, 3, 4, 5 ]
```

## 拷贝

### 浅拷贝

方法 1

```JavaScript
function copy(obj) {
  const result = Array.isArray(obj) ? [] : {};
  // 处理了键名为 Symbol 的情况
  [...Object.keys(obj), ...Object.getOwnPropertySymbols(obj)].foreach(key => {
    result[key] = obj[key]
  })
  return result;
}
```

方法 2

```js
function copy(obj) {
  return Array.isArray(obj) ? [...obj] : { ...obj };
}
```

方法 3

```js
function copy(obj) {
  return Object.assign(Array.isArray(obj) ? [] : {}, obj);
}
```

## 深拷贝

方法 1，使用 `JSON.stringify(JSON.parse(obj))`
缺点：

- 遇到 `undefined`, `NaN`, `-Infinity`, `Infinity`， `function`，`RegExp` 会默认转为 `null`,
- 遇到函数会报错
- `Date`对象会转为字符串
- 循环引用时候回报错（a = {}, a.b = a 这种情况)

```js
function deepCopy(obj) {
  return JSON.parse(JSON.stringify(obj));
}
```

上面的方法遇到函数的话，JSON 将会序列化错误，可以使用 JSON.parse 以及 JSON.stringify 的第二个参数，对函数以及其他数据类型进行处理

```js
function handleParse(key, value) {
  console.log(value);
  return eval(value);
}

function handleStringify(key, value) {
  console.log(value, typeof value);
  if (typeof value === "function") {
    // 避免 valuee = function() {} 时报错
    return `i = ${value}`;
  }

  if (value instanceof RegExp) {
    // 正则表达式
    return `${value}`;
  }

  if (
    typeof value === "string" &&
    /^\d{4}-\d{2}-\d{2}T\d{2}:\d{2}:\d{2}/.test(value)
  ) {
    // 时间日期对象，JSON.stringify(会自动把日期对象转为字符串，value已经是日期字符串格式 2020-12-26T12:38:48.964Z)
    return `new Date('${value}')`;
  }
  return value;
}

function deepCopy(obj) {
  return JSON.parse(JSON.stringify(obj, handleStringify), handleParse);
}
```

方法 2
解决了循环引用以及键名为 Symbol 的情况

```js
function deepCopy(obj, cached = new Map()) {
  // 若为函数的话，
  if (typeof obj === "function") {
    // // 直接返回
    // return obj;

    // 若想复制函数，可以
    return eval(`i = ${obj}`);
  }

  // 若为原始值，则直接返回
  if (typeof obj !== "object") return obj;

  // 处理正则表达式
  if (obj instanceof RegExp) {
    // // 直接返回
    // return obj;

    // 若想复制，可以
    return eval(`${obj}`);
  }

  // 处理 Date 对象
  if (obj instanceof Date) {
    // // 直接返回
    // return obj;

    // 若想复制，可以
    return eval(`new Date(${obj.getTime()})`);
  }

  if (cached.has(obj)) {
    // 若已经出现，则直接返回
    return cached.get(obj);
  }

  const result = Array.isArray(obj) ? [] : {};

  // 加入到缓存中
  cached.set(obj, result);

  [...Object.keys(obj), ...Object.getOwnPropertySymbols(obj)].forEach(function (
    key
  ) {
    result[key] = deepCopy(obj[key], cached);
  });

  return result;
}
```

## 字符串

### 去空格

方法 1，正则表达式替换

```js
function myTrim(str) {
  return str.replace(/(^\s*)|(\s*$)/g, "");
}

console.log(myTrim("    ok    ").length);
```

方法 2, 算出第一个不是空格的位置，以及最后一个不是空格的位置，然后截取字符对应的位置

```js
function myTrim(str) {
  let start = 0;
  let end = str.length - 1;
  for (; start < str.length; start++) {
    if (str[start] !== " ") break;
  }

  for (; end >= 0; end--) {
    if (str[end] !== " ") break;
  }

  return str.slice(start, end + 1);
}

console.log(myTrim("  ok   ").length);
```

方法 3： 字符串对象内置方法 trim()

```js
let str = "   ok  ";
console.log(str.trim());
```
