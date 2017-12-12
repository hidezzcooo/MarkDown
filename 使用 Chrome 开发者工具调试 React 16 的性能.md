title:  使用 Chrome 开发者工具调试 React 16 的性能
author:  范志强
date:  2017-12-8
categories:
- 前端技术
- React
- Javascript
- Chrome
tag:
- React 16 
- Chrome
---

>原文链接：https://building.calibreapp.com/debugging-react-performance-with-react-16-and-chrome-devtools-c90698a522ad

React是重要的前端框架之一，它的特点是其渲染性能。React的虚拟dom以高效渲染组件而闻名-当组件在毫无察觉的情况下快速渲染的时候发生了什么？

**今天，我将结合React和Chrome开发工具来演示强大的新性能跟踪功能，以及如何使用它们来找出那些渲染缓慢的组件**。

这个新性能工具有三个要求：

- 在开发模式下使用React 16
- 通过source-maps导出你的JS（可选，但*强烈推荐*）
- 使用Google Chrome或Chromium的开发工具 （可用于所有当前版本）

.   .   .

### 👷♀配置

首先，你需要一些空间来施展，所以去开发者工具里设置一下吧。爽！

![image](https://cdn-images-1.medium.com/max/1600/1*-vtAWWryE6dtsPXkuOU7xQ.gif)

在我们进行性能审查的时候，必须要明白一个道理，并不是每个人都有一个3000刀的电脑或者手机，我们需要模拟一些情况。

幸运的是，Chrome里可以做到这一点-我们将放慢JavaScript的执行速度。这么做会使性能问题更加明显，如果你可以在较慢的设备上使其速度变快，那么在桌面上运行的速度会更加快。

![image](https://cdn-images-1.medium.com/max/1600/1*aL8cq4bJxsH4RcTFblGfBg.gif)

将目前的型号的Macbook的速度降低至少4倍，这样的表现会类似于摩托罗拉的Moto G4。

---

### 🔬查看性能轨迹

在开发者模式中，随着React 16渲染，它为每个组件创建“标记和度量”事件。

要创建性能分析器审查，请打开你想要测试的页面（在本地主机），然后按“启动分析和重新加载页面”按钮。

它将记录当前页面的性能轨迹。 Chrome会在页面解决后自动停止记录。

![image](https://cdn-images-1.medium.com/max/1600/1*KXQKkbo8Ujfd8JcRDK9RAA.gif)

一旦实现了跟踪，你的窗口将如下所示：

![image](https://cdn-images-1.medium.com/max/2000/1*JH7OdBXRtDjxLUqGT3b51A.png)

有些人对“性能”选项卡可能还有困惑，我觉得我得花些时间来说明一下。

1. 这个红色条形指示器显示在这部分跟踪时间线周围有一些重要的“CPU消耗”（长时间的任务）。我们可能需要查看一下。
2. 性能窗口顶部图形中使用的颜色对应于不同类型的活动。每个类型都有自己的作用。

今天我们关注的是“ Scripting ”（Javascript运行时的性能）。


![image](http://wx2.sinaimg.cn/large/3fc2eae1ly1fme2axha9cg217e13gnpi.gif)


现在我们要调查那个红色的CPU消耗区。我们可以查看页面在跟踪期间呈现的元素。

![image](https://cdn-images-1.medium.com/max/2000/1*CsggR2GWwz_E7P68sYGJJA.gif)

缩放会立即呈现用户计时信息和一个标记为“脉冲”的组件（需要500毫秒渲染）。

在脉冲组件的下面，还有一些执行速度很快的子组件渲染。

### ⏳发现执行缓慢的函数

![image](https://cdn-images-1.medium.com/max/2000/1*q5jXMcIIefR9e_Vhc-kXRQ.png)

1. 点击你想要查看的组件，下半部分窗口只关注脉冲组件的活动。
2. 选择“ Bottom-Up ”。
3. 按总时间降序排序还是按照本体时间进行排序或者按照URL分组排序，完全看你的需要。
4. 点击开箭头，直到你找到想要查看的部分。现在看来 `map` 很可疑。它被执行了很多次，执行时间达到90毫秒。
5. **这就是为什么你需要 sourcemaps ：单击 `MetricStore.js:59` 我们就可以马上找到它！**

![image](https://cdn-images-1.medium.com/max/1600/1*X-Wkg_xcIr3cJEDMFhav6A.png)

##### 了解后改进代码是很容易的

过去的几周一直使用这种方法管理我的代码，我认为“ 这很复杂，所以我需要一点时间”。现在我知道怎样寻找JavaScript的性能问题。我肯定在未来可以编写更好的UI代码。🙈

### 🛣旁注：[++Calibre++](https://calibreapp.com/)监控JavaScript的解析，编译和运行时间

如果对你的网站在加载广告和跟踪像素以及应用内聊天时发生的情况有些疑问，你可以使用 Calibre 清楚的了解是哪些脚本（或服务）耗费你客户的时间。

🏊 [Calibre 免费试用14天](https://calibreapp.com/)，赶快尝鲜。

![image](https://cdn-images-1.medium.com/max/1600/1*Z7RFbdAMiWJwDNXDW6UAPA.png)

### 为什么Chrome中内置了React特定的工具？

React通过 [User Timing API](https://developer.mozilla.org/en-US/docs/Web/API/User_Timing_API) 发布这些指标，也可以用来在你的代码周围放置时间标记。

*考虑“组件在0.4秒内初始化”或“用户花了15秒钟按下购买按钮”。*

有关 User Timing API 的介绍，请参阅 [Alex Danilo](https://twitter.com/alexanderdanilo) 自2014年起正确命名的HTML5 Rocks文章“ [User Timing API](https://developer.mozilla.org/en-US/docs/Web/API/User_Timing_API) ”。

.   .   .

所有主流浏览器都支持 User Timing API ，但 Chrome 性能标签要让 React 应用程序的调试变得更加简单还有很长的一段路要走。

.   .   .

### 优化你的Javascript

真诚地希望这篇文章能帮助你提高你的性能审查技能。请留下评论，讨论你所学到的知识，或者在应用程序上所做的改进。尽情享受🙋
