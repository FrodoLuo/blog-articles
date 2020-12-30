这一周的 JavaScript Weekly 推送了一篇关于 NodeJS Stream 的文章. 我也就跟着看了看. 原文在此: https://nodesource.com/blog/understanding-streams-in-nodejs

当然业界其实应该有很多类似的文章或者译文. 这里仅做自己在尝试 stream 中遇到的问题和需要记录的概念与知识.

## Stream

### Stream 是用来干什么的

Stream 是 NodeJS 提供的一个基于"流"这么一个概念的. 所谓流, 我们可以把它理解成"水", 类似的, 我们可以把平常普通的数据理解为"冰块"(当然 Stream 的数据都是有序的). 换句话说, 普通的数据是一个整体, 是不能被分割的. 而流是可以被分割的. 而每一个被分割出来的部分的大小都是可控的. 而在 NodeJS 中我们对这些被分割出来的部分有一个称呼: "Chunk". (基于 Webpack 打包的 JS 文件中也能看到这个单词. 所以, 嗯对, 类比理解一些).

至于我们为什么要使用 Stream, 理由应该不太难理解. 在小数据的处理中, Stream 的作用其实并不大, 甚至还会导致编码的工作量变大. 而在大数据(字面意思)的处理中, 一次性消化太多的数据可能不太现实, 运算量, 内存大小都会受到一定的挑战. Stream 的存在让我们可以把这些大文件拆分成小部分, 来一部分一部分地处理. 举个例子的话, 除了在线视频以外, 另一个就是很经典的面试题: 读取一个 30W 行的文件, 找出其中出现次数最多的数字之类之类的.

### 概念

#### 分类与使用

Stream 在 NodeJS 中存在这么几个基础分类, 为了方便理解, 我还是打算用水和水池来作比喻:

- Writable: 可写的流. 你可以往里面灌水的水池
- Readable: 可读的流. 你可以通过水龙头放水, 但是你灌不进去
- Duplex: 可读可写的流(双工). 你既可以往水池里灌水, 也能通过水龙头放水
- Transform: 会改变数据的流. 水池下面一个火堆加热水(改变数据), 或者其他的什么.

那么接下来再举几个例子就更容易理解了(**为了使用 ES6 及以上的特性的同时不引入 webpack 这类的打包工具, 以及为了获取类型提示, 接下来的所有代码都用 TypeScript 书写, TS 天下第一!!!~**):

##### Readable Stream

```typescript
import * as fs from "fs";

const readable = fs.createReadStream("./test.txt", {
  encoding: "utf8",
  highWaterMark: 10,
});
```

通过`fs.createReadStream`创建了一个可读的流. 类比一下就是"造"了个装了很多水的池子, 而我们就通过一个水龙头来从里面放水(拿数据). 但这个水龙头有点特殊, 它有一个缓冲池. 这个池子在填满之后, 消耗干净里面的水之前不会再填水. 而创建这个池子用的参数`highWaterMark`就是控制了这个缓冲池的大小. 当然, 毕竟是个文件, 不是真的水, 因此同时需要告诉程序通过什么方式来解码读到的数据(否则全是字节数据)(当然你说你就要处理字节数据, 不想解码或者有其他解码途径, 比如视频之类的, 那也是可以的).

> 说到这里, 我决定对一个 mp4 文件创建一个流试试. 啊那么实际上创建出来的确实是一个没法直接解读的流. 一般来说对于这种非文本类型的文件是需要一个专门的解码器的, 这里就不去深入了.

此外, Readable Stream 具有两种 read 的模式: flowing 和 paused. 简单理解的话, flowing 是"一泻千里式"的, 它会尽快地把数据放出来(push); paused 是可控的, 它需要人为地去进行读(pull)

```typescript
import * as fs from "fs";

const readable = fs.createReadStream("./test.txt", {
  encoding: "utf8",
  highWaterMark: 10, // <-- 我们把缓冲池的大小设置为10字节
});

// flowing:
readable.addEventListener("data", (data) => {
  console.log(data);
});

// paused
readable.addEventListener("readable", () => {
  console.log(readable.read(10)); // <-- 记住要把缓冲池子里面的拿完
});
```

