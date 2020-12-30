# 质量更好的 JavaScript 编码实践方法

> 译者注: 实际上编写好的代码的基本原则不外乎就是书写**可读性强**, **可预期**, **可控**的代码. 本文所介绍的方法与技巧不外乎都是围绕这几点来进行的. 并且相比于书写方法上的注意事项与相关规范, 本文更多地强调了使用 TS 与 JS 的新特性来解决这些问题. 相比所谓的规范和注意事项, 译者认为这也的确是更为可靠的实践方法.

> 原文链接: https://stackoverflow.blog/2019/09/12/practical-ways-to-write-better-javascript/

![Cover](/api/uploads/cd8b090fba9d41618a6deb612466dfbc.png)

> 在 2019 年的开发者调查问卷中, 我们向用户征求了希望在*Stack Overflow*上面能看到除了问答的内容的意见. 其中最具备人气的一项是"其他开发者写的技术文章", 所以从现在开始, 我们会定期地推送一些贡献者们所撰写的文章. 如果你有一个想法想要提交投稿, 你可以向 pitches@stackoverflow.com 投递稿件.

嘿! 我是 Ryland Goldstein, 就职于*Binaries*进行一些重构工作. 这是我在 Stack Overflow 的第二篇投稿, 让我们来看看吧!

我没怎么看到有人在讨论提高 JavaScript 代码质量的一些实践性方法. 这里有一些我实际使用的提高 JS 代码质量的一些靠前的方法.

## 使用 TypeScript

你所能做出的排名第一的去提高 JavaScript 的方法就是不去写 JavaScript. 简单来说, TypeScript(TS)是一个*编译后*的 JS 超集(任何在 JS 里面能够运行的东西, 在 TS 里面都能运行). TS 在原有的 JS 基础之上添加了**全面而可选**的类型系统. 很长一段时间内, TS 所支持的技术生态的残缺性都让我找不到合适的理由来推荐 TS. 但终于这些日子一去不复返了. 现在大多数的框架和技术都开始了对 TS 的支持.

现在我们都知道了 TS 是什么, 接下来让我们来讨论讨论什么情况下你会想要使用 TS

### TypeScript 强化了类型安全

**类型安全**用来形容编译器检查一段代码中所有的类型与引用是否合法且正确的过程. 换句话说, 如果你创建了一个接收一个数字作为参数的函数`foo`:

```typescript
function foo(someNum: number): number {
  return someNum + 5;
}
```

`foo`函数应当且只应当被传入一个数字调用

```ts
// good
console.log(foo(2));

// not good
console.log(foo("two")); // 非法的TS代码
```

除了上述在代码中添加类型的一些额外代码量的开销之外, 添加类型安全支持几乎是没有负面作用的. 其另一方面的好处已经大到不能被忽视了. 类型安全提供了更高的级别的保护来防止通常容易因疏忽而出现的错误和 Bug, 通常这些错误都伴随着相对自由的开发语言出现, 比如 JS.

### TypeScript 使得重构大型项目成为可能

重构大型 JS 应用可以算得上是一大梦魇了. 重构 JS 最大的痛点来自于 JS 并没有强制要求函数签名(_function signatures_). 这意味着滥用 JS 函数是要出事情的.

> 译者注: 函数签名 *function signatures*指的是对函数的输入与输出的定义, 包括但不限于参数, 返回值, 可能会抛出的异常等. 例如对于上文中提到的`foo`函数来说, 其函数签名就是 function foo(someNum: number): number; 具体形式不受限制, 根据各个语言的不同有所不同.

打个比方来说, 你写了一个`MyAPI`函数, 这个函数还被成千上百个不同的服务调用:

```javascript
function myAPI(someNum, someString) {
  if (someNum > 0) {
    leakCredentials();
  } else {
    console.log(someString);
  }
}
```

随后, 你在某一处稍微改动了一下这个函数的签名:

```js
function myAPI(someString, someNum) {
  if (someNum > 0) {
    leakCredentials();
  } else {
    console.log(someString);
  }
}
```

开发者必须 100%地严格保证每一处调用都正确地改变了过来, 否则哪怕只有一个地方漏掉了, credential 信息也会遭到泄露.

现在让我们来看看在 TS 里面, 这种情况是怎么处理的:

