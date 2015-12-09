# Flux 简介

[React 入门教程 —— Flux](http://hulufei.gitbooks.io/react-tutorial/content/flux.html)：相当于官方教程的中文版，建议先读这篇文章。

[Flux 傻瓜教程](http://zhuanlan.zhihu.com/FrontendMagazine/19900243)：难得的通俗易懂的好文章，这类文章往往能让你比较全面的看待问题。

[The Evolution of Flux Frameworks](https://medium.com/@dan_abramov/the-evolution-of-flux-frameworks-6c16ad26bb31)：讲解了Flux框架的演进路线。

[redux类库](https://github.com/gaearon/redux):Predictable state container for JavaScript apps。


[react-redux-starter-kit](https://github.com/davezuko/react-redux-starter-kit)：React + Redux + React-Router +  Koa 的综合教程。

[Redux basic tutorial](http://segmentfault.com/a/1190000003033033):文章通过写的一个简单 `counter` 的例子 介绍 `redux` 的核心方法以及一些需要注意的地方

[使用 React 和 Flux 创建一个记事本应用](http://www.atatech.org/articles/26612)

Flux 并不是真实存在的。它只是一个概念，而不是个类库。

Flux 的概念很简单，view 层触发了一个事件（比如说，用户在文本域中输入了一个姓名），这个事件更新了 model，然后 model 触发了一个事件，view 响应了 model 的事件，使用最新的数据进行渲染。就这样。

这一数据流/解耦观察者模式被设计来保证你的资源总存在于内存/模式中。这是一件好事™。

Flux 的坏处是每个人都会重新发明轮子。由于没有在事件库，model 层，AJAX 层等达成一致，出现了很多种“Flux”的实现方式，并且它们彼此之间相互混杂。

