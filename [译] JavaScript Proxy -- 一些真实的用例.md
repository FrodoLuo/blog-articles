> 原文: [Javascript Proxies: Real World Use Case](https://www.arbazsiddiqui.me/javascript-proxies-real-world-use-cases/) -- Arbaz Siddiqui
>
> 译者注, 为了防止出现"鲁棒性"这种因翻译习惯差异导致的混淆, 文中部分术语将不会进行翻译.

## Proxy 介绍

在编程术语范畴中, Proxy 指的是帮助/替代另一个实体(Entity)完成一系列操作的实体. 一个架设在客户端与服务端之间的 Proxy 服务器分别充当了*客户端的服务端*和*服务端的客户端*. 对于 Proxy 来说, 它们的任务就是介入收到的请求/调用, 并在处理后传递给其上游. 这些**介入**允许 Proxy 添加一些额外的业务逻辑或者改变整个操作的行为.

JavaScript 的 Proxy 从某种意义上来说是相似的. 它处在代码所操作的对象与实际被操作的对象之间进行处理.

根据 MDN Web 文档

> The Proxy is used to define custom behavior for fundamental operations (e.g. property lookup, assignment, enumeration, function invocation, etc).
>
> Proxy 被用来自定义一些基础层面的操作(例如属性查找, 赋值, 枚举, 函数调用等)

## 术语

在完成一个 Proxy 的使用之前, 有三个术语需要我们提前进行了解:

### Target(目标)

Target 就是实际被 Proxy 操作修改的对象. 它可以是任何一个 JavaScript 对象.

### Traps(阱)

> 译者注: 这个地方的翻译说实话有点不太好翻译, 但实际上只要能够理解所谓 Traps 就是用来重载(代理)Target 对应的名字的属性/方法的属性/方法就行

Traps 是指那些在 Target 的属性或者方法被调用时会介入干涉的方法. 有许多定义了的 Traps 可以被实现(implement)

### Handler(处理器)

Handler 是一个所有的 Traps 生存的占位对象. 简单来说, 可以把它当做一个存放且实现各个 traps 的对象.

我们来看看下面这个例子:

```JavaScript
//movie is a target
const movie = {
	name: "Pulp Fiction",
	director: "Quentin Tarantino"
};

//this is a handler
const handler = {
	//get is a trap
	get: (target, prop) => {
		if (prop === 'director') {
			return 'God'
		}
		return target[prop]
	},

	set: function (target, prop, value) {
		if (prop === 'actor') {
			target[prop] = 'John Travolta'
		} else {
			target[prop] = value
		}
	}
};

const movieProxy = new Proxy(movie, handler);

console.log(movieProxy.director); //God

movieProxy.actor = "Tim Roth";
movieProxy.actress = "Uma Thurman";

console.log(movieProxy.actor); //John Travolta
console.log(movieProxy.actress); //Uma Thurman
```

输出如下

```shell
God
John Travolta
Uma Thurman
```

上面这个例子中, `movie`就是我们所说的 Target. 我们实现了一个拥有`set`和`get`这两个 trap 的`handler`. 在其中我们添加了两个逻辑: 在访问`director`时, `get`这个 trap 会直接返回`God`而不是它实际的值; 在对`actor`赋值时, `set`这个 trap 会干涉所有的赋值操作, 并在键为`actor`时将值改变成`John Travlota`.

## 真实的案例

虽然并不如其他的 ES2015 的特性那样广为人知, Proxy 还是有诸如所有属性的默认值这样的现在看来挺亮眼的用例. 让我们来看看其他的在真实生产环场景中能够利用 Proxy 的地方.

### 验证 Validation

既然我们已经可以干涉对象的属性赋值过程, 那么我们可以借此来校验我们将要赋予给对象属性的值. 看下面这个例子

```javascript
const handler = {
  set: function (target, prop, value) {
    const houses = ["Stark", "Lannister"];
    if (prop === "house" && !houses.includes(value)) {
      throw new Error(`House ${value} does not belong to allowed ${houses}`);
    }
    target[prop] = value;
  },
};

const gotCharacter = new Proxy({}, handler);

gotCharacter.name = "Jamie";
gotCharacter.house = "Lannister";

console.log(gotCharacter);

gotCharacter.name = "Oberyn";
gotCharacter.house = "Martell";
```

运行结果如下:

```shell
{ name: 'Jamie', house: 'Lannister' }
Error: House Martell does not belong to allowed Stark,Lannister
```

上面这个例子中, 我们严格限制了`house`这个属性所能被赋予的值的范围. 只需要创建一个`set`的 trap, 我们甚至能用这个实现方式来实现一个只读的对象.

### 副作用 Side Effects

我们可以通过 Proxy 来创建一个在读写属性时的副作用. 出发点在于某些特定的属性被访问或者写入时触发一些函数. 看下面这个例子:

```javascript
const sendEmail = () => {
  console.log("sending email after task completion");
};

const handler = {
  set: function (target, prop, value) {
    if (prop === "status" && value === "complete") {
      sendEmail();
    }
    target[prop] = value;
  },
};

const tasks = new Proxy({}, handler);

tasks.status = "complete";
```

运行结果如下:

```shell
sending email after task completion
```

这里我们干涉了`status`这个属性的写入. 当写入的值是`complete`时, 会触发一个副作用函数. 在[Sindre Sorhus](https://github.com/sindresorhus)的[on-change](https://github.com/sindresorhus/on-change)这个包中就一个很 Cooooooool 的实现.

### 缓存 Caching

利用介入干涉对象属性读写的能力, 我们能够创建一个基于内存的缓存. 它只会在值过期前返回值. 看下面这个例子:

```javascript
const cacheTarget = (target, ttl = 60) => {
  const CREATED_AT = Date.now();
  const isExpired = () => Date.now() - CREATED_AT > ttl * 1000;
  const handler = {
    get: (target, prop) => (isExpired() ? undefined : target[prop]),
  };
  return new Proxy(target, handler);
};

const cache = cacheTarget({ age: 25 }, 5);

console.log(cache.age);

setTimeout(() => {
  console.log(cache.age);
}, 6 * 1000);
```

运行结果如下:

```shell
25
undefined
```

这里我们创建了一个函数, 并返回一个 Proxy. 在获取 target 的属性前, 这个 Proxy 的 handler 首先会检查 target 对象是否过期. 基于此, 我们可以针对每个键值都设置一个基于 TTLs 或者其他机制的过期检查.

## 缺点

虽然 Proxy 具备一些很神奇的功能, 但在使用时仍然具有一些不得不小心应对的限制:

1. 性能会受到显著的影响. 在注重性能的代码中应该避免对 Proxy 的使用
2. 没有办法区分判断一个对象是一个 Proxy 的对象或者是 target 的对象
3. Proxy 可能会导致代码在可读性上面出现问题

## 总结

Proxy 很强, 在很大范围内都能够得到应用, 或者被滥用. 这篇文章中我们讨论了什么是 Proxy, 如何实现一个 Proxy, 几个真实案例中的用例, 以及它的缺陷限制.
