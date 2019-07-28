---
title: 重学JS数组
date: 2019-07-28 13:41:00
tags: "Javascript,数组"
---
## 序言

除了Object类型之外，Array类型恐怕是js中最常用的类型了，并且随着js的发展进步，数组中提供的方法也越来越来，对数组的处理也出现了各种骚操作，此篇文章将重新学习数组中的实例方法

## 数组转换

* join()方法接收一个字符串作为分隔符，并返回用分隔符连接的数组项字符串

> 参数：分隔符字符串

``` bash
const arr = [1, 2, 3]
console.log(arr.join('|')) // '1|2|3'
```

* toString()方法返回数组中每个值的字符串形式拼接而成的一个以逗号分隔的字符串

> 参数：无

``` bash
const arr = [1, 2, 3]
console.log(arr.toString()) // '1,2,3'
```

## 栈方法

* push()方法接收任意数量的参数，把它们逐个添加到数组末尾，并返回修改后的数组长度

> 参数：item...（多个数组项）

``` bash
const arr = [1, 2, 3]
console.log(arr.toString()) // '1,2,3'
```

* pop()方法从数组末尾移除最后一项，减少数组的length，返回移除的项

> 参数：无

``` bash
let arr = ['a', 'b', 'c']
const item = arr.pop()
console.log(item) // 'c'
console.log(arr) // [ 'a', 'b' ]
```

## 队列方法

* shift()方法移除数组中的第一项，并返回该项，同时数组长度减1

> 参数：无

``` bash
let arr = ['a', 'b', 'c']
const item = arr.shift()
console.log(item) // 'a'
console.log(arr) // [ 'b', 'c' ]
```

* unshit()方法在数组前端添加任意个项，并返回新数组的长度

> 参数：item...(多个数组项)

``` bash
let arr = ['a', 'b', 'c']
const count = arr.unshift('d', 'e')
console.log(count) // 5
console.log(arr) // [ 'd', 'e', 'a', 'b', 'c' ]
```

## 排序方法

* reverse()方法用于反转数组中每一项，并返回反转后的数组

> 参数：无

``` bash
let arr = ['a', 'b', 'c']
console.log(arr.reverse()) // [ 'c', 'b', 'a' ]
```

* sort()方法用将数组排序，并返回排序后的数组

> 参数：compareFunction(可选)
> 若不传compareFunction，sort()方法回调用每个数组项的toString()方法，然后比较得到的字符串

``` bash
let arr = [2, 3, 10]
arr.sort()
console.log(arr) // [ 10, 2, 3 ]
```

> '10'位于'2'之前
> 若传compareFunction(a,b)，如果返回值小于0，则a位于b之前，如果返回值等于0则位置不变，如果返回值大于，则b位于a之前

``` bash
let arr = [2, 5, 3, 1]
arr.sort((a, b) => {
  if (a < b) {
    return -1
  } else if (a > b) {
    return 1
  } else {
    return 0
  }
})
console.log(arr) // [ 1, 2, 3, 5 ]
```

## 操作方法

* concat()方法创建当前数组一个副本，然后将接收到的参数添加到这个副本末尾，最后返回新构建的数组

> 参数：item...(可以是数组项，也可以是数组)

``` bash
let arr = [1, 2, 3]
let newArr = arr.concat(4, 5, [6, 7])
console.log(newArr) // [ 1, 2, 3, 4, 5, 6, 7 ]
```

* slice()方法基于当前数组中的一或多个项创建一个新数组

> 参数：start(起始位置),end(结束位置，可选)

``` bash
let arr = [1, 2, 3, 4]
let newArr = arr.slice(1, 3)
console.log(newArr) // [ 2, 3 ]
```

>tip: 如果slice方法的参数中有一个负数，则用数组长度加上该数来确定相应的位置

* splice()方法用法有多种，根据不同的用法需要传递的参数也不一样

