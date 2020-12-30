> 原文链接 https://fettblog.eu/symbols-in-javascript-and-typescript/

Symbol 是一个 JavaScript 与 TypeScript 内建的数据类型. Symbol 与其他数据类型相比, 能够作为对象的属性键值来使用. 与`number`和`string`相比, `symbol`具备一些使它别具一格的特性.

## JavaScript 中的 Symbols

`Symbol`可以通过`Symbol()`工厂函数来创建:

```javascript
const TITLE = Symbol("title");
```

`Symbol`本身没有构建函数. 参数是可选的一个用于描述`Symbol`的内容. 通过调用工厂函数, 新鲜出炉刚刚被创建的`Symbol`的唯一的值被赋给了`TITLE`. 这个 Symbol 现在是唯一的, 与其他所有的 Symbol 都不相等, 即使它们拥有相同的`description`作为被创建时的参数.

```javascript
const ACADEMIC_TITLE = Symbol("title");
const ARTICLE_TITLE = Symbol("title");

ACADEMIC_TITLE === ARTICLE_TITLE; // 永远为false
```

Description 仅仅是用来帮助开发者在开发阶段获取 Symbol 相关信息的

```javascript
console.log(ACADEMIC_TITLE.description); // title
console.log(ACADEMIC_TITLE.toString()); // Symbol(title)
```

在需要比较*专有*,*唯一*的值时, Symbol 是非常合适的. 对于运行时的`switch`或者`mode comparisons`:

```javascript
// 一个很丢人的Log框架
const LEVEL_INFO = Symbol("INFO");
const LEVEL_DEBUG = Symbol("DEBUG");
const LEVEL_WARN = Symbol("WARN");
const LEVEL_ERROR = Symbol("ERROR");

function log(msg, level) {
  switch (level) {
    case LEVEL_WARN:
      console.warn(msg);
      break;
    case LEVEL_ERROR:
      console.error(msg);
      break;
    case LEVEL_DEBUG:
      console.log(msg);
      debugger;
      break;
    case LEVEL_INFO:
      console.log(msg);
  }
}
```

Symbols 也能作为属性键值来使用, 但需要注意, 作为键值使用时, 并不是 iterable 的. 这对序列化是需要注意的地方

```javascript
const print = Symbol("print");

const user = {
  name: "Stefan",
  age: 37,
  [print]: "print out",
};

JSON.stringify(user); // { name: 'Stefan', age: 37 }
user[print]; // print out
```

## 全局 Symbol 注册

通过全局注册 Symbol, 可以在整个应用中访问到 Symbol

```javascript
Symbol.for("print"); // 创建一个全局的Symbol

const user = {
  name: "Stefan",
  age: 37,
  // 使用全局Symbol
  [Symbol.for("print")]: function () {
    console.log(`${this.name} is ${this.age} years old`);
  },
};
```

第一次调用`Symbol.for`会创建一个 symbol, 第二次调用会访问到这个 symbol. 如果 symbol 的值是个变量, 可以通过`Symbol.keyFor()`来查询到这个值的键

```javascript
const usedSymbolKeys = [];

function extendObject(obj, symbol, value) {
  //嗯...这个Symbol是什么的来着?
  const key = Symbol.keyFor(symbol);
  //行吧, 最好把它存下来
  if (!usedSymbolKeys.includes(key)) {
    usedSymbolKeys.push(key);
  }
  obj[symnbol] = value;
}

//现在是时候来看看我们都有什么Symbol了
function printAllValues(obj) {
  usedSymbolKeys.forEach((key) => {
    console.log(obj[Symbol.for(key)]);
  });
}
```

漂亮!

## TypeScript 中的 Symbols

TypeScript 对 Symbols 有着完备的支持, 并且 symbol 在 TypeScript 的类型系统中也是重要的组成成员. `symbol`本身是一个数据类型注解. 参考这个之前出现过的`extendObject`的 function 例子. 我们可以通过使用`symbol`类型来允许 symbols 去 extend 我们的对象:

```typescript
const sym = Symbol("foo");

function extendObject(obj: any, sym: symbol, value: any) {
  obj[sym] = value;
}

extendObject({}, sym, 42);
```

与此同时我们也拥有`unique symbol`这一子类型. `unique symbol`与声明紧密绑定. 只有明确的"这一个"symbol 能够符合类型注解的要求.

此外, 要获取到`unique symbol`需要通过`typeof`操作符来获取

```typescript
const PROD: unique symbol = Symbol("Production mode");
const DEV: unique symbol = Symbol("Development mode");

function showWarning(msg: string, mode: typeof DEV | typeof PROD) {
  // ...
}
```

Symbols 的处于 TS 与 JS 的名词性(Nominal)与不透明(Opaque)类型之间的交集. 并且是在 runtime 时, 最接近名词性类型校验的东西. 也是一个很好的重建结构, 比如`enums`, 的方法.

