- [常用函数](#常用函数)
  - [实现一个bind方法（偏函数）](#实现一个bind方法偏函数)
  - [柯里化](#柯里化)
  - [缓存函数](#缓存函数)
  - [惰性函数](#惰性函数)
  - [防抖函数](#防抖函数)
  - [节流函数](#节流函数)
  - [compose 函数](#compose-函数)
  - [pipe 函数](#pipe-函数)
- [模拟实现](#模拟实现)
  - [bind](#bind)
  - [call](#call)
  - [apply](#apply)
  - [new 关键字](#new-关键字)
  - [forEach](#foreach)
  - [map](#map)
  - [filter](#filter)
  - [reduce](#reduce)
- [算法](#算法)
  - [动态规划](#动态规划)
  - [排序](#排序)
    - [冒泡排序](#冒泡排序)
    - [选择排序](#选择排序)
    - [快速排序](#快速排序)

## 常用函数
### 实现一个bind方法（偏函数）
缓存一部分参数，然后让另一些参数在使用时传入
```javascript
function bind (fn, context) {
  var outterParams = Array.prototype.slice.call(arguments, 2)
  return function () {
    var innerParams = Array.prototype.slice.call(arguments)
    var params = outterParams.concat(innerParams)
    return fn.apply(context, params)
  }
}

// eg. 改变 this 指向
var foo = {
  count: 1,
  bar: function () {
    return this.count
  }
}
var baz = {
  count: 2
}
console.log( foo.bar() ) // 1
console.log( bind(foo.bar, baz)() ) // 2

// eg. 固定一部分参数
function add (a, b) {
  return a + b
}
console.log( add(2, 1) ) // 3
console.log( add(2, 2) ) // 4
console.log( add(2, 3) ) // 5

var addTwo = bind(add, null, 2) // 这里固定 参数 2，实际上可以固定更多的参数
console.log( addTwo(1) ) // 3
console.log( addTwo(2) ) // 4
console.log( addTwo(3) ) // 5
```

### 柯里化
将一个多参数函数转化为多个嵌套的单参数函数
```javascript
function curry (fn) {
  var len = fn.length
  var outterParams = [].slice.call(arguments, 1)
  return function () {
    var innerParams = [].slice.call(arguments)
    var params = outterParams.concat(innerParams)
    if (params.length < len) {
      return curry.apply(null, [fn].concat(params))
    }
    return fn.apply(null, params)
  }
}
// or
function curry (fn) {
  var len = fn.length
  return function curried () {
    var params = [].slice.call(arguments)
    if (params.length < len) {
      return function () {
        var rest = [].slice.call(arguments)
        return curried.apply(null, params.concat(rest))
      }
    }
    return fn.apply(null, params)
  }
}
function sum (a, b, c) {
  return a + b + c
}
console.log( sum(2, 3, 5) ) // 10
// eg.
var sumCurried = curry(sum)
console.log( sumCurried(2, 3, 5) ) // 10
console.log( sumCurried(2, 3)(5) ) // 10
console.log( sumCurried(2)(3)(5) ) // 10
```

### 缓存函数
对于相同参数的函数调用，只会计算一次，后面的调用直接从缓存中返回
```javascript
function memoize() {
  var cache = {}
  return function() {
    var key = JSON.stringify([].slice.call(arguments))
    if (cache.hasOwnProperty(key)) {
      return cache[key]
    }
    var val = fn.apply(null, arguments)
    cache[key] = val
    return val
  }
}

// eg.
function add(a) {
  return a + 10
}
var memoized = memoize(add)
console.log( memoized(10) ) // 20, 第一次调用会计算
console.log( memoized(10) ) // 20, 相同参数直接返回
console.log( memoized(10) ) // 20, 相同参数直接返回
```

### 惰性函数
有些兼容性函数，每次调用的时候都会对浏览器环境进行一次判断，使用惰性函数，可以只进行一次判断
```javascript
function getCss(element, attr) {
  if (window.getComputedStyle) {
    return window.getComputedStyle(element)[attr]
  }
  return element.currentStyle[attr]
}

// 在被调用时重写函数
function getCss(element, attr) {
  if (window.getComputedStyle) {
    getCss = function (element, attr) {
      return window.getComputedStyle(element)[attr]
    }
  } else {
    getCss = function (element, attr) {
      return element.currentStyle[attr]
    }
  }
  return getCss(element, attr)
}

// 初始化函数时就指定函数
var getCss = (function () {
  if (window.getComputedStyle) {
    return function (element, attr) {
      return window.getComputedStyle(element)[attr]
    }
  } else {
    return function (element, attr) {
      return element.currentStyle[attr]
    }
  }
})()

// eg.
console.log( getCss(document.body, 'display') )
```

### 防抖函数
事件在指定时间间隔内不再触发，则会调用事件处理回调
```javascript
function debounce (fn, interval) {
  var timer = null
  interval = interval || 200
  return function () {
    var context = this
    var args = arguments
    clearTimeout(timer)
    timer = setTimeout(function () {
      fn.apply(context, args)
    }, interval)
  }
}

function handleInput () {
  console.log('input')
}
var handleInputDebounced = debounce(handleInput, 300)
document.getElementById('txt').addEventListener('input', handleInputDebounced) // 用户一直触发 input，停止触发 input 300ms后调用 handleInput
```

### 节流函数
事件一直触发，事件处理回调在每次的时间间隔内调用一次
```javascript
// 定时器版
function throttle (fn, interval) {
  var timer = null
  interval = interval || 200
  return function () {
    var context = this
    var args = arguments
    if (!timer) {
      timer = setTimeout(function () {
        timer = null
        fn.apply(context, args)
      }, interval)
    }
  }
}

// 时间差版
function throttle (fn, interval) {
  var start = new Date().getTime()
  interval = interval || 200
  return function () {
    var now = new Date().getTime()
    if (now - start >= interval) {
      fn.apply(this, arguments)
      start = now
    }
  }
}

function handleScroll (event) {
  console.log('scroll')
}
var handleScrollThrottled = throttle(handleScroll, 1000)
document.getElementById('box').addEventListener('scroll', handleScrollThrottled) // 用户一直触发 scroll，每间隔 1000ms 调用一次 handleScroll
```
```html
<div id="box">
  <div style="height: 5000px;"></div>
</div>
```
```css
#box {
  width: 600px;
  height: 600px;
  margin: 50px auto;
  border: 1px solid #eee;
  overflow-y: auto;
}
```

### compose 函数
`compose` 的作用就是将嵌套执行的方法作为参数平铺，嵌套执行的时候，里面的方法也就是右边的方法最开始执行，然后往左边返回（上个函数的输出结果是下个函数的输入参数）  
大致思想就是将 `c(b(a(1)))` 这种写法简写为 `compose(c, b, a)(x)`
```javascript
function compose () {
  var fns = [].slice.call(arguments)
  return function (val) {
    return fns.reduceRight(function (ret, cb) {
      return cb(ret)
    }, val)
  }
}
// webpack 版
var compose = (...fns) => {
  return fns.reduce(
    (prev, next) => {
      return val => prev(next(val))
    },
    val => val
  )
}

function add (a) {
  return a + 10
}
function multiply (a) {
  return a * 10
}
console.log( multiply(add(10)) ) // 200

// eg.
var calc = compose(multiply, add)
console.log( calc(10) ) // 200
```

### pipe 函数
`pipe` 函数跟 `compose` 函数的作用是一样的，也是将参数平铺，只不过它的顺序是从左往右。
```javascript
function pipe () {
  var fns = [].slice.call(arguments)
  return function (val) {
    return fns.reduce(function (ret, cb) {
      return cb(ret)
    }, val)
  }
}
```

## 模拟实现
### bind
同偏函数

### call
### apply
### new 关键字
### forEach
```javascript
function each (arr, fn) {
  for (var i = 0; i < arr.length; i++) {
    if (fn.call(arr, arr[i], i, arr) === false) {
      break
    }
  }
}
```
### map
```javascript
function map (arr, fn) {
  var ret = []
  for (var i = 0; i < arr.length; i++) {
    ret[i] = fn.call(arr, arr[i], i, arr)
  }
  return ret
}
```
### filter
```javascript
function filter (arr, fn) {
  var ret = []
  var retIndex = 0
  for (var i = 0; i < arr.length; i++) {
    var item = arr[i]
    if (fn.call(arr, item, i, arr)) {
      ret[retIndex++] = item
    }
  }
  return ret
}
```
### reduce
```javascript
function reduce(arr, fn, accumulator) {
  for (var i = 0; i < arr.length; i++) {
    accumulator = fn(accumulator, arr[i], i, arr)
  }
  return accumulator
}
```

## 算法
### 动态规划
使用斐波那契（兔子）数列来举例。
斐波那契数：
F(0) = 0,   F(1) = 1

F(N) = F(N - 1) + F(N - 2), 其中 N > 1.

给定  N，计算  F(N)。

eg.
`1、1、2、3、5、8、13、21、34`

- 递归求解
```javascript
function fib (n) {
  if (n === 1 || n === 2) {
    return n
  }
  return fib(n - 1) + fib(n - 2)
}
```
由于大量的递归调用加上不断的重复计算，导致这个算法的非常慢。

- 备忘录解法
```javascript
var lib = (function () {
  var memo = new Map()
  return function (n) {
    var memorized = memo.get(n)
    if (memorized) {
      return memorized
    }
    if (n === 1 || n === 2) {
      return n
    }
    var f1 = fib(n - 1)
    var f2 = fib(n - 2)
    memo.set(n - 1, f1)
    memo.set(n - 2, f2)

    return f1 + f2
  }
})()
```
把无用的重复求值都优化了，在速度上达到了比较优的程度。

- 动态规划
```javascript
function fib (n) {
  var seq = [0, 1] // 超出精度可以使用 Bigint 类型， var seq = [0n, 1n]
  for (var i = 2; i <= n; i++) {
    seq[i] = seq[i - 1] + seq[i - 2]
  }
  return seq[n]
}
```
备忘录解法优化了重复求值的问题，但是没解决递归调用的问题，对于第一次求值，没有被缓存的值，会进入递归调用逻辑，一旦数值太大，那么调用栈过多，会抛出堆栈溢出的错误。

比如 f(10000)，那么必然会递归调用 f(9999)、f(9998) ...... f(0)，而在递归的过程中，这些调用栈是不断叠加的，当函数调用的深处，栈已经达到了成千上万层。

动态规划解法，将解法路径由 **自顶向下** 倒置过来，变成 **自底向上** 的求解。

对于 f(10000)，计算过程为 f(1), f(2), f(3) ...... fib(10000)。

从最小的值开始计算，把每次计算的值缓存（保存在数组中，下标正好用来对应 n 的解答值）里，下面的值都由缓存的值直接得出。

### 排序
#### 冒泡排序
最易懂的排序算法，但是效率较低，生产环境中很少使用。

基本思想：
1. 依次比较相邻的两个数，如果不符合排序规则，则调换两个数的位置。这样一遍比较下来，能够保证最大（或最小）的数排在最后一位。
2. 再对最后一位以外的数组，重复前面的过程，直至全部排序完成。

```javascript
function swap (arr, a, b) {
  var temp = arr[a]
  arr[a] = arr[b]
  arr[b] = temp
}

function bubbleSort (arr) {
  for (var i = 0; i < arr.length - 1; i++) { // 外层循环，5个元素比较4趟（轮）
    var flag = true // 假设数组为排好的数组
    for (var j = 0, stop = arr.length - 1 - i; j < stop; j++) { // 内层循环，两两比较次数，减去 i 为 减去已经排好的数
      if (arr[j] > arr[j + 1]) {
        swap(arr, j, j + 1)
        flag = false // 存在交换项，确定数组为乱序的
      }
    }
    if (flag) {
      break
    }
  }
  return arr
}
```

#### 选择排序
与冒泡排序类似，也是依次对相邻的数进行两两比较。

不同之处在于，它不是每比较一次就调换位置，而是一轮比较完毕，找到最大值（或最小值）之后，将其放在正确的位置（最前面），其他数的位置不变。
```javascript
function swap (arr, a, b) {
  var temp = arr[a]
  arr[a] = arr[b]
  arr[b] = temp
}

function selectionSort (arr) {
  var len = arr.length
  var min = null
  for (var i = 0; i < len; i++) {
    min = 0 // 假设当前位置为最小值
    for (var j = i + 1; j < len; j++) { // 检查数组其余部分是否比假设值更小
      if (arr[j] < arr[min]) {
        min = j
      }
    }
    if (i !== min) { // 当前位置不是最小值，则将其换为最小值
      swap(arr, i, min)
    }
  }
  return arr
}
```

#### 快速排序
公认最快的排序算法之一，有着广泛的应用。

基本思想：先确定一个“基点”（pivot），将所有小于“基点”的值都放在该点的左侧，大于“基点”的值都放在该点的右侧，然后对左右两侧不断重复这个过程，直到所有排序完成。
```javascript
function swap (arr, a, b) {
  var temp = arr[a]
  arr[a] = arr[b]
  arr[b] = temp
}

function partition (arr, left, right) {
  var pivot = arr[Math.floor((left + right) / 2)]
  var i = left
  var j = right
  while (left <= right) {
    while (arr[left] < pivot) {
      left++
    }
    while (arr[right] > pivot) {
      right--
    }
    if (left <= right) {
      swap(arr, i, j)
      left++
      right--
    }
  }
  return i
}

function quickSort(arr, left, right) {
  if (arr.length < 2) {
    return arr
  }
  left = typeof left !== 'number' ? 0 : left
  right = typeof right !== 'number' ? arr.length - 1 : right
  var index = partition(arr, left, right)
  if (left < index - 1) {
    quickSort(arr, left, index - 1)
  }
  if (index < right) {
    quickSort(arr, index, right)
  }
  return arr
}
```
