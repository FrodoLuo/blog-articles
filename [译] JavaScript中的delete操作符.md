> 原文: [Understanding the Delete Operator in JavaScript - Chidume Nnambi](https://blog.bitsrc.io/understanding-the-delete-operator-in-javascript-3791ba6f3a08)

学习与理解`delete`操作符如何处理可变更与不可变更属性, 以及一些别的东西.

## delete 操作符

根据 ECMA 的定义与解释:

Delete(O, P)

这个方法常常被用来移除一些对象中的特定的属性. 如果属性本身是不可变更的, 那么它将抛出一个错误.

这个操作符在调用时被传入`O` - 要变更的对象, `P` - 要移除的属性的 key.

```javascript
let obj = {
  d: 88,
};

console.log(obj.d);
delete obj.d;
console.log(obj.d);
```

我们创建了一个包含`d`属性, 且值为 88 的对象`obj`. 首先我们打印输出`d`的值, 很显然会出现 88. 然后我们通过`delete`操作符来移除这个属性, 然后我们在打印它. 结果将会是`undefined`.

```sh
88
undefined
```

`undefined`是 JS 中用来表示*非值*的一个基本数据类型, 意味着数据被定义过了, 但尚未被赋值. 所以当通过`delete`删除了对象的一个属性之后, 这个属性的值就会变成`undefined`.

## 不可变更(non-configuration)属性与 delete

`delete`操作符只会对可变更(configuration)属性起作用. `delete`不能移除对象的一个不可变更的属性.

我们的`obj`对象是可变更的, 所以`d`属性能够被移除. 如果我们定义一个不可变更的属性, 那么`delete`就不能删除这个属性:

```javascript
let obj = {
  d: 88,
};

Object.defineProperty(obj, d, { configurable: false });

console.log(obj.d);
delete obj.d;
console.log(obj.d);
```

```sh
88
88
```

可以看到, 属性`d`并没有被移除. 我们通过`Object#defineProperty`方法把`d`属性设置为不可变更的, 只需要把`configurable`设置为`false`就行.

默认情况下, 对象的属性都是可变更的.

```javascript
let obj = {
  d: 88,
};

console.log(Object.getOwnPropertyDescriptor(obj, "d"));
```

```sh
{ value: 88,
  writable: true,
  enumerable: true,
  configurable: true }
```

之后如果通过`Object#defineProperty`方法来将`configurable`设置为`false`, 这个属性就不会被`delete`操作符删除了.

## var, let, const 与 delete

`var`, `let`, `const`声明的属性(变量)都是不可变更的, 因此它们声明的属性(变量)也不能通过`delete`来进行删除.

```javascript
const a = 1;
let b = 2;
var c = 3;
console.log(a, b, c);
delete a;
delete b;
delete c;
console.log(a, b, c);
```

```sh
1 2 3
1 2 3
```

函数与`delete`

函数(Function)也不能通过`delete`来进行删除

```javascript
function func() {
    console.log('inside func);
}

delete func;
func();
```

```sh
inside func
```

但是, 作为属性被定义的函数是可以被删除的

```javascript
var obj = {
  d: function () {
    console.log("inside obj.d function");
  },
};

obj.d();
delete obj.d;
obj.d();
```

```sh
inside obj.d function

TypeError: obj.d is not a function
```

与普通的属性一样, 如果这个属性被设置为不可变更的, 那么这个属性也不能被`delete`操作符删除

```javascript
const obj = {
  d: function () {
    l("inside obj d function");
  },
};
obj.d();
Object.defineProperty(obj, "d", { configurable: false });
delete obj.d;
obj.d();
```

```sh
inside obj d function
inside obj d function
```

## delete 与全局作用域

当我们不使用`var`或者`let`或者`const`定义了一个全局作用域下的变量时, 这个变量是可变更的

```javascript
f = 90;
console.log(Object.getOwnPropertyDescriptor(global, "f"));
```

```sh
{ value: 90,
  writable: true,
  enumerable: true,
  configurable: true }
```

因此他们是可以被`delete`移除的.

```js
f = 90;

console.log(f);

delete f;

console.log(f);
```

```sh
90

l(f)
  ^

ReferenceError: f is not defined
```

## delete 与局部作用域

`delete`不会影响局部作用于与函数作用域

```js
{
  var g = 88;
  delete g;
}
console.log(g);
function func() {
  var funch = 90;
  delete funch;
  console.log(funch);
}
func();
```

```sh
88
90
```

## delete 与不存在的属性

如果一个本身就不存在的属性被传递给`delete`, 它不会抛出一个错误, 而是会返回`true`

```js
var obj = {
  d: 90,
};

console.log(delete obj.f);
```

```sh
true
```

## delete 与原型链

`delete`不会影响到一个对象的原型链

```js
function Foo() {
  this.bar = 90;
}
Foo.prototype.bar = 88;

var f = new Foo();
```

我们创建构造器`Foo`, 并且在它的属性和原型链属性上都声明了一个属性`bar`, 它们同名但具有不同的值.

当直接引用这个对象是, `Foo`构造函数中定义的`bar`会被返回.

```js
f.bar; // 90
```

当我们删除了这个属性:

```js
delete f.bar;
```

他只会影响到`Foo`构造函数中定义的`bar`, 而不会影响到原型链中的. 当我们再次应用这个属性时, 原型链中的`bar`就会被返回

```js
console.log(f.bar);
delete f.bar;
console.log(f.bar);
```

```sh
90
88
```

## delete 与 JS 内建静态属性

`delete`操作符不能移除任何 API 内建的 API, 包括`Array`, `Math`, `Object`, `Date`等. 对这些属性进行`delete`操作会的到返回值`false`

```js
console.log(delete Math.PI);
```

```sh
false
```

## delete 与其在数列上的留洞性质(holey nature)

所有 JS 中的类型都继承自 JSObject 的 C++等价定义. 每一个我们定义的对象的属性, 都是 C++ JSObject 的一个成员

```js
obj = {
  d: 90,
  f: 88,
};
```

```c++
JSObject {
    d -> 90,
    f -> 88
}
```

数列也属于 JSObject, 只是他们之间存在一些差别. 差别在于, Array 的 JSObject 并不是由数列自己定义的, 而是通过数字排序定义的

```js
obj = [90, 88];
```

```c++
JSObject {
    0 -> 90
    1 -> 88
}
```

这也是为什么我们在引用数组时的方式`obj[1]`看起来与对象很类似的原因了. 在数组中, 这些数字就是它的属性.

在我们上述的数组中, 它有两个属性`0`和`1`.

类似我们对对象作出操作, 删除数组中的第一个属性(元素), 可以这么做:

```js
delete obj[0];

console.log(obj[0]); // undefined
```

但是这个操作并没有减少数组中元素的个数

```js
obj = [90, 88];
console.log(obj.length); // 2

delete obj[0];

console.log(obj.length); //2
```

```js
//obj
index:  0       1
    [       ,   88];
```

这样就在数列中留下了*孔洞*

实际上在对象中, `delete`操作也并不是完全抹除被删除的属性, 而是将它们的值设置为`undefined`. 可以通过对这些属性重新赋值来填满这些被留下的*孔洞*

## 总结

我们了解`delete`操作符是用来干什么的, 它对可变更与不可变更属性的影响, 它对全局与局部作用域的影响, 它对数组等*有洞*的属性的影响.

了解这些, 你能在使用`delete`操作符时变得更舒服, 更谨慎.

如果有任何问题, 请随意问我, 或者发一封邮件或者 DM

谢谢!!!
