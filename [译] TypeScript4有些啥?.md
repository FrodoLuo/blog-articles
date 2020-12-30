> 原文: [What's coming in TypeScript 4](https://httptoolkit.tech/blog/whats-coming-in-typescript-4)
>
> 作者: [Tim Perry](https://twitter.com/pimterry)

TypeScript4 来得很快, 这周(6.25)就有一个 Beta Release 的计划, 而最终的正式 release 将会在八月中旬到来.

需要注意的是, TypeScript 的 Release 并没有遵循 Semantic 版本规则进行, 所以 4.0 版本并不算是一个大型的更新.

> TypeScript never claimed to follow semantic versioning, in the sense that breaking changes imply major versions.
>
> TypeScript, however, promises no breaking changes after a stable release. so no breaking changes between 2.1.5 and 2.1.6, 2.1.\*.
>
> TypeScript 从来没有说过遵循 Semantic 版本规则, 即断代更新会体现在主版本号变更上
>
> 相反, TypeScript 承诺在每一个 Stable Release 之间不会有断代更新, 因此在 2.1.5, 2.1.6, 2.1.\*...上面不存在断代更新

版本号跳到了 4.0 并不意味着这里有什么大型的断代, 这不是啥改变一切的 Release, 但他还是确确实实带来了一些关于类型的棒棒的改动. 对于像 HTTP Toolkit(完全是 TypeScript 开发的)的项目来说, 这次改动能够带来更快的开发速度以及更少的 bug.

让我们来深入一下细节的部分.

## 可变元组类型 Variadic Tuple Type

所谓"可变类型", 对于 TypeScript 的类型系统来说, 是一个复杂但是颇具重要性的新 Feature.

如果没有深刻理解类型理论的话, 解释这玩意儿会有点麻烦, 但是举个例子还是很简单的. 让我们试试用元组参数来书写一个 concat 函数:

```typescript
function concat(nums: number[], strs: string[]): (string | number)[] {
  return [...nums, ...strs];
}

let vals = concat([1, 2], ["hi"]);
let val = vals[1]; // 尽管我们知道这是一个number, 但TS仍然会提示这是一个number|string

// 但TS确实支持对这些值的精确的类型定义
let typedVals = concat([1, 2], ["hi"]) as [number, number, string];
let typedVal = typedVals[1]; // => 提示number, 没问题
```

在目前, 这段代码是合法的 TS 代码, 但并不是最优的.

这段代码中, `concat`能够正确地起作用, 但是我们会丢失一些类型, 而且如果想要在其他地方获得精确的类型的话, 就不得不在之后手动地进行修正. 目前还没有可能能够完全避免这些问题.

在有了可变元组之后, 我们可以这样写

```typescript
function concat<N extends number[], S extends string[]>(
  nums: [...N],
  strs: [...S]
): [...N, ...S] {
  return [...nums, ...strs];
}

let vals = concat([1, 2], ["hi"]);
let val = vals[1]; // => 提示类型 number
const val2 = vals[1]; // => 提示值 2, 而不只是类型number

// 在深入一点, 我们可以精确地concat任何类型
function concat<T extends unknown[], U extends unknown[]>(
  t: [...T],
  u: [...U]
): [...T, ...U] {
  return [...t, ...u];
}
```

本质上来说, 元组类型现在能够包含`...T`作为一个多个元组内类型的泛型占位. 可以通过此来形容一些未知类型(`[...T]`), 或者部分类型已知的元组(`[string, ...T, boolean, ...U]`).

TypeScript 能够在之后使用的过程中提示这些类型, 因此只需要在大体地对元组形状进行描述并在之后使用, 而不需要依赖具体的细节.

这是一种相对简洁的方式, 并且比简单地连接数组要来的更为广泛. 通过组合一些已经存在的可变函数, 例如`f<T extends unknown[]>(...args: [...T])`, 就能够把函数的参数当做数组来看待, 进而能够比现在更具弹性地去描述函数的参数格式.

举个例子, 目前对函数中**剩余/可变参数**的描述必须始终放在函数参数描述的末尾, `f(a: number, ...b:string[], c: boolean)`便是一个无效的例子

在这一次的升级之后, 通过在函数参数定义中使用可变元组类型, `f<T extends string[]>(...args: [number, ...T, boolean])`便能使上述的例子成为可能

看起来有点抽象, 实际上可以这么理解, 我们能通过这一特性来做到:

- 解构数组类型:
  `type head = <H extends unkone, T extends unknonw[]>(list: [H, ...T]) => H`
- 对任意长度的数组执行类似映射类型才允许的操作, 而不仅仅是对象
- 对可变参数的函数进行完整的类型提示
- 对复杂的, 部分参数类型已知的可变参数进行正确的提示
- 对 Promisify 进行完整的类型定义
- 对诸如`curry`, `apply`, `compose`等高阶函数进行完整地参数类型描述
- 干掉所有的为了完整描述类型而产生的冗余代码.

就算现在没在写什么复杂的高阶函数, 改进类型也仍然能让我们在之后的能够更细节地去描述类型, 正确提示一些不明确的数组类型定义, 改进其他地方的类型提示.

## 带标注的元组 Labelled Tuples

这是一个跟上一个有关系, 但是要简单得多的特性: TypeScript 将允许给元组中的元素加上标注了.

看看下面这个函数的类型的描述, 你能从中获得什么信息?

```typescript
function getSize(): [number, number];
```

那这个呢

```typescript
function getSize(): [min: number, max: number];
```

这些标注会在运行时消失(编译时会被注释掉), 并且也不会做额外的类型检查. 它们能像上面那样使得元组的声明变得更加直观清晰.

对于剩余/可选参数同样适用:

```typescript
type MyTuple = [a: number, b?: number, ...c: number[]];
```

## 从构造函数的使用来推断属性类型

一个简明的类型提示的改进

```typescript
class X {
  private a;

  constructor(param: boolean) {
    if (param) {
      this.a = 123;
    } else {
      this.a = false;
    }
  }
}
```

在上面这段代码中, 目前的 TS 版本中, `a`会被认为是`any`的类型(甚至还会在`noImplicitAny`开启时报错). 属性的类型只会在直接初始化的时候得到推断. 因此需要一个初始化函数, 或者直接对其进行定义.

在 TypeScript4 中, `a`的类型会被推断为`number | boolean`: 从构造函数自动推断.

如果这种机制还不能满足, 仍然能够通过精准定义的方式来对属性进行类型声明, 并且当这类声明存在时, 他们会被更优先地使用.

## 短路赋值操作符

对类型方面的改进不感兴趣? 没问题, TypeScript4.0 同时实现了处于**Stage3**的 JS 特性: 逻辑运算赋值. 新的语法得到支持, 并会被编译到老的环境中也能运行的形式.

看起来就像这样

```typescript
a ||= b;
// 等于: a = a || b

a &&= b;
// 等于: a = a && b

a ??= b;
// 等于: a = a ?? b
```

目前, 最后一个选项可能是最有用的, 除非正儿八经地在进行布尔运算, 那这个合并运算对于默认值和错误回落值是个很完美的解决方案.

## 一些其他的部分

上面都是些主要的改动, 下面是一些其他的也很不错的部分:

- `unknown`现在能够做为`try..catch...`中的类型标注
  `try {...} catch (e: unknown) {...}`
- 支持了 React 的新的**JSX 内部**(译注: children 之类的)
- 编辑器对`@deprecated`JSDoc 注解的支持
- 在 3.9 版本的性能提升后的性能提升
- 新的编辑器可用的代码重构(比如自动的用可选链 Optional Chaining 进行重构), 改进了一些重构(更好的 auto-import), 以及一些语法高亮

上述这些改动都不是大型的改动, 但也值得重视. 至少它们帮着 TypeScript 的程序员们续命了 -- 改善了类型安全以及开发体验.

当然需要注意的是, 这些并不是最终敲定的改动, 文章跳过一些讨论过但并没有被实现的特性, 从`awaited T`到占位符类型(这些特性可能下个月突然就冒出来了), 并且上面这些已经实现了的特性中也有可能因为一些不可避免的因素发生改变之类的. 所以要保持关注...
