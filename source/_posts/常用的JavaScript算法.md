---
title: 常用的JavaScript算法
date: 2020-12-23 23:48:35
tags:
  - 算法
  - JavaScript
---

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

## 类型判断

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
