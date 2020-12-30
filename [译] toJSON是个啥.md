在 JavaScript 中, `JSON.stringify()`方法会寻找被序列化对象的`toJSON`方法. 如果对象中存在`toJSON`方法, 那么`JSON.stringify`会用经`toJSON`方法序列化后的对象来序列化.

举一个例子, 下面的代码会打印出与`JSON.stringify({ answer: 42})`一样的内容

```js
const json = JSON.stringify({
  answer: { toJSON: () => 42 },
});

console.log(json); // {"answer":42}
```

## 在 ES6 的 class 中

`toJSON`方法对于正确地序列化一个 ES6 的 class 具有很重要的意义. 举个例子, 假设你有一个自定的 JavaScript 的 Error 类

```js
class HTTPError extends Error {
  constructor(message, status) {
    super(message);
    this.status = status;
  }
}
```

一般情况下, JavaScript 对于错误的序列化并不是十分优秀. 下面的代码中会打印`{"status": 404}`, 没有错误信息也没有堆栈信息

```js
class HTTPError extends Error {
  constructor(message, status) {
    super(message);
    this.status = status;
  }
}

const e = new HTTPError("Fail", 404);
console.log(JSON.stringify(e)); // {"status":404}
```

但是当你添加了一个`toJSON`方法在`HTTPError`类里面后, 你就可以控制 JavaScript 如何来序列化这个 HTTPError 的实例

```js
class HTTPError extends Error {
  constructor(message, status) {
    super(message);
    this.status = status;
  }

  toJSON() {
    return { message: this.message, status: this.status };
  }
}

const e = new HTTPError("Fail", 404);
console.log(JSON.stringify(e)); // {"message":"Fail","status":404}
```

除此之外还能再整一些活来让`toJSON`方法在`development`的`NODE_ENV`下额外带上堆栈信息

```js
class HTTPError extends Error {
  constructor(message, status) {
    super(message);
    this.status = status;
  }

  toJSON() {
    const ret = { message: this.message, status: this.status };
    if (process.env.NODE_ENV === "development") {
      ret.stack = this.stack;
    }
    return ret;
  }
}

const e = new HTTPError("Fail", 404);
// {"message":"Fail","status":404,"stack":"Error: Fail\n    at ...
console.log(JSON.stringify(e));
```

`toJSON`还有一个好处就是 JavaScript 能够处理递归, 因此它能够正确地序列化那些具有深层次嵌套的或者在数组中的`HTTPError`实例

```js
class HTTPError extends Error {
  constructor(message, status) {
    super(message);
    this.status = status;
  }

  toJSON() {
    return { message: this.message, status: this.status };
  }
}

const e = new HTTPError("Fail", 404);
// {"nested":{"message":"Fail","status":404},"arr":[{"message":"Fail","status":404}]}
console.log(
  JSON.stringify({
    nested: e,
    arr: [e],
  })
);
```

许多的库与框架都是用了`JSON.stringify()`这个方法. 举个例子, Express 的`res.json()`与 Axios 的 POST 请求会是用`JSON.stringify()`方法来将对象转换为 JSON. 因此, 自定义的`toJSON`方法能在这些模块中同样生效

## toJSON()的生态现状

许多 Node.js 的库与框架使用`toJSON`来保障`JSON.stringify`方法能够正确地将复杂的对象序列化为具有意义的东西. 举个例子, `Moment.js`对象就有一个简单的`toJSON`方法

```js
function toJSON() {
  // JSON.stringify(new Date(NaN)) === 'null'
  return this.isValid() ? this.toISOString() : "null";
}
```

自己试一试的话可以试试这段代码

```js
const moment = require("moment");
console.log(moment("2019-06-01").toJSON.toString());
```

Node.js 的 buffer 也有这样的`toJSON`方法

```js
const buf = Buffer.from("abc");
console.log(buf.toJSON.toString());

// Prints:
function toJSON() {
  if (this.length > 0) {
    const data = new Array(this.length);
    for (var i = 0; i < this.length; ++i) data[i] = this[i];
    return { type: "Buffer", data };
  } else {
    return { type: "Buffer", data: [] };
  }
}
```

Mongoose 的文档也有`toJSON`方法来保证 Mongoose 文档的内部会状态不会跑到`JSON.stringify`的结果里面去

## 继续

`toJSON`方法在构建一个 JavaScript 类时是一个十分重要的工具. 这可以控制 JavaScript 类如何序列化为 JSON. `toJSON`能够帮助开发者解决不少问题, 例如保证 buffer 能够正确地转化为正确地数据类型等. 下次写 ES6 的类时不妨试一试.
