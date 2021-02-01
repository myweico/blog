---
title: 常用的JavaScript算法
date: 2020-12-23 23:48:35
tags:
  - 算法
  - JavaScript
categories:
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

## 字符串全排列

### 广度优先实现

```js
function combine(str) {
  if (str.length === 1) {
    return [str];
  }

  const result = [];
  for (let i = 0; i < str.length; i++) {
    const curChar = str[i];
    const combineArr = combine(str.slice(0, i) + str.slice(i + 1));
    for (let j = 0; j < combineArr.length; j++) {
      result.push(curChar + combineArr[j]);
    }
  }
  // 避免出现 [aa, aa, aa, aa] 的情况
  return [...new Set(result)];
}

console.log("combine(abc)", combine("abc"));
console.log("combine(aab)", combine("aab"));
```

### 深度优先实现

```js
function combine(str) {
  const result = [];
  (function _combine(str, path = "") {
    if (str.length === 0) {
      return result.push(path + str);
    }

    for (let i = 0; i < str.length; i++) {
      _combine(str.slice(0, i) + str.slice(i + 1), path + str[i]);
    }
  })(str);
  // 去重，避免出现'aa' => [aa, aa] 的情况
  return [...new Set(result)];
}

console.log("combine(abc)", combine("abc"));
console.log("combine(aab)", combine("aab"));
```

## 排序和查找

### 插入排序

```js
function sortInsert(arr) {
  // 遍历每一项
  for (let i = 1; i < arr.length; i++) {
    for (let j = i; j >= 0; j--) {
      if (arr[j] < arr[j - 1]) {
        // 比前面的元素小，则交换位置
        [arr[j - 1], arr[j]] = [arr[j], arr[j - 1]];
      } else {
        // 已经到合适位置，停止向前比较
        break;
      }
    }
  }

  return arr;
}

console.log(sortInsert([2, 5, 1, 2, 5, 61, 21]));
```

### 归并排序

并归排序就是将数组分隔成小组，对小组进行排序，等小组有序后，再将有序的小组合并成有序的大组，对数组递归使用并归排序，最终就会得到一个有序的数组

```js
function sortMerge(arr) {
  if (arr.length === 1) {
    // 只有一个数组的时候直接返回
    return arr;
  }

  // 中间的索引，将数组
  const indexMid = ~~(arr.length / 2);

  // 对数组进行分隔，进行并归排序，得到有序的数组
  const [part1, part2] = [
    sortMerge(arr.slice(0, indexMid)),
    sortMerge(arr.slice(indexMid)),
  ];

  console.log("part", part1, part2);

  // 对有序的数组进行合并
  let result = [];

  while (part1.length > 0 && part2.length > 0) {
    result.push(part1[0] < part2[0] ? part1.shift() : part2.shift());
  }

  result = [...result, ...part1, ...part2];

  return result;
}

console.log(sortMerge([2, 5, 1, 2, 5, 61, 21]));
```

### 快速排序

选取一个数值作为参考值，数组其他值与其比较，比它小的在其左侧，比它大的在其右侧，这样这个参考值在数组中的位置就确定了，然后继续递归对左右的数组进行快速排序，最后就会得到一个有序的数组

```js
function sortQuickly(arr) {
  if (arr.length === 0) return [];

  const indexMid = Math.floor(arr.length / 2);
  let arrLeft = [];
  let arrRight = [];

  for (let i = 0; i < arr.length; i++) {
    if (i === indexMid) continue;
    if (arr[i] < arr[indexMid]) {
      // 比选取的值小，则放到左侧
      arrLeft.push(arr[i]);
    } else {
      // 大于等于选取的值，放到右侧
      arrRight.push(arr[i]);
    }
  }

  // 递归对左右的数组进行递归快速排序
  return [...sortQuickly(arrLeft), arr[indexMid], ...sortQuickly(arrRight)];
}

console.log(sortQuickly([2, 5, 1, 2, 5, 61, 21]));
```

### 二位查找

```js
function binarySearch(arr, target) {
  if (arr.length === 0 || (arr.length === 1 && arr[0] !== target)) {
    // 数组已经没有可以比较的时候 || 最后一个元素与目标元素找不到的时候 => 元素不存在
    return -1;
  }

  // 寻找中间的值作为参考值
  const indexMid = arr.length >> 1;
  const valueMid = arr[indexMid];

  // 比较，所等于，则找到了， 目标值若小于参考值，则在左侧的数中继续二分查找，目标值若大于参考值，则取右侧数中二分查找
  if (valueMid === target) {
    return indexMid;
  } else if (target < valueMid) {
    return binarySearch(arr.slice(0, indexMid - 1), target);
  } else {
    return indexMid + 1 + binarySearch(arr.slice(indexMid + 1), target);
  }
}

console.log(binarySearch([1, 2, 3, 6, 9, 11, 28, 42, 81, 98], 81));
```

另一种写法

```js
function binarySearch(arr, target) {
  let indexLeft = 0,
    indexRight = arr.length;

  while (indexLeft <= indexLeft) {
    const indexMid = (indexLeft + indexRight) >> 1;
    const valueMid = arr[indexMid];
    if (target === valueMid) {
      return indexMid;
    } else if (target < valueMid) {
      indexRight = indexMid - 1;
    } else {
      indexLeft = indexMid + 1;
    }
  }
  return -1;
}

console.log(binarySearch([1, 2, 3, 6, 9, 11, 28, 42, 81, 98], 81));
```

