---
title: 关于 JS 的继承
date: 2019-07-25 18:01:29
tags: "JavaScript" "前端开发"
---

## 序言
最近从某个大佬的github博客上看到一个关于js继承的博客，先放上来供大家参考：

JavaScript深入之继承的多种方式和优缺点[https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Fmqyqingfeng%2FBlog%2Fissues%2F16]

看完之后，总结了几个点：

## 为什么说寄生组合式继承是最优的？

作者引用了高程的解释：

引用《JavaScript高级程序设计》中对寄生组合式继承的夸赞就是：
这种方式的高效率体现它只调用了一次 Parent 构造函数，并且因此避免了在 Parent.prototype 上面创建不必要的、多余的属性。与此同时，原型链还能保持不变；因此，还能够正常使用 instanceof 和 isPrototypeOf。开发人员普遍认为寄生组合式继承是引用类型最理想的继承范式。

当然，这个肯定是没有问题，我们在延伸一点东西出来：

这里为了方便，我把相关代码一起贴过来，供参考

### 涉及原型链继承的问题

代码如下：

``` bash
function Parent () {
    this.names = ['kevin', 'daisy'];
}

function Child () {}

Child.prototype = new Parent();

var child1 = new Child();

child1.names.push('yayu');

console.log(child1.names); // ["kevin", "daisy", "yayu"]

var child2 = new Child();

console.log(child2.names); // ["kevin", "daisy", "yayu"]

```

js的面向对象是基于原型和原型链的，不像java这种语言，java中的继承会真正生成一个与父类完全无关的子类，子类new出来的实例是一个单独的实例，不论你new多少个都是隔离的，然而js并不是这样的，熟悉js的小伙伴都知道，用原型链继承会导致一个很大的问题，就是“共享父类属性”的问题，所有的子类实例会共享一个属性，就好比你有苹果这个属性，但是你的苹果实际上并不是你的，是你从父类那里继承过来的，而且，父类可以吃掉你的苹果，你的兄弟姐妹们也可以吃掉你的苹果，好吧，想想就可怕。

### 使用单纯的调用父类构造函数继承的问题

代码(有改动)：

``` bash
function Parent () {
    this.names = ['kevin', 'daisy'];
    this.getNames = fucntion(){
        return this.names;
    }
}

function Child () {
    Parent.call(this);
}

var child1 = new Child();

child1.names.push('yayu');

console.log(child1.getNames()); // ["kevin", "daisy", "yayu"]

var child2 = new Child();

console.log(child2.getNames()); // ["kevin", "daisy"]

```

可以看到共享的问题解决了，但是，有一个额外的问题，我们是基于原型链的，但是我们并没有真正的去利用原型链的共享功能，完全抛弃了它，并且导致每次new 实例的时候，都会去调用父类的构造方法去加到子类的实例上，是完全的copy paste过程，这等于舍弃了js原型链的精髓部分，这样的代码自然是没有灵魂的~

### 组合继承？

代码：

``` bash
function Parent (name) {
    this.name = name;
    this.colors = ['red', 'blue', 'green'];
}

Parent.prototype.getName = function () {
    console.log(this.name)
}

function Child (name, age) {

    Parent.call(this, name);
    
    this.age = age;

}

Child.prototype = new Parent();
Child.prototype.constructor = Child;

var child1 = new Child('kevin', '18');

child1.colors.push('black');

console.log(child1.name); // kevin
console.log(child1.age); // 18
console.log(child1.colors); // ["red", "blue", "green", "black"]

var child2 = new Child('daisy', '20');

console.log(child2.name); // daisy
console.log(child2.age); // 20
console.log(child2.colors); // ["red", "blue", "green"]

```

在new Child的过程中会调用一次父类的构造方法，并且在指定Child的原型的时候，又会调用一次，这明显会造成一些问题，child实例上有一个自己的name，同时还有一个父类给的name，这明显不是最优的方案~

### 寄生组合

代码：

``` bash
function Parent (name) {
    this.name = name;
    this.colors = ['red', 'blue', 'green'];
}

Parent.prototype.getName = function () {
    console.log(this.name)
}

function Child (name, age) {
    Parent.call(this, name);
    this.age = age;
}

// 关键的三步
var F = function () {};

F.prototype = Parent.prototype;

Child.prototype = new F();


var child1 = new Child('kevin', '18');

console.log(child1);

```

