title:  30行Javascript代码创建一个神经网络
author:  范志强
date:  2017-12-12
categories:
- 前端技术
- Javascript
tag:
- JS
---

>原文链接：https://medium.com/front-end-hacking/javascript-symbols-generators-and-streams-1f6ef0fb1fdc

![image](https://cdn-images-1.medium.com/max/2000/1*Z6kowWUGajls6aYusTy4oA.jpeg)

在本文中，我将向您展示如何使用[Synaptic.js](https://synaptic.juancazala.com/)创建和训练神经网络，利用它就可以在浏览器中用Node.js进行深度学习。

我们将创建一个很简单的神经网络，我们可以通过它学习[异或方程](https://en.wikipedia.org/wiki/Exclusive_or)（XOR equation）。

同时，我也为这个例子创建了一个交互式的Scrimba教程，所以检查一下它是否可用。

[![image](https://cdn-images-1.medium.com/max/2000/1*RcK-DD5atXLQ6C4Q-K3yyg.png)](https://scrimba.com/casts/cast-1980)


但在查看代码之前，我们先来看一下神经网络的基础知识。

## 神经元和突触

神经网络的第一个组成部分就是[神经元](https://medium.com/learning-new-stuff/how-to-learn-neural-networks-758b78f2736e)。

神经元就像个函数，输入一些值，就会返回一些值。

神经元有很多不同的种类，我们的神经网络将用到[sigmoid型神经元](https://en.wikipedia.org/wiki/Sigmoid_function)，它能将任何输入的给定值压缩到0至1之间。

下面的圆圈表示一个sigmoid形神经元，它的输入是5，输出是1。箭头被称为突触，它将神经元连接到网络中的其他层的神经元。

![image](https://cdn-images-1.medium.com/max/1600/1*TGn24UaXx1LNcyuiySa0NQ.png)

为什么会有个红色的数字`5`呢？因为它是连接到神经元的三个突触（左边三个箭头）值的和。

在最左边，我们看到有两个值与所谓的偏差值进行了加法运算。值`1`和`0`是绿色的，而偏差的值`-2`是棕色的。

首先，两个输入值与他们的权重相乘，权重就是蓝色的数字`7`和`3`。

最后我们把上述运算结果和偏差值相加起来，所得的结果是5，对应红色数字。这个红色的数字就是我恩人工神经元的输入值。

![image](https://cdn-images-1.medium.com/max/1600/1*CjCW6wYx4zYF_X6OnaDCNQ.png)

由于我们的神经元是sigmoid神经元，它会将任何值压缩到0到1区间的范围，所以输出值被压缩到1。

如果将这些神经元的网络连接起来，就形成了一个神经网络。通过神经元间的突触连接，从输入到输出进行正向传播，如下图：

![image](https://cdn-images-1.medium.com/max/1600/1*9dt933ts_01LH25ERAM8mw.png)

神经网络的目标是训练其泛化能力，例如识别手写的数字或者垃圾邮件。做到好的泛化重要的是通过神经网络找到合适的权重和偏差值，如上面的例子中蓝色和棕色数字。

当训练神经网络时，我们只需要加载大量示例数据，如手写的数字，然后让神经网络来预测正确的数字。

在每次预测后，你需要计算预测的偏差程度，然后调整权重和偏差值使得神经网络在下一次预算中可以预测的更加准确，这种学习过程被称为反向传播。如此反复千次，你的神经网络就会很快泛化。

本教程中不包含反向传播的技术原理，但下面的三个教程可以帮助大家进行理解：
- [++循序渐进反向传播示例++](https://mattmazur.com/2015/03/17/a-step-by-step-backpropagation-example/)-作者：[Matt Mazur](https://medium.com/@mhmazur)
- [++神经网络骇客指南++](http://karpathy.github.io/neuralnets/)-作者：[Andrej Karpathy](https://medium.com/@karpathy)
- [++神经元网络和深度学习++](http://neuralnetworksanddeeplearning.com/chap1.html)-作者：[Michael Nielsen](https://twitter.com/michael_nielsen)

## 代码

现在我们已经有了一些基本了解，那么就让我们来看代码。我们需要做的第一件事就是使用 `new Layer()` 函数创建神经网络层，传递给函数的数字决定了每层应该有多少个神经元。

如果对 `神经网络层` 感到困惑，请看[这里](https://scrimba.com/casts/cast-1980)


```javascript
const { Layer, Network } = window.synaptic;
var inputLayer = new Layer(2);
var hiddenLayer = new Layer(3);
var outputLayer = new Layer(1);
```

接下来，我们将这些层连接在一起并实例化一个新的网络，如下所示：


```javascript
inputLayer.project(hiddenLayer);
hiddenLayer.project(outputLayer);
var myNetwork = new Network({
 input: inputLayer,
 hidden: [hiddenLayer],
 output: outputLayer
});
```

这是一个2-3-1结构的神经网络，可视化后如下：

![image](https://cdn-images-1.medium.com/max/1600/1*IjY3wFF24sK9UhiOlf36Bw.png)

现在让我们来训练神经网络：


```javascript
// train the network - learn XOR
var learningRate = .3;
for (var i = 0; i < 20000; i++) {
  // 0,0 => 0
  myNetwork.activate([0,0]);
  myNetwork.propagate(learningRate, [0]);
  // 0,1 => 1
  myNetwork.activate([0,1]);
  myNetwork.propagate(learningRate, [1]);
  // 1,0 => 1
  myNetwork.activate([1,0]);
  myNetwork.propagate(learningRate, [1]);
  // 1,1 => 0
  myNetwork.activate([1,1]);
  myNetwork.propagate(learningRate, [0]);
}
```

我们共进行了20,000次的训练，每一次都进行四次正向传播和反向传播运算，分别传递四个可能的输入到神经网络：`[0,0] [0,1] [1,0] [1,1]`。

我们从 `myNetwork.activate([0,0])` 激活函数开始， `[0,0]` 是神经网络的输入值，这个过程是正向传播，也被称为激活网络。在每一次正向传播后我们需要做一次反向传播，从而更新神经网络的权重和偏差值。

反向传播通过下面这行代码实现`myNetwork.propagate(learningRate, [0])`， `learningRate` 是一个常数，用来告诉神经网络每次应该对权重值进行多大程度的调整。第二个参数 `0` 表示的是当输入为[0,0]时，正确的输出参数是0。

**然后，神经网络将预测值和真实值进行对比，来判断预测是否正确。**

它将比较的结果作为调整权重和偏差值的基础，以便下次的预测可以更加准确。

在执行这个过程20,000次后，我们可以通过传递四个可能的值输入到激活网络，从而判断目前神经网络的预测情况：


```javascript
console.log(myNetwork.activate([0,0])); 
-> [0.015020775950893527]
console.log(myNetwork.activate([0,1]));
->[0.9815816381088985]
console.log(myNetwork.activate([1,0]));
-> [0.9871822457132193]
console.log(myNetwork.activate([1,1]));
-> [0.012950087641929467]
```

如果我们将这些值四舍五入到最近的整数，就将得到异或方程的正确结果。万岁！

这就是全部内容了。尽管我们只是了解了神经网络的表面，但足够开始使用Synaptic，并继续学习。[++这里++](https://github.com/cazala/synaptic/wiki)包含了很多很好的教程。

最后，当你学习到新知识的时候，一定要分享，比如创建一个[++Scrimba++](https://scrimba.com/) screencast或写一篇文章！：）