> 删除：可以删除任意数量的项，指定两个参数：删除的第一项位置和删除的数量
> 插入：可以向指定位置插入任意数量的项，第一个参数：插入的位置，第二个参数0（删除0），第三个参数以后要插入的项
> 替换：可以将指定位置的项替换，第一个参数要替换项的位置，第二个替换项个数，第三个参数以后新的项

``` bash
let arr = [1, 2, 3, 4, 5]
arr.splice(0, 1)
console.log(arr) // [ 2, 3, 4, 5 ]
arr.splice(1, 0, 'hello', 'world')
console.log(arr) // [ 2, 'hello', 'world', 3, 4, 5 ]
arr.splice(3, 1, 'js')
console.log(arr) // [ 2, 'hello', 'world', 'js', 4, 5 ]
```

## 位置方法

* indexOf()方法从头开始查找指定项，找到返回对应数组下标，没找到返回-1

> 参数：item(要查找的数组项),index(指定开始查找的位置，可选)

``` bash
let arr = [1, 2, 3, 4, 5]
console.log(arr.indexOf(3)) // 2
console.log(arr.indexOf(3, 3)) // -1
```

* lastIndexOf()方法用法和indexOf基本一致，只是从数组尾部开始查找

## 迭代方法

* every()方法对数组中每一项运行给定函数，如果该函数对每一项都返回true，则返回true

> 参数：callback(item, index, arr)

``` bash
let arr = [1, 2, 3, 4, 5]
console.log(arr.indexOf(3)) // 2
console.log(arr.indexOf(3, 3)) // -1
```

* some()方法对数组中任意一项运行给定函数，如果该函数对任意一项返回true，则返回true

> 参数：callback(item, index, arr)

``` bash
let arr = [1, 2, 3, 4]
let result = arr.some((item) => item > 3)
console.log(result) // true
```

* map()方法对数组中的每一项运行给定函数，返回每次函数调用的结果组成的数组

> 参数：callback(item, index, arr)

``` bash
let arr = [1, 2, 3]
let result = arr.map(item =>  item * 2)
console.log(result) // [ 2, 4, 6 ]
```

* filter()方法对数组中的每一项运行给定函数，返回该函数调用会返回true的项组成的数组

> 参数：callback(item, index, arr)

``` bash
let arr = [1, 2, 3, 4, 5]
let result = arr.filter(item => item > 2)
console.log(result) // [3, 4, 5]
```

* forEach()方法对数组中的每一项都运行给定函数，没有返回值

> 参数：callback(item, index, arr)

``` bash
let arr = [1, 2, 3, 4, 5]
arr.forEach(item => {
  console.log(item) // 1 2 3 4 5
})
```
> tip: 修改item的不会影响遍历的数组项

## 缩小方法

* reduce()方法对数组中的每一项执行一个reducer函数(升序执行)，并将结果汇总为单个返回值

> 参数：callback(accumulator(累计器累计回调的返回值)，currentValue(数组中正在处理的元素)，currentIndex(数组中正在处理的元素的索引，如果提供了initialValue，则起始索引号为0，否则为1，可选)，array(调用reducer的数组)), initialValue(作为第一次调用callback函数时的第一个参数的值。 如果没有提供初始值，则将使用数组中的第一个元素。 在没有初始值的空数组上调用 reduce 将报错，可选)

``` bash
let arr = [1, 2, 3, 4, 5]
let result = arr.reduce((accumulator, current, index, array) => {
  return accumulator + current
})
console.log(result) // 15 1+2+3+4+5
```

``` bash
let arr = [1, 2, 3, 4, 5]
let result = arr.reduce((accumulator, current, index, array) => {
  return accumulator + current
}, 10)
console.log(result) // 25 10+1+2+3+4+5
```

* reduceRight()方法用法与reduce()方法一致，只是redeceRight()方法调用从数组尾部开始，向前遍历

## ES6新增方法

* from()方法将类似数组的对象和可遍历的对象转化为数组