首先要理解什么是寄生，原本通过直接把父类的实例对象指给子类的原型的方式中，存在一个比较尴尬的问题，子类可以拿到父类以及父类原型的所有属性，同时组合模式下，子类上拥有的属性在父类也会有，因此我们用一个中间类，这个函数是一个空函数，我们就用它来把原型链串起来，但是不给他任何属性，这样子类的原型通过这个中间的F的实例，就可以直接访问到原本想要访问的父类属性了，同时new F() 的过程避免了重复调用原本父类的过程，比起new Parent()， new F()的性能会更好，副作用也更少，核心思想就是：

属性，比如name，colors 不需要共享的，通过调用父类构造函数

方法，比如getName，需要共享的可以直接设置到父类的原型对象上

那么，你以为，这真的是最优的js继承方式吗？

## 到底有没有最优的继承方式?

### 从“对象”谈起

任何一个自然界的对象，大方面都会有2种属性，一种静态属性【属性】，一种动态属性【方法或者行为】，比如对于一个自然人，名字，年龄，性别，地址等等，都是静态属性，它们是不可以被共享的，还有动态属性，比如：说话，走路，吃东西，等等叫做动态属性，那么我们在设计继承的时候，到底应该怎么划分？每个人的静态属性隔离，动态属性共享，就够了吗？

比如：每一个人都有鼻子，有眼睛，有头有脑，这些静态属性应该隔离吗？

再比如：羽毛球运动员都会打羽毛球，乒乓球运动员都会打乒乓球，这些动态属性都应该共享吗？

当然不是。

### 最优解

继承的最优解其实是要看当前应用场景的，最符合预期的场景就是，需要共享的，无论是静态的还是动态的，把它们放在parent的原型上，需要隔离的，把它们放在parent上，然后子类通过调用parent的构造方法来初始化为自身的属性，这样，才是真正的“最佳继承设计方式”。

## 总结：

当面试者问你的时候，我觉得没必要答什么寄生组合什么的【它只是个名字】，最优的继承方式：
对于当前需要被设计为共享属性的属性，全部通过设置原型的方式挂到父类的原型上，不分静态和动态，维度划分是是否可以共享，对于每个子类都不一样的属性，应该放到父类的构造方法上，然后通过子类调用父类构造方法的方式来实现初始化过程，对于子类独有的属性，我们通过扩展子类构造方法的方式实现，那么对于每一个子类如何拿到父类的原型方法，就需要将子类的构造方法的原型与父类构造方法的原型进行原型链关联的操作，看个具体的例子：

``` bash
const Man = function(name, age) {
    // 需要都要但是值不一样的属性，在父类的构造方法中定义
    this.name = name;
    this.age = age;
};

Man.prototype.say = function() {
    // 需要共享的动态属性
    console.log(`i am ${this.name}, i am ${this.age} years old`);
};

Man.prototype.isMan = true; // 需要共享的静态属性

const Swimmer = function(name, age, vitalCapacity) {
    Man.call(this, name, age); //继承“人类”
    this.vitalCapacity = vitalCapacity; // 对于游泳员来说肺活量是很重要的一个指标，是每个游泳员都需要但是值都不同的属性
};

const BasketBaller = function(name, age, height) {
    Man.call(this, name, age); //继承“人类”
    this.height = height; // 对于篮球运动员来说身高是一个都需要但是值都不同的指标
};

// 我们用es新的直接设置原型关系的方法来关联原型链
Object.setPrototypeOf(Swimmer.prototype, Man.prototype); // 设置子类原型和父类原型的原型链关系 达到共享原型上的属性的目的
Object.setPrototypeOf(BasketBaller.prototype, Man.prototype); // 同理

// 还可以继续扩展 Swimmer.prototype 或者 BasketBaller.prototype 上的公共属性哦

const swimmer1 = new Swimmer('swimmer1', 11, 100);
const swimmer2 = new Swimmer('swimmer2', 12, 200);

swimmer1.isMan // true 共享静态属性
swimmer1 // age: 11, name: "swimmer1", vitalCapacity: 100 自身属性有了
swimmer1.say() // i am swimmer1, i am 11 years old 共享动态属性

const basketBaller1 = new BasketBaller('basketBaller1', 20, 180);
const basketBaller2 = new BasketBaller('basketBaller2', 30, 187);

// 等等等。。。

```