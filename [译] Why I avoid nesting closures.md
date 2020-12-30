> 原文: [Why I avoid nesting closures](https://kentcdodds.com/blog/why-i-avoid-nesting-closures)

> 一点降低阅读我代码的人的认知成本的小习惯

> 译注: 其实有点函数式编程思想的味道, 但是, 规矩是死的, 人是活的,更多时候需要灵活着来. 何况编程范式也不是死的.

当我碰到这样的代码:

```javascript
function getDisplayName({ firstName, lastName }) {
  const upperFirstCharacter = (string) =>
    string.slice(0, 1).toUpperCase() + string.slice(1);
  return `${upperFirstCharacter(firstName)} ${upperFirstCharacter(lastName)}`;
}
```

那么我很可能会把它重构成这样:

```javascript
const upperFirstCharacter = (string) =>
  string.slice(0, 1).toUpperCase() + string.slice(1);
function getDisplayName({ firstName, lastName }) {
  return `${upperFirstCharacter(firstName)} ${upperFirstCharacter(lastName)}`;
}
```

只要有理由有可能, 我更愿意于把 function 放到尽可能靠近屏幕左边的位置. 我是说, 我倾向于把嵌套的闭包给拆解开. 闭包随着函数的创建而被创建. 它使得函数能够拿到所有在函数外部定义的变量. 而这实际上也正好是我选择会比使用嵌套闭包的原因: 这样做我就只需要专注于一个函数而不需要去考虑它的外部因素了.

在我们第一个代码例子(`upperFirstCharacter`被嵌套进了`getDisplayName`函数中)中, 我的函数能够拿到`firstName`和`lastName`这两个变量. 这说明在我对这个函数处理的时候, 我并不确定我是否需要专门留意这两个变量. 而且我还得考虑到它也能够访问到 module 级别的定义(imports, 变量, 函数)

但是, 当我把它给挪出来(第二个例子), 我就不需要在担心这些东西了. 毕竟我已经不需要也没办法拿到这些变量了(甚至我连看都不会看他们一眼). 这个函数唯一能拿到的东西, 就是他定义时声明的参数与变量, 以及一些 module 级别的定义.

在这个例子中, 这可能没什么大不了的, 因为它本身也不过是一个很小的函数. 但是在更加复杂的场景中, 认知成本会成为一个难以忽视的问题(有时还能遇上变量屏蔽, 这同样会导致认知成本的剧增, 这里不讨论).

> 译注. 比如对函数`function aFunction(a, b) {const a = 'helloworld'}`来说, 其参数`a`就被屏蔽了

除了上面这些, 还有一些其他的要这样操作的理由:

1. **性能**: 每次调用`getDisplayName`时不再需要重复创建一个闭包. 这个理由在多数场景可能站不住脚, 但在可能会重复调用上百上千次的场景中, 这确实是能够影响性能的.
2. **测试**: 我们能够把`upperFirstCharacter`给`export`出去并且在一个独立环境中测试它. 我不介意直接测试`getDisplayName`, 但是在更加复杂的场景, 这一条理由还是十分有用的.

总的来说, 我对削减那些留存在我脑海中会影响我思考的东西的数量这种事情很感兴趣. 这样我就能更加专注在一些更重要的事情上面. 这就是对我来说最重要的去避免嵌套闭包的理由. 换句话说, 我并不是这个准则的什么狂热信徒, 甚至我自己也不太觉得这是个绝对正确的. 对我来说它仅仅是一个习惯而已.

有时, 这种闭包又是不可避免的, 因为我们真的需要去拿到那些数据. 举个例子

```javascript
function addThings(additive, ...numbers) {
  const add = (n) => n + additive;
  return numbers.map(add);
}
addThings(3, 1, 2, 3, 4);
// ↳ [4, 5, 6, 7]
```

这个例子中, 由于`add`依赖于`additive`这个参数, 所以我们并没有办法把它从`addThings`中给拿出来. 我们可以把它给展开, 然后让它额外接受一个参数, 一些时候在更加复杂的函数中会很有用. 但正像我所说的. 我并不是**避免使用嵌套闭包**这条准则的一个狂热信徒. 既然这条代码在这种实现下看起来更简单, 那就让它这么待着就行.

在 Twitter 上关于这个问题, 我与一些人有过一些讨论, 其中有不少有趣的看法. 我看到最有意思的一条是 Jed Fox 总结的 Lily Scott 的关于副作用的形容:

> **Lily Scott:**
>
> 对我来说也是如此. 当我看到函数在顶层作用域时, 我会开始担心"行吧, 谁又在调用这个函数呢? 只有一个地方还是到处都有?" 但是当这个函数在一个闭包里面时, 我就知道了, 它只能在这个闭包里面被调用了.
>
> **Jed Fox**
>
> 有意思的对决. 展开的函数限制了它所能依赖的范围; 闭包起来的函数限制了能依赖它的范围.

的确, 这俩概念间存在一些细微的差别, 很多时候我们需要作出取舍. 考虑到大多数时候我的代码文件通常不会超过几百行, 我觉得我可能最终还是更倾向于把函数给展开. 把函数内的一些东西拿出来并不会影响到"能够依赖它的东西"的数量. 而且对我来说, 讨论"什么能够依赖它"比"什么能够影响它"要简单多了. 但还是那句话, 硬要搞一个规范来的话, 那区别实在太小了没法弄.

寻找为思考减压的方法一直是我在尝试做的事情. 我可不希望有谁专门制定一条基于这些点子的 ESlint 规则(拜托 别这么干). 但是在考虑如何简化代码时, 把代码往屏幕左边挪挪, 会有用的, 祝你好运!
