### lodash 源码学习
####add.js

`lodash.add`是一个很简单的函数，将两个数字相加，我直接找到lodash中`add`的源码实现如下
```
const add = createMathOperation((augend, addend) => augend + addend, 0)
```

我们可以发现，当我们使用`add`方法的时候，实际使用的是`createMathOperation`生成出的方法。
```
function createMathOperation(operator, defaultValue) {
  return (value, other) => {
    ...
    ...
    return operator(value, other)
  }
}
```
那么`createMathOperation`方法的作用是什么呢，
搜索一下`createMathOperation`方法，我发现这个方法共被使用了4次，对应的就是`lodash`中的加减乘除操作。

由此我们思考一下，`createMathOperation`方法的主要用途是什么呢？
先来看一下这个方法源码
如下：`value`为`add`方法的第一个参数，`other`为第二个参数
```
	if (value === undefined && other === undefined) {
      return defaultValue
    }
    if (value !== undefined && other === undefined) {
      return value
    }
    if (other !== undefined && value === undefined) {
      return other
    }
    if (typeof value === 'string' || typeof other === 'string') {
      value = baseToString(value)
      other = baseToString(other)
    }
    else {
      value = baseToNumber(value)
      other = baseToNumber(other)
    }
```

* 如果两个值都是 undefined 直接返回默认值
* 如果某一方为undefined 直接返回不为 undefined 的值
* 如果有一方 type === string 对两者都进行 baseToString 操作
* 否则都调用`baseToNumber`方法

当我查看`baseToString`方法的时候，我发现很有意思的一行代码

```
const INFINITY = 1 / 0
const result = `${value}`
return (result == '0' && (1 / value) == -INFINITY) ? '-0' : result
```

第一行是将 value 直接转换为 字符串类型，之前我使用的方法一般是`value+''`，看了这个又有了更好的一个选择

第二行 根据我的分析，这行代码是将数字`-0`转换为字符串`'-0'`，这行代码的双等，涉及了很多隐式转换 ，如果传入 `value`  = `-0`
```
const result = `${value}` //'0'
'0'=='0'// true
1/-0 == -1/0 //true 负无穷等于无穷 返回true
```
将返回字符串`'-0'`

这个结果不同于 数字的`toString`方法

所以这也会影响 lodash add 方法与 普通js 的加法的返回结果

```
lodash.add("1",-0) //'1-0'
'1'+-0//'10'
```

两者结果明显不同

最后再看一下`baseToNumber`方法

```
function baseToNumber(value) {
  if (typeof value === 'number') {
    return value
  }
  if (isSymbol(value)) {
    return NAN
  }
  return +value
}
```
转换为数字
一个很细节的点`+value`可以方便的将值转换为`number`类型，涉及到的是`+`操作符的隐式转换。

