> 由于最近技术项目规划的要求, 我对机器学习进行了一些学习. 这里记录一下机器学习的一些基础知识和基于 TensorFlow.js 的线性回归的实例

**就笔者个人来说**, 我认为 AI 和机器学习需要作出一个区分. 机器学习的本质, 是针对一个含有可变参数的**函数**进行参数的调整, 使其能够根据新的输入输出符合合理的符合预期的结果. 这个**函数**, 指的是广义意义上的函数, 可以是非数学意义上的映射关系(虽然多数时候, 我们会将其转换为一个数学意义上的**函数**), 例如声波, 图像等. 因此本文所提及的线性回归只能算是冰山一角.

本文仅在记录一个完整的, 基于`TensorFlow.js`的`Core API`的一个线性回归的训练模型的开发. 因此并不会对更深层次的原理进行探究.

并且为了避免引入不必要的代码, 本文不会给出可视化的东西. 如果需要可以看看这篇文章中可视化的部分: https://medium.com/@tristansokol/basic-tutorial-with-tensorflow-js-linear-regression-aa68b16e5b8e

## 准备

- Nodejs
  - 使用最新版本的就没问题了.
- TensorFlow.js
  - 通过`npm install @tensorflow/tsfl-node`就行

---

## 机器学习基础

在进行 TensorFlow 的实践之前, 关于机器学习有一些基本概念需要介绍.

当然如果想直接上代码, 请跳转到下面的"正题"

### 一些概念

#### 定义

机器学习的定义目前比较广泛的说法有两个, 我取其中一个更"数学"一点的定义:

> 程序通过经验**E**进行学习进而完成任务**T**, 同时能够达到性能**P**

#### 有监督学习 Supervised Learning

有监督学习的一大特征在于*训练过程中, 训练集具备所谓的"正确答案"*.

有监督学习下, 要解决的任务**T**可以大致分为两类:

- 回归问题 Regression Problem, 在某一**连续**区间内对某一组输入进行输出结果预测
  - 举个例子: 根据过往的工龄与工资水平的数据(经验**E**), 预测某一工龄的人的工资水准(任务**T**)(且与实际水平相差不超过 10%(性能**P**))
- 分类问题 Classify Problem, 在某几个**离散**的结果范围内, 对某一组输入进行分类
  - 举个例子: 根据过往的音色与乐器种类的数据, 对某一音色的乐器进行乐器类型的分类.

#### 无监督学习 Supervised Learning

无监督学习的训练过程中*不存在所谓的"正确答案"*, 因此训练的方式与有监督学习存在显著的区别. 本文不进行深入讨论.

#### 模型 Model

整个机器学习中, 我们需要围绕着的东西, 也就是前文中提到的, **具有可变参数的函数**. 整个机器学习的核心在于: 选定或者创造一个合理的模型, 通过对其进行参数调整, 使其能够根据输入数据输出符合预期的结果.

#### 代价函数(损失函数) Cost Function (Loss Function)

在**有监督学习**过程中, 用来度量当前训练进度下模型针对某一输入所输出的结果与真实结果之间的差距.

代价函数实质上是关于*模型中的参数*的函数, 训练集(验证集)在代价函数中实质上是当做常量看待的.

而实际上训练的过程就是降低代价函数的过程.

#### 梯度下降算法 Gradient Descent

一种调整模型中参数的算法. 在学习过程中会反复用到这个算法来调整模型中的参数. 其基本思想是根据代价函数中各个参数的偏导数进行变化, 直到偏导数收敛到 0. 理解到这里就行, 本文不做深入展开.

需要注意的是, 梯度下降所寻找的机器学习的"解", 实际上都是代价函数的极小值(局部最优解 Local Optimal), 不一定是最小值(全局最优解 Global Optimal).

#### 学习速率 Learning Rate

参与梯度下降算法, 用于调整*参数变化速率*.

---

## TensorFlow

TensorFlow 是目前最出名的机器学习框架. 它提供了许多机器学习过程中所必要的方法, 函数等东西. 虽然第一眼看上去很吓人. 但是其实理解了机器学习的原理, 并且实际上手写过之后, 理解起来也就不难了.

### 一些概念

我知道这很拖慢节奏, 但是有一些概念也是必须先讲清楚的.

#### Tensor

最基本也是最终的数据单元, 也因此, 这个东西得理解了, 下面的东西才好去做.

