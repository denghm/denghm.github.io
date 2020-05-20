---
layout: article
title: "react-redux框架的状态传递"
modified: 2020-04-01
author: Voyager
toc: false
comments: true
categories: technology
summary: React-Redux的状态传递相当晦涩，React组件自身的状态，Redux组件的全局状态，不同的component生成方式，这些都带来了状态传递复杂多样。
---

# React组件的状态
# Redux组件的状态
redux组件提供的了状态的统一存储，所有状态存放于store.js, 一般采用以下方法创建

```javascript
export default createStore(reducer, {}, storeEnhancers);
```

这些的第二个参数，**“{}”**， 实现了状态存储的初始化

不同的reducer, 生成的状态都存放于状态存储中

# React组件如何获取Redux组件存放的状态

## 函数创建React组件



```javascript
const Weather = ({status, cityName, weather, lowestTemp, highestTemp}) => {
  switch (status) {
    case Status.LOADING: {
      return <div>天气信息请求中...</div>;
    }
    case Status.SUCCESS: {
      return (
        <div>
          {cityName} {weather} 最低气温 {lowestTemp} 最高气温 {highestTemp}
        </div>
      )
    }
    case Status.FAILURE: {
      return <div>天气信息装载失败</div> 
    }
    default: {
      throw new Error('unexpected status ' + status);
    }
  }
}
```

这里匿名函数的{status, cityName, weather, lowestTemp, highestTemp} 获取了store.js中的状态存储，而且状态存储（实际就是dict）已经被展开了, 原因在于reducer返回的状态采用了扩展运算符操作state --**" ...state"**

```javascript
return {...state, status: Status.SUCCESS, ...action.result}
```



函数方式创建额react组件，当每次组件要刷新展示时就会调用该函数，所以能确保每次都可以获得最新设置的全局状态



## React类

React类比较复杂，全局的状态需要通过props传递给React组件，也就是需要实现mapStateToProps函数，比如

```javascript
const mapStateToProps = (state) => {
  return {
    registry: state.registry
  }
};
```

而且在connect函数中以该函数作为参数

```javascript
export default connect(mapStateToProps, mapDispatchToProps)(Registry);
```

这样React组件中就可以通过props来获取最新的全局状态。

容易引起混淆的地方来了

### React组件自身的state与全局state

因为react组件往往有表单组件，这时为了绑定表单的输入，需要有内部定义的state, 比如shouldComponentUpdate的第二个参数是react组件的state。为什么，是用的state字段名进行选取的吗？目前还没有分析清楚

```javascript
shouldComponentUpdate(nextProps, nextState)
```



需要与用户交互的控件，比如Input表单，状态用内部的state做绑定；

不需要与用户交互的状态，比如列表，状态与全局的state做绑定



### React组件自身的state与form表单input的绑定

state与input的绑定都是采用



### React组件获取全局状态

React组件的constructor只在类初始化时调用一次，这里无法获取最新的全局状态（对不？）

所以必须找一个react生命周期函数，把全局状态设置到React自身状态state上，然后React组件刷新时才可以自身的state来刷新显示

因为render()函数渲染界面从一个地方获取数据，第一次界面初始化时，从state获取；第二次从store加载到数据后，再进行渲染，需要把store数据设置到state中

而且对数据变化进行记录，但store数据不能更新，所以在通过界面操作数据时，比如翻页，这涉及当前页等动态变化的，只能把store数据放入React组件的进行操作

在render()渲染前调用props -> state的映射，目前没有合适的生命周期函数

1. mapStateToProps() 

   可以通过函数把store映射为props, 但无法把props映射为内部state

2. shouldComponentUpdate()

   在props全局数据变化或state局部数据变化时都得到调用，而最好是只有props全局数据变化时调用，故只有自己注册一个store的监听函数








# React-Redux框架

## 基本元素

* 静态界面与状态组成应用

* 全局状态

* 组件交互
  * 状态传递
  * 事件传递
  * 人机交互
* 刷新