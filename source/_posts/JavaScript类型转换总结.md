---
title: JavaScript类型转换总结
date: 2021-02-20 21:20:10
tags:
- JavaScript
---

你知道 `[] != ![]` 的结果吗，结果是 true，你知道为什么吗？看完这篇你就知道了。

<!-- more -->

## Boolean
类型 | 转为 true | 转为 false
--- | --- | ---
Boolean | `true` | `false`
String | 非字符串 | `""`（空字符串）
Number | 不为 `0` 或者 `NaN` 的数值 | `0` 或者 `NaN`
Object | 任意对象 | `null`
Undefined | `N/A`（不存在） | `undefined`
Symbol | 任意 `Symbol` | `N/A`（不存在）

## Number
数值转换有三种：
- Number
- parseInt
- parseFloat

### Number
转换规则：
- 布尔值
    - true => 1
    - false => 0
- 数值，直接返回
- 字符串
    - 若为有效数值符号，包括正负号，则转换为字符串
        - Number("1") => 1
        - Number("-21") => 21
        - Number("0123") => 123，忽略前面的0
    - 包含有效的浮点值格式如"1.1"，则会转换为相应的浮点值（同样，忽略前面的零）
        - Number("1.1") => 1.1
    - 包含有效的十六进制，则转为对应的十进制数值
        - Number("0x1c") => 28
    - 若为空字符串，返回0
        - Number("") => 0
    - 其他字符串，返回 NaN
        - Number("wmy1.1") => NaN
- undefined, 转为 NaN
- null，转为 0
- 对象
    - 先调用 valueOf 方法，若返回原始值，采用上面的转换方法
    - 调用 toString 方法（若没有 valueOf 方法或者 valueOf() 没有返回原始值），获取到原始值，采用以上转换方法
    - 否则报错 TypeError
```
let num1 = Number("Helloworld!");   // NaN
let num2 = Number("");              // 0
let num3 = Number("000011");        // 11
let num4 = Number(true);            // 1
let num5 = Number("1.1b")           // NaN
let obj1 = {
    valueOf() {
        console.log('in valueOf');
        return 1
    },
    toString() {
        console.log('in toString');
        return '2'
    }
}
let obj2 = {
        valueOf() {
        console.log('in valueOf');
        return {}
    },
    toString() {
        console.log('in toString');
        return '2'
    }
}
let obj3 = {
       valueOf() {
        console.log('in valueOf');
        return {}
    },
    toString() {
        console.log('in toString');
        return {}
    } 
}
let num6 = Number(obj1); // in valueOf, num6 => 1
let num7 = Number(obj2); // in valueOf in toString num7 => 2
let num8 = Number(obj3); // in valueOf in toString TypeError
```

### parseInt
考虑到用Number()函数转换字符串时相对复杂且有点反常规，通常在需要得到整数时可以优先使用parseInt()函数。

```
parseInt(string [, radix])
```
- string：接受的字符串，若不是字符串，会调用 toString 将参数先转为字符串
- radix：字符串表示的进制数
> MDN: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/parseInt

转换规则：
- 若 string 不是以数字字符开头（包括正负号），直接返回 NaN
    - string 为空字符串，返回 NaN
    - stirng 为 undefined、null、对象的时候，返回 NaN
- 若 string 表示为进制数值
    - 0x开头的，将十六进制转为十进制
    - 0开头的，某些实现会将八进制转为十进制
    ```js
    let num1 = parseInt("1234blue");   //1234
    let num2 = parseInt("");       //NaN
    let num3 = parseInt("0xA");    //10，解释为十六进制整数
    let num4 = parseInt(22.5);     //22
    let num5 = parseInt("70");     //70，解释为十进制值
    let num6 = parseInt("0xf");    //15，解释为十六进制整数
    ```
- 第二个参数，可以指定字符串的表示基数
```js
let num7 = parseInt("12", 8)    // 10
```

### parseFloat

转换规则：
- parseFloat 跟 parseInt 类似，也是从第一个字符开始检测，检测到字符串结尾或者不是有效浮点数字符为止
- 只能识别十进制，不能指定底数，始终忽略前面的0，所以十六进制的字符串始终返回 0
- 若字符串表示整数，则最后返回整数

```js
let num1 = parseFloat("1234blue");    //1234，按整数解析
let num2 = parseFloat("0xA");         // 0
let num3 = parseFloat("22.5");        // 22.5
let num4 = parseFloat("22.34.5");     // 22.34
let num5 = parseFloat("0908.5");      // 908.5
let num6 = parseFloat("3.125e7");     // 31250000
```

## String
转为字符串有两种方法
- 调用 toString 方法，对象以及原始类型包装对象都有 toSting 方法
    - Number、Boolean、String、Object
- 使用 String 方法转换

转换规则如下：

类型 | 字符串
--- | ---
`true` | `'true'`
`false` | `'false'`
`null` | `'null'`
`undefined` | `'undefined'`
`NaN` | `'NaN'`
`0` | `'0'`
`123` | `'123'`
`'string'` | `'string'`（返回字符串的副本）
`object` | `'[object Object]'`
`[]` | `''`
`[1, 2, 3, 'abc']` | `"1,2,3,abc"`
`function () {}` | `"function() {}"`
`new Date()` | `"Sat Feb 20 2021 17:06:49 GMT+0800 (China Standard Time)"`
`/abcd/` | `"/abcd/"`