```ts
// before
function myAPITS(someNum, number, someString: string) { ... }

// after
function myAPITS(someString: string, someNum: number) { ... }
```

显然, 我们针对`myAPITS`这个函数做出了与 JS 代码中相同的改变. 但是与 JS 最终会产生一个"合法的"JavaScript 代码不同, 这段代码在 TS 当中是不合法的 -- 成千上百处的调用中都传递了错误的, 与函数签名不相符合的参数. 由于类型安全会阻止这段代码进行编译, 我们能够提早地发现这些错误并加以改正.

### TypeScript 使得团队架构间的交流更为容易

当 TS 被正确的设置之后, 不进行前期设计类与接口而直接进行开发会变得比较困难. 因此它也提供了一个去与团队内部人员分享架构原则, 概念与设计的动机与途径. 在 TS 之前, 已经存在其他的解决方案来处理这些问题, 但它们都没有*原生*地解决这些问题, 都将会带来一些额外的工作量. 举个例子, 如果我想要为后端接口定义一个`Request`类型, 我可以通过向团队成员发送这些 TS 代码

```ts
interface BasicReqeust {
  body: Buffer;
  headers: { [header: string]: string | string[] | undefined };
  secret: Shhh;
}
```

在已经编写了 TS 代码的情况下, 不用再额外花费时间与精力去额外定义接口. 无论如何, 不管 TS 是否会比 JS 带来更少的 Bug, TS 强迫开发者在编码前进行类型和接口定义的行为肯定能带来质量更可靠的代码. 总的来说, TS 已经从 JS 进化成为了一个成熟, 更具可预测性的语言. 对开发者来说当然用着 JS 是一件更舒服的事情, 但最近的新项目, 我已经在项目一开始使用 TS 了.

## 使用现代 JS 的特性

JS 是当今世界最流行的编程语言之一. 你可能认为一个超过了 20 年寿命, 被成百上千万人使用的语言发展到现在一定是被大多数人所熟知的, 然而事实恰恰相反. 在最近的一段时间内, JS(或者说 ECMAScript)经历数次改动和功能特性的添加. 这些改动正在逐渐地改变开发者的实际体验.

### async 与 await

很长一段时间内, 异步的, 事件驱动的回调函数是 JS 开发不可避免的一环(~~不爽不要玩!~~)

```js
// 传统的回调函数
makeHttpRequest("google.com", function (err, result) {
  if (err) {
    console.log("Opps, It is an err");
  } else {
    console.log(result);
  }
});
```