### 查找出现得最多的元素

```js
function getMost(arr) {
  // 新建一个map，键值作为map的键名，键值为出现的次数
  let map = new Map();
  arr.forEach((item) => {
    if (map.has(item)) {
      // 若 map 中存在，则次数加 1
      map.set(item, map.get(item) + 1);
    } else {
      // 不存在，则初始化为 1
      map.set(item, 1);
    }
  });

  // 统计完后，取出出现次数最多的键名，就是出现得最多的元素
  let [maxCount, maxValues] = [map.get(arr[0]), [arr[0]]];
  map.forEach((count, value) => {
    if (count > maxCount) {
      maxCount = count;
      maxValues = [value];
    } else if (count === maxCount) {
      maxValues.push(value);
    }
  });

  return maxValues;
}
console.log(getMost(["1", "2", "3", "3", "55", "3", "55", "55", "12", "12"]));
```

## 功能函数

### 使用 setTimeout 实现 setInterval

```js
// 需要返回 id 可以停止定时器
function myInterval(fn, interval, ...args) {
  let isContinuous = true;
  (function _interval() {
    setTimeout(() => {
      fn.apply(this, args);
      if (isContinuous) {
        _interval(fn, interval, ...args);
      }
    }, interval);
  })();

  return () => {
    isContinuous = false;
  };
}

const stop = myInterval(() => {
  console.log("going on");
}, 1000);
```

## 节流和防抖

codepen 实例，请点击 => [https://codepen.io/WeicoMY/pen/OJRvPPj](https://codepen.io/WeicoMY/pen/OJRvPPj)

### 防抖

防抖就是防止抖动，事件持续时间内触发的话只会触发最后一次

```js
function debounce(fn, interval, context) {
  // 设置标志位
  let timeId = undefined;

  // 返回一个函数
  return (...args) => {
    // 一段时间内再次触发的话，取消上次事件
    if (timeId) {
      clearTimeout(timeId);
      timeId = undefined;
    }

    // 设置若干秒后执行
    timeId = setTimeout(() => {
      fn.apply(context, args);
    }, interval);
  };
}
```

### 节流

节流，节省流量，限制触发的频率，限制一段时间内触发的次数

```js
function throttle(fn, interval, context) {
  // 设置标志位
  let isBusy = false;

  // 返回一个函数
  return (...args) => {
    if (isBusy) return;
    isBusy = true;
    fn.apply(context, args);
    setTimeout(() => {
      isBusy = false;
    }, interval);
  };
}
```

## 原生 api 的实现

### bind

```js
Function.prototype.myBind = (context, ...argsBind) => {
  return (...argsPass) => {
    this.apply(context, [...argsBind, ...argsPass]);
  };
};
```

### call

```js
Function.prototype.myCall = function (context, ...args) {
  const __randomKey = Math.random().toString().slice(2);
  context[__randomKey] = this;
  const result = context[__randomKey](...args);
  delete context[__randomKey];
  return result;
};
```

### apply

```js
Function.prototype.myApply = function (context, args) {
  const __randomKey = Math.random().toString().slice(2);
  context[__randomKey] = this;
  const result = context[__randomKey](...args);
  delete context[__randomKey];
  return result;
};

var a = 1;
var obj = {
  a: 2,
};
let func = function (args) {
  console.log("args: ", args);
  console.log("this.a is: ", this.a);
};

func("global.a");
func.myApply(obj, ["obj.a"]);
```

### instanceof

```js
function myInstanceOf(child, parent) {
  while (child.__proto__) {
    if (child.__proto__.constructor === parent) {
      return true;
    }
    child = child.__proto__;
  }
  return false;
}

myInstanceOf([], Array);
```

### new

```js
function myNew(consFunc, ...args) {
  // 构造函数四步

  // 创建一个对象
  const _newObj = {};

  // 将创建对象的 prototype 设置为该构造函数的  prototype
  _newObj.__proto__ = consFunc.prototype;

  // 在这个对象上执行构造函数
  const result = consFunc.apply(_newObj, args);

  // 若执行构造函数返回一个对象，则返回该对象，否则返回创建的对象;
  return typeof result === "object" ? result : _newObj;
}
```

### reduce()

```js
Array.prototype.myReduce = (func, init) => {
  // 接受一个函数，以及一个初始值

  // 若有初始值，则从数组的第一个数值开始
  // 若没有初始值，则初始值就是第一个元素，后续从数组的第二个元素开始
  const hasInit = init !== undefined
  init = init || this[0];
  // 每次都执行函数，将函数的结果赋值给init
  for(let i = hasInit ? 0 : 1; i < this.length i++) {
    init = func(init, this[i], i, this);
  }
  return init;
}
```

### forEach()

```js
Array.prototype.myForEach = (func) => {
  for (let i = 0; i < this.length; i++) {
    func(this[i], i, this);
  }
};
```

### isArray()

```js
Array.myIsArray = (elem) => {
  return Object.prototype.toString.apply(elem).slice(8, -1) === "Array";
  // return Object.prototype.toString.apply(elem).toLowerCase() === "[object array]";
};
```

### sleep()

```js
function sleep(time) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve();
    }, time);
  });
}
```

### Promise()
