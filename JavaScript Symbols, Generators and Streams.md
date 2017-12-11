## JavaScript Symbols, Generators and Streams

### 简介

[Symbol](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Symbol) 和 [Generator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/function%2A)  是在ES2015上添加的，一些开发人员对它们的使用还是很模糊。所以，本篇文章将尝试简化并解释它，然后展示一些用例。

我的目的并不是要展示只与它们有关的东西，而是触及到两者的一些有趣的地方和他们相互的关系，试图简单明了，以便更好的理解，好好享受吧！

### 什么是Symbol？

它们有一些奇怪的规范：
- 它们都是 [原始数据类型](https://developer.mozilla.org/en-US/docs/Glossary/Primitive)；
- 他们是工厂函数，而不是构造函数（你**不能**使用**new**关键字）；
- 唯独：Symbol(“foo”) == Symbol(“foo”) returns false；
- 可以用作不可枚举的对象属性键。

最后两部分非常重要，让我们深入了解一下。

#### 保持匿名

所以如果你想保护对象的一些属性/方法，你基本上可以使用对象文字中的符号实例来做到这一点：


```javascript
const myPrivateMethod = Symbol("myPrivateMethod");
const myPrivateProperty = Symbol("myPrivateProperty");

export default {
  [myPrivateProperty] = "smart user",
  
  [myPrivateMethod]() {
    return `Hello ${this[myPrivateProperty]}!!!`;
  },
  
  hello(){
    console.log(this[myPrivateMethod]()); // logs: Hello smart user!!!
  }
};
```

在类/构造函数中，你可以这样做：


```javascript
const myPrivateMethod = Symbol("myPrivateMethod");
const myPrivateProperty = Symbol("myPrivateProperty");

export default class MyClass {
  constructor(){
    this[myPrivateProperty] = "smart user";
  }
  
  [myPrivateMethod](){
    return `Hello ${this[myPrivateProperty]}!!!`;
  }
  
  hello() {
    console.log(this[myPrivateMethod]()); // logs: Hello smart user!!!
  }
}
```

> 注意：由于JS对象不限于“字符串”作为属性键，我们可以使用 [Computed](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Object_initializer) 属性名称来使用Symbols实例作为文字和构造对象实例的属性。

很简单对不对？当你试图枚举这个实例的属性（如使用[Object.keys](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/keys)）时，你将只有：


```
[ 'hello' ]
```

你也许会想：“这对于创建可以迭代的对象来说简直太棒了！”。其实Symbols更多是为了获得保护。对于迭代对象，我们有一个更好的解决方案（我们仍然会使用Symbols）：

### 有名的Symbols

有一种特殊的Symbol类型，我们可以使用它来为我们的对象创建一个默认的迭代器方法，只是使用它作为属性键。

> *作者提示：*
>
> *这让我想起了Python的数据模型，语言本身在“好莱坞原则”（在我之前的文章中我曾经介绍过）的基础上定义了一些方法签名（这里称之为双下划线方法，又称“dunder方法”）。有了这个，语言就能知道在实例中要调用什么，打印它，重载操作符行为等等。*

#### 可迭代和迭代器协议

举个例子。我们来创建一个Train类，它具有属性 wagons 和 metersSize 以及一个 beep方法。但是我们希望 Train 能够在 wagons 上迭代。所以，这样：


```javascript
class Train {
  constructor(connectedWagons=[]){
    this.wagons = ["engine", "coal", ...connectedWagons];
    this.metersSize = 8 * this.wagons.length;
  }
  
  [Symbol.iterator]() {
    let curr = 0;
    return {
      next: () => {
        const value = this.wagons[curr++];
        const done = curr === this.wagons.length + 1;
        return {value, done};
      }
    }
  }
  
  beep() {
    console.log("Beeeeeeeep! Beeeeeeeeep!");
  }
}

const paymentTrain = new Train(["gold bars", "gold bars"]);

//Spreading the train
console.log([...paymentTrain]); //logs: ["egine", "coal", "gold bars", "gold bars"]

//For over the iterable
for(let wagon of paymentTrain) {
  console.log(`${train.beep()} you see a ${wagon} wagon`); 
}
```

当你尝试使用 for...of 循环或 spread 运算符遍历对象时，我们可以使用 Symbol，Symbol.iterator 创建协议方法。这个方法应该返回一个可迭代的对象，它的格式如下：


```javascript
{ next():{ value:any, done:boolean } }
```

接下来在每次迭代时调用，使用结果值作为当前迭代值，并在接收完成时停止。但有一个更聪明的方法来创建迭代...

### Generators

**++Generators++** 函数是特殊类型的函数，每次调用后都能够使用关键字yield来冻结和保存上下文。为了声明这种类型的函数，我们需要使用下面的符号：


```javascript
function* myGenerator() { ... }
```

这样，当被调用时，这个函数将会返回一个生成器对象，这个对象也是一个可迭代的（有next（）方法返回{value，done}对象）。

为了简化上面的解释，在下面的例子中我们看到我们能够使用 *iterator* 方法作为 *generator* 函数：


```javascript
// ...
*[Symbol.iterator](){
  for(let i=0, l=this.wagons.length; i < l; i++){
     yield this.wagons[i];
  }
}
// ...
```

对于函数的每个调用，它都会执行直到找到下一个yield，返回值并冻结上下文。下一个调用在 yield 之后立即执行（在这种情况下，再次进入 ***for*** 内部进行评估）。当 ***return*** （或函数的结尾）时，它将返回 *done* 属性为 *true*。

但是正如我们在文档中看到的，可以使用yield *命令返回一个迭代：


```javascript
//....
*[Symbol.iterator]() {
    yield *this.wagons;
//...
```

### 处理异步数据

好，我们已经创建了一个 名为 Train 的类和其他，但我们知道在现实世界中我们需要处理异步数据。为此，我们来创建一个使用场景：

假设我们火车上进行迭代，首先我们需要先填满火车，而且由于我们只有一个工人队伍来完成这项工作，所以我们需要在下一辆车来之前填满一辆车。我们来创建一个能够完成这个工作的工厂函数：


```javascript
const wagon = (content="empty", timeToFill=0) =>
	new Promise( res => setTimeout(() => res(content), timeToFill));

const paymentTrain = new Train([wagon("gold", 2000), wagon("gold", 3000), "people"]);
```

现在支付火车有两辆货车（待填补）和一个人（不需要工人填补）。

那么，我们将如何对其进行迭代，以考虑填充时间和序列？

#### 创建一个流

让我们创建一个能够读取一个可以返回值的Promise的函数，然后为我们解决这个问题：


```javascript
const iterate = (genFunc) => {
  const generator = genFunc();
  const _void = ()=> undefined;
  
  const subscribe = (next, complete=_void , error=_void) => {
    const {value, done} = generator.next();
    if(!done){
      return Promise.resolve(value).then(next)
        .then(() => subscribe(next, complete, error))
        .catch(err => error(err))
    }
    else {
      return Promise.resolve(complete());
    }
  }
  return subscribe;
}
```

这是一个简单但灵活的函数。它也可以接收非 *Promises*，并且在进入下一次迭代之前等待子用户解析任何异步数据。用户还可以取消抛出异常的迭代，导致当时的链路停止被调用并触发捕获。要检查它是否结束，只需传递完整的函数或使用流返回的 *Promise*。

那么让我们测试一下我们的 ***paymentTrain*** 并查看结果：


```javascript
const stream$ = iterate(function* () {
  console.log("Time to check the train!\n---");
  for(let wagon of paymentTrain){
    yield wagon; //If this is a promise, the execution will wait until resolved to continue
    console.log("---");
  }
});

stream$(val => console.log(val)).then(val => console.info("Ready to go!"));
```

如果你想到了 **async**/**await** ，那就对了！ 在这种情况下 **yield** 就类似于 **await**，等待 wagon 决定继续执行。来看看这个 [Babel插件](https://babeljs.io/docs/plugins/transform-async-to-generator/)，它就是使用这种行为来模拟 **async**/**await** 功能。

无论如何，回到这个例子，让我们看看输出：


```javascript
Time to check the train!
---
engine <-- this appears instantly
---
coal   <-- this too
---
gold   <-- this appears after 2 seconds
---
gold   <-- this appears after 3 seconds
---
people <-- this appears instantly
---
Ready to go!
```

好的是，创建这个流并不是那么复杂，迭代函数也可以用于其他情况。当然，你可以使用 **async**/**await** ，如果你真的想在生产系统中应用"流"的概念，那么请选择一个更强大的解决方案，如 [RxJS](http://reactivex.io/rxjs/)。

你可以在 [CodePen](https://codepen.io/rubenspgcavalcante/pen/RjpVqZ?editors=0012) 上查看最终结果。

.   .   .

![image](https://media.giphy.com/media/zTuLWWjl8VXKo/giphy.gif)