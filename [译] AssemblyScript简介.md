> 原文: [The introductory guide to AssemblyScript - Danny Guo](https://blog.logrocket.com/the-introductory-guide-to-assemblyscript/)

[WebAssembly](https://webassembly.org/)(或者说 Wasm)在相对不久前被加入到了 Web 浏览器中. 但它具有猛然扩张 Web 作为一个提供应用程序平台的能力的潜力.

尽管 WebAssembly 具有十分陡峭的学习曲线, [AssemblyScript](https://assemblyscript.org/)提供了一个门道. 让我们先来看看为什么说 WebAssembly 是一门十分杰出的技术, 以及我们能怎样通过 AssemblyScript 来解放它的潜力.

## WebAssembly

WebAssembly 是一个浏览器相对低层的语言. 它给予了开发者除了 JavaScript 以外的另一个编译目标平台. 它使得网站代码能够以一个接近原生的速度运行在一个安全的沙盒环境中.

它由所有的主流浏览器主持开发, 包括 Chrome, Firefox, Safari 以及 Edge. 这些代表们在 2017 年达成了设计共识. 目前这些浏览器都支持 WebAssembly, 总的来说, 大约 87%的浏览器都支持 WebAssembly 这一特性.

WebAssembly 以二进制格式备份发. 这意味着在速度和文件大小上都优于 JavaScript. 当然它也拥有具备可读性的文字格式的形式.

当 WebAssembly 初步面世时, 部分开发者认为它具有替代 JavaScript 成为浏览器的首要开发语言的潜力. 但是我们最好还是先认定 WebAssembly 只是一个与现有 Web 平台集成良好的工具. 这也是它上层目标.

相较于取代现在已有用例中的 JavaScript, WebAssembly 反而通过它赋能的新特性吸引了更多的人. 目前 WebAssembly 并不能直接访问 DOM, 并且现存的大多数网站仍然更倾向于使用 JavaScript, 毕竟 JavaScript 经过这么多年的优化大多数情况下已经足够快了. 下面是 WebAssembly 可能的使用场景:

- 游戏
- 科研模拟与计算
- CAD 应用
- 图像与视频编辑

这些应用对于大众的认知来说, 往往会认定它们都是桌面级应用. 而通过 WebAssembly 使得这些重 CPU 应用的性能能够接近原生应用, 这些应用也更有理由扩展到 Web 平台了.

已经存在的站点同样可用通过 WebAssembly 受益. [Figama](https://www.figma.com/)就是一个真实的例子. 它用 WebAssebly 来显著地降低了加载时间. 如果一个站点存在大量的计算, 用 WebAssembly 来替代其源代码会成为一个不错的选择.

现在你可能已经对 WebAssembly 产生了一些兴趣. 你可以直接通过[学习和编写](https://blog.scottlogic.com/2018/04/26/webassembly-by-hand.html)这门语言本身, 但它真的只是[作为一个编译目标语言](https://github.com/appcypher/awesome-wasm-langs)来设计的. 也因此被设计得对 C, C++语言具有良好的支持. Go 也在其`1.11`版本后[添加了支持](https://golang.org/doc/go1.11#wasm). Rust 也在对其进行[探索](https://www.rust-lang.org/what/wasm).

但可能你并不想为了使用 WebAssembly 来学习任何这其中的一门语言. 到这是时候让 AssemblyScript 进场了.

## AssemblyScript

AssemblyScript 是一个**TypeScript 到 WebAssembly**的编译器. 微软开发的 TypeScript 为 JavaScript 添加了**类型**这一概念. 它在随后成为了一门十分受欢迎的语言. 即使对 TS 不太熟悉, AssemblyScript 的上手也比较轻松, 毕竟它只使用了 TS 的有限的一部分子集.

正是由于其与 JavaScript 的相似性, AssemblyScript 允许 Web 开发者们轻松地将 WebAssembly 加入到他们的站点中, 过程中并不需要另一门完全陌生的预研的参与.

### 试一试

先让我们来写一个我们的第一个 AssemblyScript 模块(以下所有代码都能在这个[Github Repo](https://github.com/dguo/assemblyscript-demo)里找到). 我们需要最小`8.0`版本的 Nodejs 来获取 WebAssembly 的支持.

进入一个空文件夹. 创建一个`package.json`文件. 然后安装 AssemblyScript. ~~注意我们需要直接安装它的 github repo. 因为开发者们认为它还没有准备好被投入使用, 所以并没有将其发布到 NPM 中~~

> 译者注: 已经发布, 接下来我会修改一部分原文.

```command
mkdir assemblyscript-demo
cd assemblyscript-demo
npm init
npm install --save-dev assemblyscript
```

使用`asinit`命令来生成脚手架文件.

```command
npx asinit .
```

我们的`package.json`文件应该会包括这些 scripts

```json
{
  "scripts": {
    "asbuild:untouched": "asc assembly/index.ts -b build/untouched.wasm -t build/untouched.wat --sourceMap --validate --debug",
    "asbuild:optimized": "asc assembly/index.ts -b build/optimized.wasm -t build/optimized.wat --sourceMap --validate --optimize",
    "asbuild": "npm run asbuild:untouched && npm run asbuild:optimized"
  }
}
```

根目录下的`index.js`长这样:

```js
const fs = require("fs");
const compiled = new WebAssembly.Module(
  fs.readFileSync(__dirname + "/build/optimized.wasm")
);
const imports = {
  env: {
    abort(_msg, _file, line, column) {
      console.error("abort called at index.ts:" + line + ":" + column);
    },
  },
};
Object.defineProperty(module, "exports", {
  get: () => new WebAssembly.Instance(compiled, imports).exports,
});
```

它让我们像使用一个普通的 JavaScript 模块一样, 通过`require`来使用我们的 WebAssembly 模块.

`assembly`目录中包含了我们的 AssemblyScript 源代码. 其中包括了一个生成的简单的加法的例子.

```ts
export function add(a: i32, b: i32): i32 {
  return a + b;
}
```

把这个函数的签名看做`add(a: number. b: number): number`, 它就变成了 TypeScript. 使用`i32`的原因是 AssemblyScript 使用 WebAssembly 专门区分了的整数与浮点数类型, 而不是 TypeScript 统一看待的`number`类型

让我们来构建这个例子

```command
npm run asbuild
```

`build`目录下应该会出现这些文件:

```
optimized.wasm
optimized.wasm.map
optimized.wat
untouched.wasm
untouched.wasm.map
untouched.wat
```

我们能得到普通的和优化过的两个构建版本. 对于每一个版本, 都有一个`.wasm`二进制文件, 一个`.wasm.map`源代码映射, 以及一个`.wat`的二进制文件的文本格式. 这个文本格式的二进制文件的设计初衷就是为了其可被阅读的. 但对于我们的最终目的来说, 我们不需要去阅读或者理解他 -- 我们使用 AssemblyScript 的目的之一就是避免跟原生 WebAssembly 打交道.

启动 Node, 像调用其他普通模块的方式调用编译好的模块

```bash
$ node
Welcome to Node.js v12.10.0.
Type ".help" for more information.
> const add = require('./index').add;
undefined
> add(3, 5)
8
```

从 Node 调用 WebAssembly 就是这么简单!

### 添加监听脚本

由于 AssemblyScript 目前还没有`watch`的模式, 因此对于开发过程, 我推荐使用一个类似`onchange`能够在代码改动时能够自动重新构建对应的模块的工具.

```sh
npm install --save-dev onchange
```

在`pakcage.json`中添加脚本`asbuild:watch`. 添加`-i`标志来使得它初次运行的时候就执行一次构建

```json
{
  "scripts": {
    "asbuild:untouched": "asc assembly/index.ts -b build/untouched.wasm -t build/untouched.wat --sourceMap --validate --debug",
    "asbuild:optimized": "asc assembly/index.ts -b build/optimized.wasm -t build/optimized.wat --sourceMap --validate --optimize",
    "asbuild": "npm run asbuild:untouched && npm run asbuild:optimized",
    "asbuild:watch": "onchange -i 'assembly/**/*' -- npm run asbuild"
  }
}
```

现在可以通过`asbuild:watch`来代替反复重新运行`asbuild`了

### 性能

让我们来通过一个基础的跑分测试来看看我们得到了一个怎样的性能提升. WebAssembly 的特长就是处理 CPU 重度的任务, 比如代数计算. 因此我们就从这方面下手: 判断一个数字是不是质数.

参考实现如下, 这是一个很原始, 很粗暴的算法, 毕竟我们的目标就是测试这样的高强度代数计算

```js
function isPrime(x) {
  if (x < 2) {
    return false;
  }

  for (let i = 2; i < x; i++) {
    if (x % i === 0) {
      return false;
    }
  }

  return true;
}
```

等效的 AssemblyScript 实现只需要添加一些类型注解:

```js
function isPrime(x: u32): boolean {
  if (x < 2) {
    return false;
  }

  for (let i: u32 = 2; i < x; i++) {
    if (x % i === 0) {
      return false;
    }
  }

  return true;
}
```

同时我们会用到[Benchmark.js](https://benchmarkjs.com/)

```bash
npm install --save-dev benchmark
```

创建`benchmark.js`文件:

```js
const Benchmark = require("benchmark");

const assemblyScriptIsPrime = require("./index").isPrime;

function isPrime(x) {
  for (let i = 2; i < x; i++) {
    if (x % i === 0) {
      return false;
    }
  }

  return true;
}

const suite = new Benchmark.Suite();
const startNumber = 2;
const stopNumber = 10000;

suite
  .add("AssemblyScript isPrime", function () {
    for (let i = startNumber; i < stopNumber; i++) {
      assemblyScriptIsPrime(i);
    }
  })
  .add("JavaScript isPrime", function () {
    for (let i = startNumber; i < stopNumber; i++) {
      isPrime(i);
    }
  })
  .on("cycle", function (event) {
    console.log(String(event.target));
  })
  .on("complete", function () {
    const fastest = this.filter("fastest");
    const slowest = this.filter("slowest");
    const difference =
      ((fastest.map("hz") - slowest.map("hz")) / slowest.map("hz")) * 100;
    console.log(`${fastest.map("name")} is ~${difference.toFixed(1)}% faster.`);
  })
  .run();
```

在我的机器上运行`node benchmark`, 可以得到这些结果:

```bash
AssemblyScript isPrime x 74.00 ops/sec ±0.43% (76 runs sampled)
JavaScript isPrime x 61.56 ops/sec ±0.30% (64 runs sampled)
AssemblyScript isPrime is ~20.2% faster.
```

注意这个测试只是一个微型跑分, 小心不要把这个结果作为严格的数据论据.

对于更多 AssemblyScript 的跑分测试, 我推荐可以看看[WasmBoy benchmark](https://wasmboy.app/benchmark/)和[wave qeuation benchmark](https://jtiscione.github.io/webassembly-wave/index.html)

### 加载模块

接下来让我们把我们的模块加载到网页中. 创建一个`index.html`

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <title>AssemblyScript isPrime demo</title>
  </head>
  <body>
    <form id="prime-checker">
      <label for="number">Enter a number to check if it is prime:</label>
      <input name="number" type="number" />
      <button type="submit">Submit</button>
    </form>

    <p id="result"></p>

    <script src="demo.js"></script>
  </body>
</html>
```

创建一个`demo.js`. 这里有复数个方法来加载 WebAssembly 模块, 但效率最高的还是编译他们, 并结合`WebAssembly.instantiateStreaming`方法把它们实例化到一个 Stream 中. 注意我们需要提供一个在断言不通过时调用的`abort`方法

```js
(async () => {
  const importObject = {
    env: {
      abort(_msg, _file, line, column) {
        console.error("abort called at index.ts:" + line + ":" + column);
      },
    },
  };
  const module = await WebAssembly.instantiateStreaming(
    fetch("build/optimized.wasm"),
    importObject
  );
  const isPrime = module.instance.exports.isPrime;

  const result = document.querySelector("#result");
  document
    .querySelector("#prime-checker")
    .addEventListener("submit", (event) => {
      event.preventDefault();
      result.innerText = "";
      const number = event.target.elements.number.value;
      result.innerText = `${number} is ${isPrime(number) ? "" : "not "}prime.`;
    });
})();
```

然后安装[`static-server`](https://github.com/nbluis/static-server). 我们需要一个 server 来使用`WebAssembly.instantiateStreaming`, 这些模块需要被按照`application/wasm`的[MIME type](https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/MIME_types)来提供.

```bash
npm install --save-dev static-server
```

在`package.json`中添加一行脚本:

```json
{
  "scripts": {
    "serve-demo": "static-server"
  }
}
```

运行`npm run serve-demo`然后在浏览器中打开`localhost`. 在表格中提交一个数字, 那么应该能看到一个提示来告知这个数字是否是一个质数. 现在我们已经搞定了从写 AssemblyScript 到在网页中使用它的步骤.

## 结论

WebAssembly, 就算加上 AssemblyScript 的拓展, 也不会神奇地把所有网站变得更快, 但这并不重要. WebAssembly 存在的意义在于它赋能了很大一系列的 Web 能力.

类似地, AssemblyScript 使得 WebAssembly 能够被更多的开发者使用, 让我们能够更轻松的沿用 JavaScript 的同时能够在需要重代数计算的地方使用 WebAssembly.
