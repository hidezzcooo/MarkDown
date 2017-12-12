title:  使用 React 16 的 Portal 来做个很酷的效果
author:  范志强
date:  2017-12-7
categories:
- 前端技术
- React
- Javascript
tag:
- React 16 
---

>原文链接：https://hackernoon.com/using-a-react-16-portal-to-do-something-cool-2a2d627b0202

在React 16里，新增了个有趣的 `Portals` 。

`Portals` 让你可以在父组件之外渲染一些React控制的DOM。React [<u>官方文档</u>](https://reactjs.org/docs/portals.html) 用一个例子做了很好地解释。它也适用于做信息提示（这是作者之前做的 [<u>例子</u>](https://codepen.io/davidgilbertson/pen/ooXVyw) ）。

但我感觉也并没有什么意思。所以让我们来点干货...

因为 `Portals` 所做的是把一个元素添加到其他元素，并且不局限于在当前文档中。你可以将其添加到另一个文档中，也许是一个完全不同的窗口中的文件。


下面的图片中有一个页面（图片左边），上边有一个计数器和一个艳红色的按钮，还有一个窗口（图片右边）里边有着和左边页面相同的React程序的一部分，还有同样的背景（绿色的森林）。

事实上，右边的窗口是一个相同React程序，我想你应该已经感觉到有些激动了。

![image](https://cdn-images-1.medium.com/max/2000/1*ogsV-9IGNtaVjne2fb_oEA.png)

上图中的所有内容（树除外）代码都在下面的组件中。

```javascript
class App extends React.PureComponent {
  constructor(props) {
    super(props);
    
    this.state = {
      counter: 0,
      showWindowPortal: false,
    };
    
    this.toggleWindowPortal = this.toggleWindowPortal.bind(this);
  }

  componentDidMount() {
    window.setInterval(() => {
      this.setState(state => ({
        ...state,
        counter: state.counter + 1,
      }));
    }, 1000);
  }
  
  toggleWindowPortal() {
    this.setState(state => ({
      ...state,
      showWindowPortal: !state.showWindowPortal,
    }));
  }
  
  render() {
    return (
      <div>
        <h1>Counter: {this.state.counter}</h1>
        
        <button onClick={this.toggleWindowPortal}>
          {this.state.showWindowPortal ? 'Close the' : 'Open a'} Portal
        </button>
        
        {this.state.showWindowPortal && (
          <MyWindowPortal>
            <h1>Counter in a portal: {this.state.counter}</h1>
            <p>Even though I render in a different window, I share state!</p>
            
            <button onClick={() => this.setState({ showWindowPortal: false })} >
              Close me!
            </button>
          </MyWindowPortal>
        )}
      </div>
    );
  }
}
```

到此你可能发现 `<MyWindowPortal>` 有点特别，其中的任何内容都能够在不同的窗口中呈现。

恩！你是对的，你真棒！。`<MyWindowPortal>` 做了下面两件事：

>1. 当组件装载时打开一个新的浏览器窗口
>2. 创建一个 `<MyWindowPortal>` ，并将 `props.children` 添加到新的窗口主体。

这不是最酷的东西吗？

我兴奋的控制不住自己想去自由的飞翔。

.   .   .

我看到了一只烤鸭！

.   .   .


下面是上面组件的主体部分。代码的第11行就是新增的 `ReactDOM.createPortal` 方法---神奇之处就在这里。


```javascript
class MyWindowPortal extends React.PureComponent {
  constructor(props) {
    super(props);
    // STEP 1: create a container <div>
    this.containerEl = document.createElement('div');
    this.externalWindow = null;
  }
  
  render() {
    // STEP 2: append props.children to the container <div> that isn't mounted anywhere yet
    return ReactDOM.createPortal(this.props.children, this.containerEl);
  }

  componentDidMount() {
    // STEP 3: open a new browser window and store a reference to it
    this.externalWindow = window.open('', '', 'width=600,height=400,left=200,top=200');

    // STEP 4: append the container <div> (that has props.children appended to it) to the body of the new window
    this.externalWindow.document.body.appendChild(this.containerEl);
  }

  componentWillUnmount() {
    // STEP 5: This will fire when this.state.showWindowPortal in the parent component becomes false
    // So we tidy up by closing the window
    this.externalWindow.close();
  }
}
```
那这样有意义么？组件运行在别的地方且不返回任何东西。

也许有种想法是这样的：通常来说，一个父组件对一个子组件说："嘿！给我渲染一些DOM，然后把结果添加到我身上"，子组件按着父组件说的做了。但除此之外，这熊孩子可能这么说："不！我要在不同的窗口渲染东西，还得写一篇关于它的博客！"。

.   .   .

现在，我知道你在想什么。

你一定渴了，你想知道你是否得喝一杯。是的，去吧。

你可能会想到一件事：如果没有样式，将DOM注入到一个空白窗口中有什么意思？如果这是在 Craigslist 或者维基百科可能没人会注意到，但是你的网站这么漂亮怎能容忍，绝不能让弹出的聊天窗口都只能使用' times-new-romany '字体。

下面告诉大家一个好消息！

![image](https://cdn-images-1.medium.com/max/2000/1*eU-7ArIucnG5OreaPIJlEg.png)

起初我希望能有一种简单的方法将样式复制到新窗口中，但是没有发现。

由于我实在是太闲了，空虚充斥着我的生活，所以没事就写了下面这个很有趣的函数！


```javascript
function copyStyles(sourceDoc, targetDoc) {
  Array.from(sourceDoc.styleSheets).forEach(styleSheet => {
    if (styleSheet.cssRules) { // for <style> elements
      const newStyleEl = sourceDoc.createElement('style');

      Array.from(styleSheet.cssRules).forEach(cssRule => {
        // write the text of each rule into the body of the style element
        newStyleEl.appendChild(sourceDoc.createTextNode(cssRule.cssText));
      });

      targetDoc.head.appendChild(newStyleEl);
    } else if (styleSheet.href) { // for <link> elements loading CSS from a URL
      const newLinkEl = sourceDoc.createElement('link');

      newLinkEl.rel = 'stylesheet';
      newLinkEl.href = styleSheet.href;
      targetDoc.head.appendChild(newLinkEl);
    }
  });
}
```

这些 `styleSheet` 我并不是十分了解，所以我期待着在评论中得到更好的办法。

现在我可以在打开新窗口之后复制样式，就像这样：


```javascript
this.externalWindow = window.open(/* ... */);
                                  
copyStyles(document, this.externalWindow.document);
```

全部的代码在[++这里++](https://codepen.io/anon/pen/EoYKde)：

至此文章结束啦。

拜拜！