我想应该不需要再花费时间来解释为什么这种回调函数的方式会最终导致问题([但我解释过](https://www.cdevn.com/parallel-computing-simplified-starring-gordon-ramsay/?_blank)). 为了解决这些回调函数的问题, 一个新的概念--_Promise_--被引入了 JS. *Promises*允许开发者书写异步的逻辑, 同时避免了嵌套的回调函数的问题.

```js
// Promise
makeHttpRequest("google.com")
  .then(function (result) {
    console.log(result);
  })
  .catch(function (err) {
    console.log("Opps, It is an err");
  });
```

Promise 相比回调函数最大的优势在于它的可读性和链式调用性. 虽然 Promise 很棒, 但仍然有一些东西值得期待. 对许多人来说, Promise 的体验仍然与回调函数有着相似的地方.

值得一提的是, 开发者们正在寻求一种 Promise 的另一种可行的使用方式. 为了解决这个问题, ECMAScript 委员会决定添加一组新的规范来使用 Promise, `async`与`await`

```js
// async and await
try {
  const result = await makeHttpRequest('google.com');
  console.log(result);
} catch (err) [
  console.log('Opps, It is an error');
]
```

需要注意的是, 所有使用了`await`的地方都需要声明`async`

```js
async function makeHttpRequest(url) { ... }
```

当然也可以直接 await 一个 Promise, 毕竟 async 函数本身就是对 Promise 的一个很棒的封装. 同时这也意味着*async/await*代码与 Promise 代码在功能上是相同的. 所以说完全可以自由地使用*async/await*.

> 译者注: 因为 async/await 是 es7 的特性, 所以在部分浏览器中的使用可能需要使用 babel 来编译. 但亲测 Chromium 至少在 76 版本已经内建支持了

### let 与 const

很长一段时间, 在 JS 中只存在一个变量作用域的定义`var`. 在如何处理作用域这个问题上, `var`具备一些非常独特/有趣的规则. `var`的作用域行为是反复无常, 令人混淆的. 而这也往往导致非预期的行为与 Bug. 但是从 ES6 开始, 对于`var`我们有了新的替换: `const`与`let`. 现在既然我们不再必须使用`var`了, 那么就不要用它了. 任何使用`var`的逻辑都能被转换为等效的使用`const`与`let`的代码.

至于 const 与 let 之间的选择, 我建议先使用 const 进行声明, 在必要对变量进行二次赋值的时候再使用 let, 总的来说, 大约只有 1/20 的变量使用了 let(甚至更少), 其余的全是 const. const 本身是更为严格且不可变更的. 这也通常能带来更好的代码质量.

相比于 C/C++中的 const, JS 中的 const 是存在一定差异的. 对于 JS 运行时来说, const 意味着*引用*不会发生变化, 并不代表其中的值和内容不会发生变化. 对于基本类型`string`, `number`, `boolean`来说, const 的确赋予了不可变更的特性(他们都是单纯的内存地址). 但是对于其他的对象`class`, `array`等, const 并不保证不可变性.

### 箭头函数

箭头函数是一个更简洁的声明匿名函数的方式. 匿名函数通常作为回调函数或者事件的钩子被传递.

```js
// 一段寻常的匿名函数
someMethod(1, function () {
  console.log("called");
});
```

大多数时候, 这样写没什么不对. 但需要注意, 这种普通的匿名函数在作用域(`this`等)上会有一些需要留意的地方, 而这通常会导致一些非预期的 Bug. 而在箭头函数中则不存在类似的混淆.

```js
someMethod(1, () => {
  console.log("called");
});
```

除了更为简洁的书写以外, 箭头函数在作用域上也存在一些更有用的特性. 箭头函数会从定义它的结构中继承`this`的引用. 需要澄清的一点是, 这并不同与`var`需要被废弃那样, 使用普通的匿名函数仍然是有效合法的. 但经常性地默认地使用箭头函数能节约大量的 Debug 的精力和时间.

### 展开运算符

将一个对象的所有键值对展开并添加到另一个对象当中是一个非常寻常的场景. 以往我们有几种方法来实现这些东西, 但往往这些方法都看起来有点啰嗦.

```js
const obj1 = { dog: "woof" };
const obj2 = { cat: "meow" };
const merged = Object.assign({}, obj1, obj2);
```

这种写法十分常见, 也十分的枯燥. 但有了展开运算符, 就不再需要这种写法了.

```js
const obj1 = { dog: "woof" };
const obj2 = { cat: "meow" };

const merged = { ...obj1, ...obj2 };
```

除此之外, 在数组中也能使用

```js
const arr1 = [1, 2];
const arr2 = [3, 4];

console.log([...arr1, ...arr2]); // [1, 2, 3, 4]
```

可能这并不是现代 JS 最重要的一个特性, 但我真的十分喜欢它.

### 模板字符串

字符串是最常见的程序结构之一. 但这也导致各个编程语言对字符串声明的支持之弱显得十分尴尬. 很长一段时间内, JS 都属于*辣鸡字符串*大家庭. 但是最近模板字符串的支持使得 JS 在字符串领域独树一帜. 模板字符串天然而方便地解决了两个在书写字符串时的最大的痛点: 动态内容, 多行书写.

```js
const name = "Ryland";
const helloString = `Hello
${name}`;
```

我认为这段代码已经足够说明这一切了. 这实现太强了

### 对象解构

对象解构是一个避免使用迭代的方式来展开获取数据集中数据的方法.

```js
// 老方法
function animalParty(dogSound, catSound) {}

const myDict = {
  dog: "woof",
  cat: "meow",
};

animalParty(myDict.dog, myDict.cat);

// 解构

function animalParty(dogSound, catSound) {}

const myDict = {
  dog: "woof",
  cat: "meow",
};

const { dog, cat } = myDict;
animalParty(dog, cat);
```

等等, 除此之外, 你也可以直接解构函数的参数签名

```js
function animalParty({ dog, cat }) {}

const myDict = {
  dog: "woof",
  cat: "meow",
};

animalParty(myDict);
```

数组也可以用

```js
[a, b] = [10, 20];

console.log(a); // 10
```

除此之外还有许多应该用起来的现代 JS 特性. 这有一些其他的对我来说很有帮助的特性:

- [剩余参数](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/rest_parameters?_blank)
- [`import`替代`require`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import?_blank)
- [数组元素查找](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import?_blank)

## 确保你的系统资源得到充分利用

> 译者注: 这部分有点高能

当进行并发处理时, 其目的旨在优化同时进行的任务数量. 假设有一个四线程的处理器, 但是代码只利用了其中一个, 那么 75%的潜力都将被浪费. 这些意味着*阻塞*,*同步操作*是并发任务中最大的问题所在. 但考虑到 JS 是单线程语言, 并不存在多线程这个说法, 那么我们在讨论的是什么?

JS 是基于单线程的, 但并不是基于单文件的. 虽然它并不是多线程的, 但它仍然存在一些"同时"的情况. 发送并接收到 HTTP 的请求通常会花上几秒钟甚至几分钟, 如果 JS 直到拿到请求前都停止执行发生阻塞, 那么这个语言基本上可以说是废了.

JS 通过`事件循环 Event Loop`来解决这个问题. 事件通过基于时钟和优先级的逻辑的事件注册和执行来完成循环过程. 这使得同时处理上千个 HTTP 请求或者同时读取多个文件成为可能. 重点: JS 只有在正确使用这些特性是, 才能利用这些能力. 一个最简单的例子是`for-循环`

```js
let sum = 0;
const myArray = [1,2,3,4,5...99, 100];
for (let i = 0, i < myArray.length; i += 1) {
  sum += myArray[i];
}
```

一个原始的 For 循环是编程过程中出现的最不并发的结构. 在我上一份工作中, 我带着我的团队花费了几个月的时间试图把一个传统的 R 语言循环转换为可以自动并发执行的代码. 这几乎是一个不可能完成的任务, 直到我们深入研究学习了才能解决. 对一个循环结构进行并发操作的难点在于一些有问题的写法形式. 线型顺序的循环是非常少见的, 但它们也就使得拆分 for 循环成为不可能.

```js
let runningTotal = 0;
for (let i = 0; i < myArray.length; i += 1) {
  if (i === 50 && runningTotal > 50) {
    runningTotal = 0;
  }
  runningTotal += Math.random() + runningTotal;
}
```

如果这段代码的循环是被一个一个顺序执行的, 那么产生的结果始终是按照我们期望的样子. 但如果有多个迭代器来迭代, 处理器可能会因为并不准确的变量值而产生不同的结果, 最终的结果可能不会符合我们的预期. 如果是 C 语言代码, 那我们倒是有很多东西可以说, 也很有一些编译器能对循环做的一些小技巧. 在 JavaScript 里面, 传统的 for 循环, 应当且仅应当不得不去使用它的时候再去用它. 其他的时候使用这些结构:

```js
// Map
// 降低关联
const urls = ['google.com', 'yahoo.com', 'aol.com', 'netscape.com'];
const resultingPromises = urls.map(url => makeHttpRequest(url));
const results = await Promsie.all(resultingPromises);

// Map with Index
// 降低关联
const urls = ['google.com', 'yahoo.com', 'aol.com', 'netscape.com'];
const resultingPromises = urls.map((url, index) => makeHttpRequest(url, index));
const results = await Promise.all(resultingPromises);

// forEach
const urls = ['google', 'yahoo.com', 'aol.com', 'netscape.com'];
// 注意 这里是非阻塞性的
urls.forEach(saync url => {
  try {
    await makeHttpRequest(url);
  } catch (err) {
    console.log(`${err} bad practice`);
  }
});
```

我会解释为什么这些结构对于传统的 for 循环是一个提升. 与传统 for 循环对每一个元素进行顺序迭代不同, 诸如`map`这样的结构将所有元素一次性拿出来, 并且将他们每一个作为独立的"事件"分发给开发者定义的`map`函数中. 大多数时候, 这些独立的部分互相之间没有依赖, 也就允许他们能够同时执行. 当然这不是说在 for 循环里面就做不到这些事情. 实际上看起来是像这样的

> 译者按: 实际上这里是一种"函数式编程"的思想, 即函数本身不带有副作用, 对于参数相同的多次执行, 其结果是相同的

```js
const items = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9];

async function testCall() {
  // 做一些异步的事情
}

for (let i = 0; i < 10; i += 1) {
  testCall();
}
```

可以看到, for 循环实际上没有阻止你这么干, 但实际上, 用`map`的话会让代码看起更加简洁

```js
const items = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9];

item.map(async (item) => {
  // 做一些异步的事情
});
```

可以看到, `map`也能做到. 如果想要在所有的异步函数执行完毕之前对程序进行一个阻塞, 那么`map`的优点会变的更加明显.

```js
const items = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9];
const allResults = await Promsie.all(
  items.map(async (item) => {
    // 做一些异步的事情
  })
);
```

有许多案例和使用场景中, for 循环也能与跟`map`这些结构一样出色, 甚至更优秀. 但我仍然认为, 利用这些完善定义的 API 的好处, 即使少写一些循环语句也是值得的. 这样的话, 将来的任何在这些 API 上的提升都能使你的代码也同样得到好处. for 循环实在是太常见了, 它也值得有一些具有意义的优化和提升.

除了`map`和`forEach`, 也有注入`for-await-of`这样的异步结构

## 检查代码, 强制代码风格

没有统一风格的代码读起来是相当痛苦的. 因此, 无论哪个开发语言, 一个书写高质量代码的关键就是坚持统一的代码风格习惯. 由于 JS 生态的活跃, 我们有一大堆能用来检查代码风格的选择. 我只是想要强调一点: 用工具来检查和强制代码风格比用哪个工具或者哪个风格要重要得多. 天底下很难有人跟自己用的是完全一致的代码. 在这方面优化意义不大.

有很多人问我, 应该用哪个东西, `eslint`还是`prettier`. 对我来说, 他们的目的其实并不相同. 因此他们更应该结合起来使用. ESLint 是一个大多数时候适用的代码的检查器(linter), 更多的时候, 它更多的是查找并解决代码中的问题, 而很少对代码风格下功夫. 举个例子, 如果适用`Airbnb`的规则, 那么下面这行代码则会让 linter 报错

```js
var fooVar = 3; // Airbnb规则禁止了var的使用
```

ESlint 是怎么影响开发过程已经很明显了. 关键点在于它让开发者遵循一系列规定哪些实践是好的, 哪些是不好的规则. 也因此, Linter 实际上是有一些"固执"的, 当所有的固执集合起来, Linter 可能会犯错误.

Prettier 是一个很好的代码格式化工具. 它更少关心代码的正确与否, 而更关注与代码风格的一致性和统一性. Prettier 不会因为使用`var`而报错, 它所做的更多的是对其括号这类格式化上面的事情. 在我个人的使用习惯中, Prettier 通常是最后一步的工作. 如果让 Prettier 在每次 commit 的时候自动运行, 那就更棒了. 这能保证 repo 里面的代码的一致性和统一性.

## 测试代码

编写测试代码是一个间接的但很重要的提升代码质量的方面. 我推荐习惯尽可能多的测试工具, 毕竟测试需求是在不停的变动的. 没有哪一个单独的工具能够解决所有的这些事情. 有许多非常棒的 JS 测试同居. 根据个人的口味选择一些.

- 测试驱动 - [Ava](https://github.com/avajs?_blank)

- 埋点与打桩 - [Sinon](https://github.com/sinonjs/sinon?_blank)

- Mock - [Nock](https://github.com/nock/nock?source=post_page---------------------------?_blank)

- Web 测试自动化 - [Selenium](https://github.com/SeleniumHQ/selenium?_blank)

> 译者: 这部分工具就暂时不翻译了, 感兴趣可以去主页看看

## 不要停下来啊

书写更高质量的 JS 代码是一个持续的过程. 代码总是能够变得更加整洁, 新特性总是在不停添加, 测试用例永远不嫌多. 这看起来有点过头了, 但这也是因为总是存在值得提升的东西. 一点一滴地进步, 总是能在不知不觉的时候成为 JavaScript 的王牌.