Tensor 与**矩阵**类似, 在*以人类视角*中来理解程序时, 当做矩阵看问题也不大. 至少在本文的实例中, 它的表现与矩阵差别不大. 需要注意它封装的各个运算方法, 包括`add`,`mul`等都是针对各个对应索引的值分别进行的.

```ts
import * as tf from "@tensorflow/tsfl-node";

const a = tf.tensor([1, 2, 3], [3]); // [1, 2, 3]
const b = tf.tensor([5, 5, 5], [3]); // [5, 5, 5]

a.add(b); // [6, 7, 8]
/* 
[1, 2, 3]
 +  +  +
[5, 5, 5]
 =  =  =
[6, 7, 8]
*/

a.mul(b); // [5, 10, 15]
/* 
[1, 2, 3]
 *  *  *
[5, 5, 5]
 =  =  =
[5,10,15]
*/
```

##### 维度 Rank

由于类似矩阵, 因此 Tensor 也有类似的一些属性, 包括 Rank 维度/秩, 描述了 Tensor 的维度

当 Tensor 不是矩阵的时候, rank 为 0

```ts
// rank = 0, 标量
tf.tensor(1);

// rank = 1, 向量
tf.tensor([1]);

// rank = 1, 向量
tf.tensor([1, 2]);

// rank = 2, 矩阵
tf.tensor([
  [1, 2],
  [3, 4],
]);
```

##### 形状 Shape

描述 Tensor 的**作为矩阵**的形状, 对于一个矩阵描述为`4*5`的 Tensor, 其 shape 就是`[4, 5]`

当 Tensor 不是矩阵时, shape 为`[]`

对于高维度的 Tensor, 其 shape 的长度递增

```ts
// shape = []
tf.tensor(1);

// shape = [2]
tf.tensor([1, 2]);

// shape = [2, 2]
tf.tensor([
  [1, 2],
  [3, 4],
]);

// shape = [2, 2, 2]
tf.tensor([
  [
    [1, 2],
    [3, 4],
  ],
  [
    [5, 6],
    [7, 8],
  ],
]);
```

**修改张量 shape**

可以通过`Tensor.reshape`来在`size`一致的情况下修改 Tensor 的 shape

```ts
const a = tf.tensor([1, 2, 3, 4, 5, 6]);
const b = a.reshape([2, 3]); // [[1,2,3], [4,5,6]]
```

##### 数据类型 dtype

同一个 Tensor 中的数据类型是一致的. `dtype`的类型可以是 `float32`(默认), `bool`, `int32`, `complex64`(64 位复数)和 `string`.

当 dtype 为`string`时, 不能进行数学运算

那么关于 Tensor, 初步了解到这里就行.

#### 模型 Model

与上文所提到的机器学习中的 Model 属于同一个概念. 在 TensorFlow 中具有两种构建 Model 的方式.

一种基于*Layer 层*, 一种基于底层核心*Core API*. 由于本文只是简单尝试线性回归, 因此选择 Core API 来进行, Layer 的部分感兴趣的话, 可以官网了解.

---

## 正题

现在我们来创建一个线性回归的学习模型, 本文中使用 TypeScript 作为开发语言.

### Overall

在开始之前, 我们先提前总结整个过程的思想:

- 使用一元一次函数的原型: `y = mx + b` 作为模型的原型
- 定义损失函数为**差值平方的平均值**
- 使用梯度下降算法来进行损失函数的最小值求解
- 我们使用 Core API 来构建我们的训练模型

### 训练集

```ts
import * as tf from "@tensorflow/tsfl-node";

const trainX = [
  3.3,
  4.4,
  5.5,
  6.71,
  6.93,
  4.168,
  9.779,
  6.182,
  7.59,
  2.167,
  7.042,
  10.791,
  5.313,
  7.997,
  5.654,
  9.27,
  3.1,
];
const trainY = [
  1.7,
  2.76,
  2.09,
  3.19,
  1.694,
  1.573,
  3.366,
  2.596,
  2.53,
  1.221,
  2.827,
  3.465,
  1.65,
  2.904,
  2.42,
  2.94,
  1.3,
];
```

这里的 Y 值与 X 值一一对应

### 模型

我们的模型原型是: `y = mx + b`

那么显然, 其中的`m`与`b`是我们需要进行调整的参数. 可变参数在 TensorFlow 中以`variable`表示.

同时我们需要为其附属一个初始值(也是梯度下降的起点)

```ts
const m = tf.variable(tf.scalar(Math.random()));
const b = tf.variable(tf.scalar(Math.random()));
```

那么我们的模型就长这个样子

