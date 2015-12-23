{% raw %}

# Redux教程2：链接React



通过前面的教程，我们有了简单的环境，并且可以运行 `Redux` 的程序，也对 **如何编写Redux示例** 有了初步的印象；




掌握了 **使用Redux控制状态转移**，继而驱动 **React** 组件发生改变，这才是学习Redux的初衷。

本篇我们将 Redux 和 React 联合起来，着重讲解 `redux-react` 模块的使用；


## 1、编写红绿灯React组件

在原有的基础上，我们编写红绿灯组件：

```shell
touch components/light/index.js components/light/index.less
```

在 **components/light/index.js** 中写React代码，其结构非常简单：
```js
import React, { PropTypes, Component } from 'react'
import { render } from 'react-dom'
import classnames from 'classnames'
import './index.less'

class Light extends Component{
    render(){
        let color = this.props.light.color;
        return(
            <div className="traffic-light">
                <span className={classnames('light',color)} />
            </div>
        )
    }
}

Light.propTypes = {
    light: PropTypes.object.isRequired
}

Light.defaultProps = {
    light : {color:'red',time:'4'}
}

export default Light
```

根据更改样式类名（'red'、'green'、'yellow'），从而移动 **sprite图** 产生灯变换的效果：
```css
.traffic-light{
  .light{
    display: inline-block;
    background: url(//lh3.googleusercontent.com/-YWLqWZXDYHU/VmWC7GHoAuI/AAAAAAAACgk/nXvEmSWAhQU/s800/light.png) no-repeat 0 0;
    background-size: auto 100%;
    overflow: hidden;
    width:140px / 2;
    height:328px / 2;

    &.red{
      background-position: 0,0;
    }
    &.yellow{
      background-position: -78px , 0;
    }
    &.green{
      background-position: -156px , 0;
    }
  }
}
```

修改 **components/light/demo.js** 文件代码为：
```js
import React, {Component, PropTypes} from 'react'
import {render} from 'react-dom'
import Light from './index'

var color = 'red';

render(
    <div id="traffic">
        <Light color={color}/>
    </div>,
    document.getElementById('demo')
)
```

这样就能通过 http://localhost:3000/light/demo 预览这个组件了；

