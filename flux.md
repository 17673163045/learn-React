### 理解flux

```js
`react组件`直接传递'参数'或者'事件'都需要'props'一层层代理，对于复杂组件，它可能嵌套的子组件非常多，'层级也比较深'，那么，如果还采用props链条来维护组件通信或者数据共享，将非常困难，也'不利于开发和维护'。
```

```js
Flux框架也是一种MVC框架，不同于传统的MVC，它采用单向数据流，不允许Model和Control互相引用。
```

```js
Actions: 驱动Dispatcher发起改变,自定义对象,拥有type和value属性
Dispatcher: 负责分发动作（事件）
Store: 储存数据，处理数据
View: 视图部分
```

![](https://upload-images.jianshu.io/upload_images/25750-c70caaecac305f9e.png?imageMogr2/auto-orient/strip|imageView2/2/w/585/format/webp)

```js
安装:npm install flux
引入:import flux from "flux"
打印flux: console.log(flux)
```

![](.\images\flux.PNG)

```js

Dispatcher只会暴露一个函数dispatch,接受Action为参数，发起动作。
需要增加新功能，不需要改变或者增加接口，只需增加Action类型。
```

```js
// Dispatcher是flux的核心,事件调度器.
import { Dispatcher } from "flux"
// new一个Dispatcher对象
const dispatch = new Dispatcher();
// dispatch.register接受一个回调函数,参数是action
dispatch.register(callback)
//dispatch.dispatch(action)接受一个对象作为action,并触发register的相应的action
componentDidMount() {
     dispatch.register((action) => {
         switch (action.type) {
             case 'add':
                 console.log(action.value)
         }
     })
     dispatch.dispatch({ type: 'add', value: 111 })
}
```

```js
至此,我们了解了flux的Dispatch的基本用法:
    1.new Dispatch()对象有register方法,注册一个回调函数,利用switch来匹配		  action.type控制不同的行为.
    2.new Dispatch()对象有dispatch方法,传入一个action对象,触发register的相	     应的action事件.
```

### 管理state

```js
思路:
1. 定义数据仓库,即store对象.
1.1 store对象有state数据,即自定义的数据;
  有getState()方法,即return this.state,调用可以获取最新的state;
  有修改state数据的方法,自定义操纵数据的方法.
  有`upDate(cb)`方法,要封装Observer发布订阅模式,利用发布订阅通知视图更改.
2. 定义new Dispatch().register((action)=>{})注册action行为,匹配相应的行为,调用store的方法修改store的数据
3. 在组件调用dispatch.dispatch(action)触发action,修改了state的数据,但是视图不更新.
4. 要视图更新的话,封装observer发布订阅.
  在修改数据的方法中使用Observer.$emit("eventName");
  在store的`upDate(cb)`方法Observer.$on("eventName",cb);
4.1 在组件中定义handleUpdate函数,作为store的`upDate(cb)`的回调函数
    在组件的constructor传入回调函数:
    store.upDate(this.handleUpdate.bind(this));
```

#### store.js

```js
import Observer from "../Observer.js"
const store = {
    state: {
        num:0
    },
    getState() {
        return this.state
    },
    handleAdd() {
        this.state.num++;
        Observer.$emit("numAdd")
    },
    update(fn) {
        Observer.$on("numAdd",fn)
    }
}
export default store
```

#### index.js

```js
import store from "./store.js"
import { Dispatcher } from "flux"
const dispatch = new Dispatcher();
dispatch.register((action) => {
    switch (action.type) {
        case "NUM_ADD":
            store.handleAdd();
            break;
    }
})
export {
    store,
    dispatch
}
```

#### 组件中

```js
import React, { Component } from "react"
import {store,dispatch} from "./store/index.js"
class App extends Component {
    constructor() {
        super();
        this.state = {
            ...store.getState()
        }
        store.update(this.handleUpdate.bind(this))
    }
    render() {
        return (
            //点击flux的数据变化,视图更新.
            <button onClick={this.test}>{this.state.num}</button>
        )
    }
    test() {
        let action = {
            type: "NUM_ADD",
            value:0
        }
        dispatch.dispatch(action)
    }
    handleUpdate() {
        this.setState(store.getState())
    }
}
export default App
```













