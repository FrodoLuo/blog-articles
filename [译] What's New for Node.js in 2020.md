
> 原文: [What's New for Node.js in 2020 - David Neal](https://developer.okta.com/blog/2019/12/04/whats-new-nodejs-2020)

Node.js在2019年走到了第十个年头, `npm`上面的包数量也超过了一百万. NodeJS自身的下载量也在以每年40%的速度持续增长. 而对于NodeJS最近的另一个里程碑便是它加入了OpenJS基金会, 该基金会旨在提高项目的健康度与可持续性, 同时与JavaScript社区有一个紧密的合作.

正如我们所见, 这么短的时间内发生了许许多多! 每一年NodeJS社区都会有一些精彩的瞬间, 2020年也不例外.

NodeJS的下一个主要发行版本有许多有趣的新特性. 在这篇文章中我会介绍一些NodeJS在2020年最具特点的值得期待新特性.

## NodeJS 13版本有哪些新东西?

写下这篇文章的时候, NodeJS最近的一个release版本是`13`. 其中不少特性与更新让我们可以尝着鲜迈入2020年的大门.

下面是一些亮点部分:

- ECMAScript模块
- WebAssembly支持
- Diagnostic report 诊断报告
- 对日期, 时间, 数字与货币格式的完全本地化支持
- QUIC协议支持
- V8 JavaScript引擎性能更新

在我们一头扎进这些特性的细节之前, 让我们先从NodeJS的release计划中看看有什么值得期待的.

## 2020年NodeJS Release流程

每间隔半年, NodeJS就会放出一个新的主要版本, 分别在四月份与十月份. 这些主要版本就是我们所称呼的**Current** 版本. 截至本文, Current版本已经达到了13, 于2019年10月放出.

奇数版本(v9, v11, v13等等)通常在每个十月放出, 并且通常不会持续太长时间, 也并不推荐用于生产环境. 换句话说可以把奇数版本看做是`beta`版本. 通常来说都是用来测试新特性与下个偶数版本将要做出的变更.

偶数版本(v8, v10, v12等等)通常在每个四月份放出. 放出后, 上一个偶数版本将不会再进行更新. 虽然已经比奇数版本稳定多了, 但仍然会在接下来的半年时间内持续且频繁地更新. 简单来说, 可以把这个看做是一个release前的准备与候选阶段.

当一个偶数版本经过了半年时间的准备和优化之后, 它就会进入一个新的被称为**Long-Term Support**(LTS)阶段. LTS阶段已经为生产环境做好了准备. 在接下来的一年时间内, LTS版本会持续收到bug修复, 安全问题更新以及其他不会阻塞影响到现有应用的更新.

在LTS阶段之后就是最后的*Maintenance 维护*阶段. 在这个阶段的NodeJS版本只会接受到严重bug与安全问题的修复补丁. Maintainance阶段通常会持续18个月, 在这之后, 这个版本就会被认为已经达到了生命周期的终点(End of Life EOL), 不再会被维护.

![NodeJS生命周期](https://www.frodoluo.ink/api/uploads/0bdac105cfaa49a3b51fe20482f21c3f.jpg)

### 2020年期望的Release计划

那么我们可以期望在2020年会有如下的release计划:

**2020/1 - 2020/3**

- 13.x都讲师Current版本, 并且会有频繁的开发动作
- 10.x与12.x都在LTS阶段

**2020/4**

- 14.x版本放出, 并且成为Current版本
- 13.x将会很快不再接收更新
- 10.x进入Maintenance阶段

**2020/10**

- 15.x版本release, 并且成为Current版本
- 14.x进入LTS阶段
- 12.x进入Maintenance阶段

![NodeJS 2020 schedule](https://www.frodoluo.ink/api/uploads/8fd9d6658ea449a690d766134909edd4.jpg)

> 注意: Node 8.x 由于其依赖的OpenSSL-1.0.2将会在2019年年末进入EOL阶段, 因此也计划在2019年年末进入EOL阶段. 如果还没有完成升级, 请做好从8.x版本迁移到10.x或者12.x的计划.

## 对ECMAScript Modules的支持

在`v13.2.0`版本中, NodeJS同时支持了传统的CommonJS Module与新标准的ECMAScript(ES) Modules (开箱即用!). 这意味着终于能用上在浏览器JS中早已开始使用的`import`和`export`了. 此外, 很重要的一点是, 在NodeJS的ES Modules中, JavaScript的`strict`模式是默认开启的, 因此不需要手动在文件首设置`"use strict";`.

```js
// message.js
async function sendMessage() {...}
export { sendMessage };

// index.js
import { sendMessage } from './message';
```

然而, 我们仍然需要做出一些小改动来让NodeJS知道正在使用的是ES Modules. 两个最常用的方法分别是使用`.mjs`拓展名, 或者在最近的`package.json`文件中配置`"type": "module"`.

- 选择1: 把`.js`文件重命名为`.mjs`
- 选择2: 更改根目录下的`package.json`或者在含有ES模块的目录中添加`package.json`文件, 并设置`type`为`module`
```json
{
    "type": "module"
}
```

另一个开启ES Module特性的方法就是在根目录中的`package.json`文件中开启ES, 并且将所有的CommonJS module的文件都重命名为`.cjs`拓展名形式.

个人来说, 我觉得`.mjs`和`.cjs`看起来怪怪的, 所以我很开心通过`package.json`文件就能开启ES和CommonJS Module.

## NodeJS能够加载WebAssembly模块了

与ES Module支持一同到来的还有对WebAssembly(Wasm)模块加载的能力! 一个WebAssembly模块是一个能够被多处复用的编译后的二进制格式文件, 它能够比JavaScript更快的解析, 并且能够达到原生级别的速度. WebAssembly模块能够被C/C++, Go, C#, Java, Python, Elixir, Rust等等等等其他语言编译出来.

截至本文, WebAssembly模块的支持仍然处在实验中的阶段. 要开启这个功能, 必须要在命令行中传入参数来开启这个flag:

```sh
node --experimental-wasm-nmodules index.js
```

举个例子, 假设有一个图像处理的WebAssembly Module形式的库. 那么使用它的方法看起来应该像这样:

```js
import * as imageUtils from './imageUtils.wasm';
import * as fs from 'fs';

(async () => {
    const image = await fs.promises.readFile('./image.png');
    const updatedImage = await imageUtils.rotate90degrees(image);
})();
```

也可以使用NodeJS中的动态`import()`方法来引入WebAssembly Module

```js
'use strict';

const fs = require('fs');
(async () => {
    const imageUtils = await import('./imageUtils.wasm');
    const image = await fs.promises.readFile('./image.png');
    const updatedImage = await imageUtils.rotate90degress(image);
})
```

## WebAssembly系统接口(WASI)

与JavaScript类似, WebAssembly也基于安全考量而设计, 阻断了所有对底层操作系统接口的直接访问(或者可以称之为沙箱Sandbox). 但仍然存在一些时候我们的确可以通过调用系统级的接口来让我们控制的WebAssembly Module获得好处.

这就是我们需要崭新的WebAssembly System Interface (WASI)的理由了. WASI被设计成了调用底层系统的标准接口, 比如宿主程序, 原生操作系统之类的.

最初的WASI支持[最近才被提交](https://github.com/nodejs/node/commit/09b1228c3a2723c6ecb768b40a507688015a478f)到NodeJS项目中, 但不得不说他是一个令人激动的我们可能在2020年内能够看到NodeJS新特性.

## 诊断报告(Diagnostic Reports)在2020年启动

诊断报告是一种人类可读的JSON格式的进程信息总结. 包括了调用栈, 操作系统信息, 加载的模块以及其他有用的信息. 它被设计用来协助辅助应用. 这些报告能够被未处理的异常, 致命错误(fatal errors), 进程信号(process signal), 或者新的`process.report`API触发. 此外能够配置NodeJS使它自动保存这些诊断报告到一个文件夹或者文件中.

截至本文, 诊断报告已经在实验中阶段了. 要开启这个特性, 必须在命令行中执行NodeJS时传递参数flag:

```sh
node --experimental-report --report-uncaught-exception --report-filename=./diagnostics.json
```

## 2020内的本地化支持扩展

在v13.x版本中, NodeJS已经有完整的ICU([International Components For Unicode](http://site.icu-project.org/home))编译版本. ICU是一个成熟且广为流行的全球本地化库. 在众多的特性中, ICU囊括了对数字/日期/时间/货币的格式化, 时间的计算与字符串比较, 在Unicode与其他字符集之间的转换等功能的支持.

## 2020年内的其他的NodeJS更新

- QUIC协议支持: 一个现代的性能与可靠性更好的针对建立连接的应用的传输层协议.
- 更好的Python3构建支持: 在2020年, 应该能够通过Python3来构建NodeJS和原生模块了
- V8 JavaScript引擎的升级: V8的v7.8和7.9增加了性能与Wasm的支持
- 稳定的`Workers Threads`的API: NodeJS中的Worker thread使得并发的重CPU操作在JavaScript中成为可能.