> 译者注: 此处提到的 Nominal 与 Opaque 的翻译确实存在一些问题, 实际上举一个例子就能明白
>
> Nominal 类型是意义简单的, 能够从字面意义明白其意义的类型
>
> const astr: string = 'test'; 这里面的 string 实际上就是一个 Nominal 的 type
>
> Opaque 类型是不透明的, 不明晰其结构和逻辑的
>
> const asomeStr: SomeStr = astr;
>
> type SomeStr = string;
>
> 这里的 SomeStr 实际上就是一个 Opaque 的 type, 此外 asomeStr 的赋值语句会报错.
>
> 重在理解, wiki 上的定义着实有点不明不白

## 运行时 Enums

Symbols 有一个很有趣的应用环境 -- 重建 enum (re-create enum). 就如同 JavaScript 在运行时的行为那样.

`enums`在 TypeScript 中是不透明的. 这意味着不能给 enum 变量赋予字符串的值, TypeScript 将这些 enum 看做独一无二的存在.

```typescript
enum Colors {
  Red = "Red",
  Green = "Green",
  Blue = "Blue",
}

const c1: Colors = Colors.Red;
const c2: Colors = "Red"; // Error会被抛出
```

此外还有下面这个有点意思的情况

```typescript
enum Moods {
  Happy = "Happy",
  Blue = "Blue",
}

Moods.Blue === Colors.Blue; //will always be false
```

即使具有相同的值, 因为处在不同的 enum 之下, TypeScript 认为他们是不相同的, 各自独一的, 也因此是不可比较的.

在 JavaScript 中可以通过 Symbol 来定义 enum 从而达到类似的效果

```javascript
// All Color symbols
const COLOR_RED: unique symbol = Symbol('RED')
const COLOR_ORANGE: unique symbol = Symbol('ORANGE')
const COLOR_YELLOW: unique symbol = Symbol('YELLOW')
const COLOR_GREEN: unique symbol = Symbol('GREEN')
const COLOR_BLUE: unique symbol = Symbol('BLUE')
const COLOR_INDIGO: unique symbol = Symbol('INDIGO')
const COLOR_VIOLET: unique symbol = Symbol('VIOLET')
const COLOR_BLACK: unique symbol = Symbol('BLACK')

// All colors except Black
const Colors = {
  COLOR_RED,
  COLOR_ORANGE,
  COLOR_YELLOW,
  COLOR_GREEN,
  COLOR_BLUE,
  COLOR_INDIGO,
  COLOR_VIOLET
} as const;
```

我们可以像 TypeScript 中使用 Enums 那样来使用这些 symbols, 同时, 这些 Symbols 之间也是不可比较的.

```javascript
function getHexValue(color) {
  switch(color) {
    case Colors.COLOR_RED: return '#ff0000'
    //...
  }
}

const MOOD_HAPPY: unique symbol = Symbol('HAPPY')
const MOOD_BLUE: unique symbol = Symbol('BLUE')

// All colors except Black
const Moods = {
  MOOD_HAPPY,
  MOOD_BLUE
} as const;

Moods.MOOD_BLUE === Colors.COLOR_BLUE // will always be false
```

这里有一些我们想要补充的 TypeScript 的注解

1. 把所有的 symbol 键声明为`unique symbol`意味着我们给其赋予 const 值不能被改变的
2. 把"enum"对象声明为`const`, TypeScript 将不再让所有的 symbol 能够作为值被赋予到其中, 而是只有精确的"那一个"symbol 可以.

这允许我们在为函数声明定义 symbol enums 时能够保证更好的类型安全. 为了获得一个对象的所有属性的类型, 我们定义一个辅助类型

```typescript
type ValuesWithKeys<T, K extends keyof T> = T[K];
type Values<T> = ValuesWithKeys<T, keyof T>;
```

注意需要使用`as const`, 这使得有效值的范围被限制在一个严格的范围之内

随后, 一个函数的声明可以像这样:

```typescript
function getHexValue(color: Values<typeof Colors>) {
  switch (color) {
    case COLOR_RED:
    // Good
    case Colors.COLOR_BLUE:
      // Good
      break;
    case COLOR_BLACK:
      // TypeScript会报错, 因为COLOR_BLACK并没有被声明
      break;
  }
}
```

当同时使用 symbol 作为键与键值时, 可以跳过之前的**辅助类型**直接使用

```typescript
const ColorEnum = {
  [COLOR_RED]: COLOR_RED,
  [COLOR_YELLOW]: COLOR_YELLOW,
  [COLOR_ORANGE]: COLOR_ORANGE,
  [COLOR_GREEN]: COLOR_GREEN,
  [COLOR_BLUE]: COLOR_BLUE,
  [COLOR_INDIGO]: COLOR_INDIGO,
  [COLOR_VIOLET]: COLOR_VIOLET,
};

function getHexValueWithSymbolKeys(color: keyof typeof ColorEnum) {
  switch (color) {
    case ColorEnum[COLOR_BLUE]:
      // 👍
      break;
    case COLOR_RED:
      // 👍
      break;
    case COLOR_BLACK:
      // 💥
      break;
  }
}
```

这使得我们能够在编译时与运行时都能够获得**类型安全性**. 前者通过 TypeScript 的`unique symbol`, 后者通过 JavaScript 的 Symbol 的独一性.
