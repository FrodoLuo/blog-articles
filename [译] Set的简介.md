> 原文: [The JavaScript Set Object](https://typeofnan.dev/the-javascript-set-object/) - Nick Scialli

`Set`时我最喜欢的一个 JavaScript 内建对象类型. 今天我将介绍`Set`对象, 并且讨论一下关于它的几种用法.

## Set 对象

`Set`对象是一组「值」的集合, 在其中可以存储一些**唯一的**的简单值或者对象的引用. **唯一性**是一个值得注意的关键--所有的值和对象引用都不能重复添加到 Set 中.

### 如何使用 Set

要使用 Set, 只需要创建一个 Set 的实例

```js
const mySet = new Set();
```

现在我们就有了一个空的 Set, 我们可以调用它的`add`方法来向其中添加一个数字`1`

```js
mySet.add(1);
```

怎样能知道我们已经往里面添加了一个`1`呢? 我们可以使用`has`方法来进行检查

```js
console.log(mySet.has(1))l
// true
```

让我们往其中添加一个对象的应用, 然后检查看看者的对象的引用是否在 Set 中

```js
const obj = { name: "Daffodil" };
mySet.add(obj);
console.log(mySet.has(obj));
// true
```

需要记住的一点是, 对对象引用进行检查时, 检查的是引用, 而不是对象里面的键名.

```js
console.log(mySet.has({name:'Daffodil}));
// false
```

我们可以通过`Set`的`size`属性来检查有多少个元素

```js
console.log(mySet.size);
// 2
```

继续, 我们能通过`delete`方法来移除某一个值

```js
mySet.delete(1);
console.log(mySet.has(1));
// false
```

最后, 我们能通过`clear`方法来清空`Set`

```js
mySet.clear();
console.log(mySet.size);
// 0
```

### 遍历一个 Set

最简单的方法是调用`Set`的`forEach`方法

```js
new Set([1, 2, 3]).forEach((el) => {
  console.log(el * 2);
});
// 2
// 4
// 6
```

`Set`对象同样拥有`entries`, `keys`与`values`方法, 这些同样能够用来遍历一个 Set, 但这就是这个教程之外的话题了.

### 实际中的使用例子

我发现`Set`对象对于跟踪记录二元状态变化是一个很好的东西. 一个很好的例子就是可折叠的 Accordion 菜单: 每个菜单中的 item 要么是展开的, 要么是收起的. 我们能创建一个叫`isOpen`的`Set`来跟踪记录这些 item 的状态, 然后使用一个`toggle`方法来开关这些状态

```js
const isOpen = new Set();

function toggle(menuItem) {
  if (isOpen.has(menuItem)) {
    isOpen.delete(menuItem);
  } else {
    isOpen.add(menuItem);
  }
}
```

### 关于效率

你可能会觉得`Set`对象跟数组十分相似, 但这里确实存在一个很大的能导致性能差异的不同. `Set`被用 Hash 映射实现.

在数组中存储数据时, 可能会遍历整个数组来找回他. 但在`Set`中, 查找过程是立刻的. 虽然实际上这些性能差异并不大, 但是当数据量足够大时, 这些差异就展现出来了.

## 总结

希望这能帮助你理解`Set`, 并且让你有了一个崭新的 JavaScript 的工具!