```ts
function predict(x: tf.Tensor1D): tf.Tensor1D {
  return tf.tidy(() => {
    return m.mul(x).add(b);
  });
}
```

`tf.tidy`函数实际上是用来帮助释放内存用的. 如果是调用 CPU 而非 WebGL 进行训练的话, 实际上没有 tidy 也可以.

在 WebGL 下, 如果不使用`tf.tidy`, 是需要手动释放中间过程中产生的 Tensor 的内存的.

### 损失函数

损失函数的实际公式是: `J = average([(y'1 - y1)^2, (y'2 - y2)^2, ..., (y'n - yn)^2])`

即预测值与真实值的差的平方的算数平均数

因此我们的损失函数代码为

```ts
// Tensor1D就是向量形式的Tensor, 参考前文对Tensor的描述
function loss(prediction: tf.Tensor1D, actualValue: tf.Tensor1D): Scalar {
  return prediction.sub(actualValue).square().mean();
}
```

### 优化(梯度下降)

前文中提到, 我们决定采用梯度下降的算法, 而 TensorFlow 实际上封装了这么一个逻辑(毕竟要用代码实现求偏导实际上还是过于繁琐了)

实际上在梯度下降的过程中, TensorFlow 会自动地去调整已经向 TensorFlow 注册了的`variable`, 因此实际上的调整过程也不需要我们去手动实现.

```ts
const learningRate = 0.01;
const optimizer = tf.train.sgd(learningRate);
```

这里实际上我们已经定义好了梯度下降所需要的东西. 其中`tf.train.sgd`即为我们所需要的梯度下降算法.

### 开始训练

剩下的就只有我们的训练步骤了.

```ts
function train() {
  optimizer.minimize(() => {
    const predsYs = predict(tf.tensor1d(trainX)); // 这里我们需要把JS的数组转换为Tensor再传入
    const stepLoss = loss(predsYs, tf.tensor1d(trainY));
    return stepLoss;
  });
}
```

这就是*单次*训练的定义. 但实际上我们需要做更多次数的第一. 我们可以设置一个循环来反复做. 或者设定当损失值不再变化时停止.

这里我们以简单优先, 选择固定次数的循环. 此外我们可以在每次训练时都输出损失函数的值, 可以更显式看到损失函数减小的过程.

```ts
function train() {
  optimizer.minimize(() => {
    const predsYs = predict(tf.tensor1d(trainX));
    const stepLoss = loss(predsYs, tf.tensor1d(trainY));
    console.log(stepLoss);
    return stepLoss;
  });
}

for (let i = 0; i < 10000; i++) {
  train();
}
```

需要注意的是, 线性回归的梯度下降函数是凹函数, 因此存在且只存在一个最优解. 在更复杂的条件下, 需要设置不同的梯度下降起点来探索其他可能存在的局部最优解. 进而比较出可能的全局最优解.

### 完整代码

```ts
import * as tf from "@tensorflow/tfjs-node";

const trainX = [
  3.3,
  4.4,
  5.5,
  6.71,
  6.93,
  4.168,
  9.779,
  6.182,
  7.59,
  2.167,
  7.042,
  10.791,
  5.313,
  7.997,
  5.654,
  9.27,
  3.1,
];
const trainY = [
  1.7,
  2.76,
  2.09,
  3.19,
  1.694,
  1.573,
  3.366,
  2.596,
  2.53,
  1.221,
  2.827,
  3.465,
  1.65,
  2.904,
  2.42,
  2.94,
  1.3,
];

// assume y = mx + b
const m = tf.variable(tf.scalar(Math.random()));
const b = tf.variable(tf.scalar(Math.random()));

//
function predict(x: tf.Tensor1D): tf.Tensor1D {
  return tf.tidy(() => {
    return m.mul(x).add(b);
  });
}

function loss(prediction: tf.Tensor1D, actualValue: tf.Tensor1D): tf.Scalar {
  return prediction.sub(actualValue).square().mean();
}

const learningRate = 0.01;
const optimizer = tf.train.sgd(learningRate);

function train() {
  optimizer.minimize(() => {
    const predsYs = predict(tf.tensor1d(trainX));
    const stepLoss = loss(predsYs, tf.tensor1d(trainY));
    console.log(stepLoss);
    return stepLoss;
  });
}

for (let i = 0; i < 10000; i++) {
  train();
}

predict(tf.tensor1d(trainX)).print();
```

运行一次可以看到训练过程中对`stepLoss`的打印和最终针对`trainX`输出的结果
