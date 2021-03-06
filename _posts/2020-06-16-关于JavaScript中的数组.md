---
layout: post
title: 关于JavaScript中的数组
subtitle: Array的方法
date: 2020-06-16
author: CHM
header-img: img/post-bg-desk.jpg
catalog: true
tags:
  - JS
  - Array
---

## 问题

#### 判断一个变量是否是数组

```
Array.isArray([]) // true

Object.prototype.toString.call([]) // "[object Array]"

[] instanceof Array / [].constructor === Array; // 存在弊端，不可靠 原因是Array实质是一个引用，用instanceof方法（包括下面的constructor方法）都是利用和引用地址进行比较的方法来确定的，但是在frame嵌套的情况下，每一个Array的引用地址都是不同的，比较起来结果也是不确定的，所以这种方法有其局限性。

```

#### 数组的原生方法有哪些？

`改变自身的方法`

- shift();
- unshift();
- push();
- pop();
- reverse();
- sort();
- splice(); //删除 or 替换 params: start , deleteCount（可选,如果不指定，那么 start 之后数组的所有元素都会被删除。） , item...（可选，如果不指定，将只删除数组元素）
- copyWithin();
- fill();

`不改变自身的方法`

- concat();
- includes();
- join();
- slice(begin,end); // 包含 begin 不包含 end
- flat(); // 使用 Infinity，可展开任意深度的嵌套数组 & flat()方法会移除数组中的空项:
- toString();
- indexOf();
- lastIndexOf();

`遍历`

- map();
- filter();
- forEach();
- find();
- findIndex();
- keys();
- values();
- some();
- every();
- reduce();

#### 如何将类数组转化为数组

```

// es6
Array.from(类数组);
[...类数组] // 扩展运算符

[].slice.call(类数组) / Array.prototype.slice.call(类数组);

遍历
var args = [];
for (var i = 1; i < arguments.length; i++) {
args.push(arguments[i]);
}

```

#### 说一说 ES6 中对于数组有哪些扩展

- 增加了扩展运算符（spread）...;

- 增加了两个方法，Array.from()和 Array.of()方法。

- 增加了一些实例方法，如 copyWithin()、entries（）、keys()、values（）,includes() 等。

#### 数组去重

```

// es6
const arr = [1,2,"2",2,3,5,32,2,24]
const newArr = [...new Set(arr)] / Array.form(new Set(arr));

// filter
// 排序去重的方法
function unique(array) {
return array.concat().sort().filter(function(item, index, array){
return !index || item !== array[index - 1]
})
}
// indexOf()
function unique(array) {
var res = array.filter(function(item, index, array){
return array.indexOf(item) === index;
})
return res;
}

//for 循环 includes()/indexOf();
function unique(data){
const arr = [];
for(let i = 0;i<data.length;i++){
if(!arr.includes(data[i])){ // arr.indexOf(data[i]) === -1
arr.push(arr[i])
}
}
return arr;
}

// Object 键值对
function unique(data){
const obj = {};
return data.filter((item,index,arr)=>{
return obj.hasOwnProperty(typeof item + item) ? false : (obj[typeof item + item] = true)
})
}

// 双层循环

```

#### 如何“打平”一个嵌套数组，如[1,[2,[3]],4,[5]] => [1,2,3,4,5]?你能说出多少种方法？

```

const arr = [1,[2,[3]],4,[5]];

// lodash.js
const newArr = \_.flattenDeep(arr)

const newArr = JSON.parse(`[${arr.toString()}]`) / JSON.parse(`[${arr.join()}]`)

const newArr = arr.flat(Infinity);

// reduce
function flatten(data){
return data.reduce((pre,next)=>{
pre.concat(Array.isArray(next)?flatten(next):next)
},[])
}

// ES6 扩展运算符 ...
function flatten(arr) {

    while (arr.some(item => Array.isArray(item))) {
        arr = [].concat(...arr);
    }

    return arr;

}

```

#### 如何克隆一个数组？你能说出多少种？

```

const arr = [1,2,3,4,3]

// 浅拷贝
const newArr = \_.clone(arr); // lodash.js
const newArr = arr.slice()/arr.concat();
const newArr = [...arr];
const newArr = Object.assign([],arr)

// 深拷贝
const newArr = \_.cloneDeep(arr); // lodash.js
const newArr = JSON.parse(JSON.stringify(arr)); // 不能拷贝函数

递归循环
var deepCopy = function(obj) {
if (typeof obj !== 'object') return;
var newObj = obj instanceof Array ? [] : {};
for (var key in obj) {
if (obj.hasOwnProperty(key)) {
newObj[key] = typeof obj[key] === 'object' ? deepCopy(obj[key]) : obj[key];
}
}
return newObj;
}

```

## Polyfill

#### `forEach`

```
if(typeof Array.prototype.forEach !== 'function'){
    Array.prototype.forEach = function (fn,context) {
        if(typeof fn === 'function'){
            for(let k = 0; k<this.length; k++){
                if(Object.hasOwnProperty.call(this,this[k])){
                    fn.call(context,this[k],k,this)
                }
            }
        }
    }
}
```

#### `map (纯函数)`

```
Array.prototype.map = function (fn,context) {
    let newArray = []

    if(typeof fn === 'function') {
        for (let k = 0, length = this.length ; k<length; k++){
        newArray.push(fn.call(this[k],k,this))
    }

    return newArray ;
}
```

#### `filter (纯函数)`

```
Array.prototype.filter = function (fn,context) {
    let newArray = []

    if(typeof fn === 'function') {
        for(var k = 0,length = this.length; k<length;k++){
            fn.call(this[k],k,this) && newArray.push(this[k])
        }
    }

    return newArray;
}
```

#### `some`

```
Array.prototype.some = function (fn,context) {
    let passed = false;
    if(typeof fn === 'function') {
        for(let k = 0,length = this.length; k<length;k++){
            if(passed === true){
                break;
            }
            passed = !!fn.call(this[k],k,this)
        }
    }

    return passed;
}
```

#### `every`

```
Array.prototype.every = function(fn,context){
    let passed = true;
    if(typeof fn === 'function'){
        for(ket k=0,length=this.length;k<length;k++){
            if(passed !== true){
                break;
            }
            passed = !!fn.call(this[k],k,this)
        }
    }

    return passed;
}
```

#### `indexOf`

```
Array.prototype.indexOf = function (searchEle,position) {
    let index = -1;
    position = position * 1 || 0

    for(var k=0,length = this.length;k<length;k++) {
        if(k>=position && this[k]===searchEle){
            index = k;
            break;
        }
    }

    return index
}
```

#### `reduce`

```
Array.prototype.reduce = function (callBack,initialVal) {
    let previous = initialVal;
    let k = 0,length = this.length;

    if(previous === undefined) {
        previois = this[0];
        k = 1
    }

    if(typeof callBack === 'function') {
        for (k;k<length;k++) {
            previous = callBack.call(this,previous,this[k],k,this)
        }
    }

    return previous
}
```