![demo light](http://ww2.sinaimg.cn/large/514b710agw1eyrez2zql0j207006tdfu.jpg)

## 2、链接React和redux

有了React和之前的Redux，现在就要将两者链接起来了。我们的目标是让红绿灯运行起来，就好比平时在十字路口看到的那样；


### 2.1、创建示例文件

再创建一个示例文件，就不叫demo了，叫做`redux`好了：
```sh
touch components/light/redux.js
```

> 之所以示例文件名称为 `demo.js` 或 `redux.js` ，是因为我在 **webpack.config.js** 中配置了，如果想用其他的文件名，只要依样画葫芦就可以；

首先在 **components/light/redux.js** 中输入最基本的脚手架代码，引入所需要的组件或模块：

```js
import React, {Component, PropTypes} from 'react'
import {render} from 'react-dom'
import { Provider, connect } from 'react-redux'
import { bindActionCreators } from 'redux'
import * as LightActions from '../../actions/light/'
import lightStore from '../../stores/light/'
import Light from './index'

// 声明store
let store = lightStore();

```

### 2.2、创建容器React

继而创建一个 **App React类** ，作为总的容器，将上述的 **Light** 组件放入其中：

```js
import React, {Component, PropTypes} from 'react'
import {render} from 'react-dom'
import { Provider, connect } from 'react-redux'
import { bindActionCreators } from 'redux'
import * as LightActions from '../../actions/light/'
import lightStore from '../../stores/light/'
import Light from './index'

// 声明store
let store = lightStore();

class App extends Component{
    _bind(...methods){
        methods.forEach((method)=>this[method] = this[method].bind(this));
    }
    constructor(){
        super();
        this._bind('autoChange','handleClick');
        this.state = {
            count : 0,
            timeId : null
        }
    }

    autoChange(){ // 自动更改红绿灯
        var _self = this;

        // 这里放置逻辑代码

        this.state.timeId = setTimeout(function(){
            // 递归调用，实现 setInterval 方法
            _self.autoChange();
        },1000);
    }
    handleClick(e){  // 用点击模拟红路灯

        if(this.state.timeId){
            clearTimeout(this.state.timeId);
            this.state.timeId = null;
        } else {
            this.autoChange();
        }

    }
    render(){
        // 通过connect 注入 redux 的 dispatch 方法
        return (
            <div id="traffic" onClick={this.handleClick}>
                <Light light={'yellow'}/>
            </div>
        )
    }
}

```

上面的代码还是个半成品，看不到效果；简单描述一下上面的代码做了什么：

 - 定义 `App` 容器，将 **Light** 组件放在其 `render` 方法中
 - **constructor** 方法引用了 **_bind**方法，方便一次性绑定 `this` 上下文，该方法来自文章 [Refactoring React Components to ES6 Classes](http://www.newmediacampaigns.com/blog/refactoring-react-components-to-es6-classes)
 - **handleClick** 方法是纯粹是为了演示，当用户点击红绿灯的时候，红绿灯调用 **autoChange方法** 开始自动变换，用户再次点击的时候就停止变换；
 - **autoChange** 方法用于红绿灯状态自动转换的，这里占位；本质是使用 `setTimeout` 代替 `setInterval` 实现；


### 2.3、链接React组件和Redux类

这是最为关键的一个步骤，
```js

  ...


class App extends Component{

    ...
}

// 声明 connect 连接
// 将 redux 中的 state传给 App
function mapStateToProps(state){
    return{
        light:state
    }
}

function mapDispatchToProps(dispatch){
    return{
        actions : bindActionCreators(LightActions,dispatch)
    }
}

// 声明 connect 连接
App = connect(mapStateToProps,mapDispatchToProps)(App);

// 真正的连接
render(
    <Provider store={store}>
        <App />
    </Provider>,
    document.getElementById('demo')
)
```

这里使用 **react-redux** 提供 `connect` 的方法 **链接React组件和Redux类**：

```js
// 声明 connect 连接
App = connect(mapStateToProps,mapDispatchToProps)(App);
```

 - **connect** 方法不会改变原来的组件类，反而返回一个新的 **已与 Redux store 连接的** 组件类。注意这里并没有注入 `store` 对象，真正 `store` 对象的注入靠最后的 `<Provider store>`组件；（更多说明请参考 [react-redux 的 API][1]）
 - 传入 **connect** 的 **mapStateToProps方法** ，正如其名，是将 Redux 的状态 映射到 React组件的props属性。任何时候，**只要 Redux store 发生改变，mapStateToProps 函数就会被调用**。这里返回对象是 `{light:state}`，这样确保 Redux 中的 state 发生改变时，组件的 props.light 都是最新的 Redux state。
 - **mapDispatchToProps方法** 则是将 Store 中的 dispatch方法 直接封装成对象的一个属性，一般会用到 Redux 的辅助函数 [bindActionCreators()](http://camsong.github.io/redux-in-chinese/docs/api/bindActionCreators.html)；这里将 `dispatch` 绑定到 `action`属性，这样在红绿灯组件内让其变成红灯的时候，不需要 `dispatch(changeRed())` 这么调用，直接使用 `actions.changeRed()`，语义化更好；（更多说明请参考 [react-redux 的 API][1]）
 - 最后的 `<Provider store>` 使组件层级中的 **connect()** 方法都能够获得 **Redux store**，这里才真正注入`store`变量，之前的只是声明而已（之前的好比store是个形参，到了这一步store就是实参了）。（更多说明请参考 [react-redux 的 API][1]）

经过上面的语句，Redux就将 **state属性** 、 （**store** 的） `dispatch方法` 与 R
eact 组件的 **props** 绑定在一起，凡是更改 **redux** 的 states，就会更新所连接组件的 `props` 属性。

> **react-redux** 中的 **connect** 方法就算是HOC（High Order Component，高阶组件）了，具体原理可参考文章 [初识React中的High Order Component](http://leozdgao.me/chushi-hoc/)，这是因为如果使用ES6 写React组件的话，mixin是不支持的
，因此使用High Order Component代替；

### 2.4、利用redux驱动react

理解了最为困难的部分，之后的事情就水到渠成了；

现在，只要记住 **在App中可以直接使用Redux中的一切了** 就行了

我们回过头来，完善 `App` 组件的代码，完善 **autoChange**方法：

```js

...

class App extends Component{
    _bind(...methods){
        methods.forEach((method)=>this[method] = this[method].bind(this));
    }
    constructor(){
        super();
        this._bind('changeColor','handleClick','autoChange');
        this.state = {
            count : 0,
            timeId : null
        }
    }
    changeColor(light,actions){ // 红路灯变换规则
        switch(light.color){
            case 'red':
                actions.changeGreen();
                break;
            case 'green':
                actions.changeYellow();
                break;
            case 'yellow':
                actions.changeRed();
                break;
            default:
                actions.changeRed();
        }       
    }
    autoChange(){ // 自动更改红绿灯
        const { light, actions } = this.props;
        let _self = this;

        let curCount = ++this.state.count;

        // console.log('xx,',curCount);
        if(this.state.count > +light.time){
            curCount = 0;
            this.changeColor(light,actions);
        }
        // 自动更改
        this.state.timeId = setTimeout(function(){
            _self.setState({count:curCount});
            _self.autoChange();
        },1000);

    }
    handleClick(e){  // 用点击模拟红路灯

        if(this.state.timeId){
            clearTimeout(this.state.timeId);
        } else {
            this.autoChange();
        }

    }
    render(){
        // 通过connect 注入 redux 的 dispatch 方法
        const { light, actions } = this.props;
        return (
            <div id="traffic" onClick={this.handleClick.bind(this)}>
                <Light light={light}/>
            </div>
        )
    }
}

...

```

至此已经完成本节示例，通过 `npm start` 开启服务， 在 http://localhost:3000/light/redux 中查看。

在这个示例里，通过点击红绿灯，每隔若干秒红绿灯就会变换颜色，这说明两者已经链接起来；

![demo light](http://ww1.sinaimg.cn/large/514b710agw1eys5p5yrptg208w0581kx.gif)

在后一篇文章，将示例如何处理多个Redux、React的情形；

[1] http://camsong.github.io/redux-in-chinese/docs/react-redux/api.html

{% endraw %}