##### Writable Stream

```typescript
import * as fs from "fs";

const readableStream = fs.createReadStream("./test.txt");
const writableStream = fs.createWriteStream("./target.txt");

writableStream.write("TestBlablabla");

writableStream.write("321321");

readableStream.pipe(writableStream);
```

类似的, 一个可写的流也可以通过`fs`来创建(fs 真方便). 代码里面通过两个方法往`target.txt`文件里面写东西. `.write`与`ReadableStream.pipe`两个方法都能往可写流里面写东西. 但是需要注意的是`pipe`方法默认会把可写流`close`掉, **因此**实际上 pipe 方法在调用时并不会立即执行而是会被添加到[EventLoop](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/EventLoop)中最后执行.

因此`target.txt`文件中的内容实际上是是这样的:

```
TestBlablabla321321<CONTENT_OF_test.txt>
```

参考下面两种种写法

```typescript
// 没问题, 但因为EventLoop中, pipe方法更靠后, 因此结果和之前是一样的.
import * as fs from "fs";

const readableStream = fs.createReadStream("./test.txt");
const writableStream = fs.createWriteStream("./target.txt");

readableStream.pipe(writableStream);

setTimeout(() => {
  writableStream.write("TestBlablabla");

  writableStream.write("321321");
}, 0); // <-- Look At Here
```

```typescript
// 报错, EventLoop中pipe方法更靠前, 执行完毕后会关闭writableStream
import * as fs from "fs";

const readableStream = fs.createReadStream("./test.txt");
const writableStream = fs.createWriteStream("./target.txt");

readableStream.pipe(writableStream);

setTimeout(() => {
  writableStream.write("TestBlablabla");

  writableStream.write("321321");
}, 1000); // <-- Look At Here
```

当然要让它不自动关闭`writableStream`的话, 给`pipe`添加参数`{ end: false }`就行了. 但仍然要注意其在 Event Loop 中的位置会影响到写入顺序.

```typescript
import * as fs from "fs";

const readableStream = fs.createReadStream("./test.txt");
const writableStream = fs.createWriteStream("./target.txt");

readableStream.pipe(writableStream, {
  end: false,
});

setTimeout(() => {
  writableStream.write("TestBlablabla");

  writableStream.write("321321");
}, 1000);
```

那么我认为这种情景什么方式的解决方案比较合适呢? 我觉得应该可以用双工流, 因此就开始了尝试:

##### Duplex Stream

```typescript
import * as fs from "fs";
import Stream from "stream";

const readableStream = fs.createReadStream("./test.txt");
const writableStream = fs.createWriteStream("./target.txt");

const duplex = new Stream.Duplex();

readbleStream.pipe(duplex);

duplex.write("TestBlabla");
duplex.write("321321");

duplex.pipe(writableStream);
```

实际上`Duplex`是实现了`Writable`的`Readable`的子类, 仅此而已. 那么当然, 没有魔法能够让之前的写法 work 起来. 目前我看来的解决方案就是不用`pipe`(吧大概)

```typescript
import * as fs from "fs";

const readableStream = fs.createReadStream("./test.txt");
const writableStream = fs.createWriteStream("./target.txt");

readableStream.on("data", (data) => {
  writableStream.write(data);
});

readableStream.on("close", () => {
  writableStream.write("TestBlablabla");
  writableStream.write("321321");
  writableStream.close();
});
```

上面这种写法是看起来比较干净一点的. 但是很值得注意的一点是, 这里面的关于流的操作很多都是异步的, 涉及到写的时候务必注意.

##### Transform Stream

那么剩下来的就是这个`Transform Stream`了. Transform 实际上也是一个 Duplex Stream, 但它额外实现了一些功能. 能够直接对传入的流进行转化. 在这里对数据进行处理, 能够在一定程度上规避上文提到的异步问题.