> 参数：arrayLike(想要转换成数组的伪数组对象或可迭代对象)，mapFn(如果指定了该参数，新组数中的每个元素会执行此回调函数，可选)，thisArg(执行回调函数时this对象，可选)

``` bash
let arrayLike = {
  0: 1,
  1: 2,
  2: 3,
  length: 3
}
console.log(Array.from(arrayLike)) // [1, 2, 3]
```

``` bash
let arrayLike = {
  0: 1,
  1: 2,
  2: 3,
  length: 3
}
console.log(Array.from(arrayLike, (item) => item * 2)) // [2, 4, 6]
```
> 在实际应用中更多应用于Dom操作返回的集合以及函数内部的arguments对象

* copyWithin()方法，在当前数组内部，将指定位置的成员复制到其他位置(会覆盖原有位置成员)，返回当前数组

> 参数：target(从该位置开始替换数据)，start(从该位置开始读取数据，默认为0，如果为负值表示倒数，可选)，end(到该位置前，停止读取数据，默认为数组长度，如果为负值，表示倒数)

``` bash
let arr = [1, 2, 3, 4, 5]
arr.copyWithin(0, 3, 5)
console.log(arr) // [ 4, 5, 3, 4, 5 ]
```

* find()方法用于找出第一个符合条件的数组成员，若没有符合条件的，返回undefined

> 参数：callback(item, index, arr)

``` bash
let arr = [1, 2, 3, 4, 5]
let result = arr.find(item => item > 3)
console.log(result) // 4
```

* findIndex()方法用法与find()方法相似，返回第一个符合条件的成员的位置，若没有符合条件的，返回-1

* fill()方法用一个固定值填充一个数组中从起始索引到终止索引内的全部元素，不包括终止索引

> 参数：value(填充数组元素的值)，start(起始索引，可选)，end(终止索引，可选)

``` bash
let arr = [1, 2, 3, 4, 5]
arr.fill(6, 2, 5)
console.log(arr) // [ 1, 2, 6, 6, 6 ]
```

* fill()方法用一个固定值填充一个数组中从起始索引到终止索引内的全部元素，不包括终止索引

> 参数：value(填充数组元素的值)，start(起始索引，可选)，end(终止索引，可选)

``` bash
let arr = [1, 2, 3, 4, 5]
arr.fill(6, 2, 5)
console.log(arr) // [ 1, 2, 6, 6, 6 ]
```

* entries()、keys()、values() , 三个数组遍历方法，返回一个遍历器对象，entries()键值对的遍历，keys()键名的遍历，values()键值的遍历

> 参数：无

``` bash
let arr = ['a', 'b', 'c']
for (let idx of arr.keys()) {
  console.log(idx) // 0 1 2
}
for (let item of arr.values()) {
  console.log(item) // 'a' 'b' 'c'
}
for (let [idx, item] of arr.entries()) {
  console.log(idx + '---' + item) // '0---a' '1---b' '2---c'
}
```

* includes()方法用来判断一个数组是否包含一个指定的值，如果包含返回true，否则返回false

> 参数：value(要查找的元素)，start(开始查找的位置，可选)

``` bash
let arr = ['a', 'b', 'c']
console.log(arr.includes('a')) // true
console.log(arr.includes('d')) // false
```

* flat()方法会按照一个指定的深度递归遍历数组，并将所有元素与遍历到的子数组中的元素和并到一个新数组中返回

> 参数：depth(要提取数组的嵌套结构深度，默认为1，可选)

``` bash
let arr = ['a', 'b', ['c', 'd']]
let result = arr.flat() // ["a", "b", "c", "d"]
let arr1 = ['a', ['b', ['c']]]
//使用 Infinity 作为深度，展开任意深度的嵌套数组
let result1 = arr1.flat(Infinity) // ["a", "b", "c"]
```

## 总结

此篇文章记录了JS中数组常用的方法，毕竟数组的实例方法有那么多，好记性不如烂笔头，写下此篇文章加深记忆，同时希望对大家也能有所帮助。如果有错误或不严谨的地方，欢迎批评指正!