对于对象：
- 先调用 toString()，若返回原始值，将原始值转为字符串
- 若 toString 不存在或者没有返回原始值，调用 valueOf()，若返回原始值，将原始值转为字符串
- 否则报错，TypeError

## 加法操作符
运算规则：
- 若两个操作数都是数值
   - 如果有任一操作数是NaN，则返回NaN；
   - 如果是Infinity加Infinity，则返回Infinity；
   - 如果是Infinity加Infinity，则返回Infinity；
   - 如果是Infinity加Infinity，则返回NaN；
   - 如果是+0加+0，则返回+0；
   - 如果是0加+0，则返回+0；
   - 如果是-0加-0，则返回-0。
- 若有操作数是字符串
    - 将两个操作数都转为字符串，然后拼接起来
- 若为其他情况，都将其使用 String 转为字符串，然后拼接起来

## 减法操作符
运算规则：
- 若操作数为非数值，都先使用 Number 将操作数转为数值
- 如果是Infinity减Infinity，则返回NaN。
- 如果是-Infinity减Infinity，则返回-Infinity。
- 如果是Infinity减-Infinity，则返回Infinity。
- 如果是+0减+0，则返回+0。
- 如果是+0减-0，则返回-0。
- 如果是-0减-0，则返回+0。

## 递增/递减操作符
转换规则：
- 字符串
    - 有效的字符串，转换为数值，再应用改变
    - 无效的数值形式，转为 NaN
- 布尔值
    - true，转为 1，再应用改变
    - false，转为 0，再应用改变
- 浮点数
    - 加1 或 减1
- 对象
    - 先调用 valueOf()，得到原始值，再应用上面的改变
    - 若为 vallueOf 不存在或者没有返回原始值，再调用 toSting()，获取到原始值再应用上面的规则
    - 否则报错，TypeError

## 一元加和减
相当于将操作数转为数值，Number(target)

## 乘法操作符
运算规则：
- 如果操作数都是数值，则执行常规的乘法运算，即两个正值相乘是正值，两个负值相乘也是正值，正负符号不同的值相乘得到负值。如果ECMAScript不能表示乘积，则返回Infinity或-Infinity。
- 如果有任一操作数是NaN，则返回NaN。
- 如果是Infinity乘以0，则返回NaN。
- 如果是Infinity乘以非0的有限数值，则根据第二个操作数的符号返回Infinity或Infinity。
- 如果是Infinity乘以Infinity，则返回Infinity。
- 如果有不是数值的操作数，则先在后台用Number()将其转换为数值，然后再应用上述规则。

## 除法操作符
运算规则：
- 如果操作数都是数值，则执行常规的除法运算，即两个正值相除是正值，两个负值相除也是正值，符号不同的值相除得到负值。如果ECMAScript不能表示商，则返回Infinity或Infinity。
- 如果有任一操作数是NaN，则返回NaN。
- 如果是Infinity除以Infinity，则返回NaN。
- 如果是0除以0，则返回NaN。
- 如果是非0的有限值除以0，则根据第一个操作数的符号返回Infinity或-Infinity。
- 如果是Infinity除以任何数值，则根据第二个操作数的符号返回Infinity或-Infinity。
- 如果有不是数值的操作数，则先在后台用Number()函数将其转换为数值，然后再应用上述规则。

## 位操作符
转换规则：
- 会使用Number()函数将该值转换为数值。
- 有一点要注意，特殊值NaN和Infinity在位操作中都会被当成0处理。
- ECMAScript中的所有数值都以IEEE75464位格式存储，但位操作并不直接应用到64位表示，而是先把值转换为32位整数，再进行位操作，之后再把结果转换为64位。

## 关系运算符
转换规则：
- 若两个都是字符串，则按照字符串比较
- 否则，将两个操作数都使用 Number 转成数值进行比较

例子：
```js
"23" < "3"      // true
"23" < 3        // false，因为 "23" 会转换为 数值进行比较，23 < 3 为 false
"a" < 3         // false，因为 "a" 会转化为 NaN，所以返回 false
```

## 等号和非等号
类型转换规则：
- 若任一操作数为布尔值，则转化为数值比较，ture 转换为 1，false 转换为 0
- 若有一个操作数是字符串，另一个操作数为数值，使用 Number() 将字符串转换为数值
- 若有一个操作数是对象，另一个不是，使用 Number() 将对象转换为数值比较

比较的规则：
- undefined 和 null 相等
- undefined 和 null 不再进行类型转换
- NaN 和 任意值都不相等，包括自身
- 若两个操作数都是对象，则比较他们是否为同一对象（引用值是否一致），是同一个对象则返回 true，否则返回 false

特殊的情况：

表达式 | 结果
--- | ---
`null == undefined` | `true`
`"NaN" == NaN` | `false`
`5 == NaN` | `false`
`NaN == NaN` | `false`
`NaN != NaN` | `true`
`false == 0` | `true`
`true == 1` | `true`
`true == 2` | `false`
`undefined == 0` | `false`
`null == 0` | `false`
`"5" == 5` | `true`

## 参考文档
马特·弗里斯比. JavaScript高级程序设计（第4版）（图灵图书） (Chinese Edition) (Kindle Locations 2306-2308). Kindle Edition. 