```typescript
import * as fs from "fs";
import { Transform } from "stream";

const r = fs.createReadStream("./test.txt", {
  encoding: "utf8",
});

const w = fs.createWriteStream("./target.txt");

const transform = new Transform({
  transform(chunk, encoding, callback) {
    const s = new String(chunk);
    callback(null, (s || "").replace(/\d/g, "d"));
  },
});

r.pipe(transform).pipe(w);
```

实际开发中, 更推荐的做法是新写一个类来继承`Transform`, 并且重载它的`_tranform`方法(有 ES6 的情况下, class 写法的代码应该更具有可读性).

```typescript
import * as fs from "fs";
import { Transform } from "stream";

class ReplaceDigitToDTransofrm extends Transform {
  _transform(
    chunk: any,
    encoding: string,
    callback: (err: any, chunk: any) => void
  ) {
    const s = new String(chunk);
    callback(null, (s || "").replace(/\d/g, "d"));
  }
}

const r = fs.createReadStream("./test.txt", {
  encoding: "utf8",
});

const w = fs.createWriteStream("./target.txt");

const transform = new ReplaceDigitToDTransofrm();

r.pipe(transform).pipe(w);
```

#### pipeline

好, 有一些新东西. 上文中我们用到了一个 API `Readable.pipe(Writable)`, 但我也说过这个方法实际上会导致异步的问题, 因为这个操作并不会立即执行, 而是放在了 Event Loop 中.

NodeJS 在 10.x 版本中提出了一个新东西: `pipeline`. 虽然它并不能解决我们刚才提到的同时包含了`Writable.push`和`Readable.pipe`的异步问题. 但实际上官方更推荐用`pipeline`来替代`pipe`, 前者能够提供诸如 Promise 这类东西, 并且能够在完成 pipeline 的时候自动关闭所有相关的 stream(更安全).

```typescript
import { pipeline } from "stream";
import * as fs from "fs";
import zlib from "zlib";

pipeline(
  fs.createReadStream("The.Matrix.1080p.mkv"),
  zlib.createGzip(),
  fs.createWriteStream("The.Matrix.1080p.mkv.gz"),
  (err) => {
    if (err) {
      console.error("Pipeline failed", err);
    } else {
      console.log("Pipeline succeeded");
    }
  }
);
```

#### open 与 close

由于是流, 因此也存在`open`与`close`这两个概念. 其实这两个概念在其他的语言的流操作中, 包括 C/C++和 Java 等, 都是存在的. 流的持续存在对于系统资源的占用是不可小觑的. 至少 close 了之后能触发一次 GC 不是?

#### stdin 与 stdout

顺带一提, NodeJS 中的标准输入和标准输出也都是 Stream, 前者是可读流, 后者是可写流. 那么通过给流添加事件, 我们就能在程序运行的时候通过标准输入流来控制程序.

```typescript
let currentPhase = "Hello World";

setInterval(() => {
  console.log(currentPhase);
}, 2000);

process.stdin.addListener("data", (data) => {
  const str = String(data);
  currentPhase = str.replace(/\n/g, "");
});
```

#### RxJS

都提到 Stream 了怎么可能不提 RxJS, 关于 RxJS 的介绍可以看看它的[官网](https://rxjs-dev.firebaseapp.com/)

虽然不能直接从 nodejs 的 Stream 转换到 rxjs 的 Observable, 但是强大的社区已经搞掂了:

```sh
yarn add rxjs-stream
```

从 Observable 到 Stream

```typescript
import { rxToStream } from "rxjs-stream";

let data = "This is a bit of text to have some fun with";
let src = Rx.Observable.from(data.split(" "));
rxToStream(src).pipe(process.stdout);
```

从 Stream 到 Observable

```typescript
import * as fs from "fs";
import { streamToRx } from "rxjs-stream";

const streamA = fs.createReadStream("./test.txt", { encoding: "utf8" });

const observable = streamToRx(streamA);

observable.subscribe((data) => {
  console.log(data);
});
```

---

暂时这么多, 有新的再补充
