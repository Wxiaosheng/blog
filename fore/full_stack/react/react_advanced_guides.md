1. [高级指引](/fore/full_stack/react/react_advanced_guides?id=高级指引)
2. [Context](/fore/full_stack/react/react_context)




## 高级指引
#### 代码分割
&emsp;&emsp;打包是从一个文件开始，找到其所有有关的文件，并最终合并到一个文件中。   
&emsp;&emsp;将打包的结果分成多个包文件并在运行时动态加载。  
&emsp;&emsp;对应用进行代码分割能够帮助你"**懒加载**"当前用户所需要的内容，能够显著提高应用的性能。尽管并没有减少应用的整体的代码体积，但是**能够避免加载用户永远也用不到的代码，并在初始加载的时候减少所需的代码量**。  

#### import()
&emsp;&emsp;在应用中引入代码分割的最佳方式是通过动态 import() 语法。  
&emsp;&emsp;注意：  
&emsp;&emsp;&emsp;&emsp;1、动态 import() 语法目前只是一个 ECMAScript 提案，而不是正式的标准语法。  
&emsp;&emsp;&emsp;&emsp;2、如果使用 Create React App 该功能已经配置好，如果是自己搭建的 Webpack，则需要进行代码分割的配置。  
&emsp;&emsp;&emsp;&emsp;3、当使用babel时，要使用 babel-plugin-syntax-dynamic-import 插件 解析动态 import 语法，而不是对其进行转换。  

#### React.lazy
&emsp;&emsp;&emsp;&emsp;React.lazy 函数能让你像渲染常规组件一样处理动态引入(的组件)  
&emsp;&emsp;&emsp;&emsp;React.lazy 接受一个函数，该函数需要动态的调用 import()，它必须返回一个 Promise，该 Promise 需要 resolve 一个 default export 的 React 组件。  
&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;**const OtherComponent = React.lazy(() => import('./OtherComponent'))**  
&emsp;&emsp;此代码将**在组件首次渲染时，自动导入包含 OtherComponent 组件的包**。  
&emsp;&emsp;应在 Suspense 组件中渲染 lazy 组件，fallback 属性接受任何在组件加载过程中你想展现的 React 组件，如此使得在等待加载 lazy 组件时做优雅降级。  
&emsp;&emsp;Suspense 组件置于懒加载组件之上的任何位置，你甚至可以用一个 Suspense 组件包裹多个懒加载组件，通常我们一般放置在最外层即可。

##### 异常捕获边界
&emsp;&emsp;如果模块加载失败(网络问题)，它会触发一个错误。可以通过 **异常捕获边界(Errorboundaries)** 技术来处理这些情况，给用户以良好的体验。  
##### 基于路由的代码分割
&emsp;&emsp;决定在哪里引入代码分割需要一些技巧，因为需要确保选择的位置能够均匀地分割代码包而不会影响用户体验。一个不错的选择就是从路由开始，大多数网络用户习惯于页面之间能有个加载切换的过程。   


### 错误边界
&emsp;&emsp;部分 UI(组件) 的 JavaScript 错误不应该导致整个应用的崩溃，为了解决这个问题，React 16 引入了一个新的概念，-- 错误边界(Error Boundary)。  
&emsp;&emsp;边界错误是一种 React 组件，这种组件**可以捕获并打印发生在其子组件树任何位置的 JavaScript 错误**，并且它会渲染出备用 UI，而不会渲染那些崩溃了的子组件树。

&emsp;&emsp;注意：(错误边界 **无法捕获** 以下场景中产生的错误)  
&emsp;&emsp;&emsp;&emsp;1、事件处理([了解更多][1])  
&emsp;&emsp;&emsp;&emsp;2、异步代码(例如 setTimeout 或 requestAnimationFrame 回调函数)  
&emsp;&emsp;&emsp;&emsp;3、服务端渲染  
&emsp;&emsp;&emsp;&emsp;4、自身抛出的错误(并非它的子组件)  

&emsp;&emsp;是用一个 class 组件定义了 static getDerivedStateFromError() 或 componentDidCatch() 这两个生命周期方法中的任意一个(或两个)时，那么它就变成了一个错误边界。  
&emsp;&emsp;通常情况下，**在发生错误后，使用 static getDerivedStateFromError() 渲染备用UI，使用 componentDidCatch() 打印或上报错误信息**。   
&emsp;&emsp;👇使用方法请参照下面的代码： 
```javascript
// 错误边界组件
import React, { Component } from 'react'

export default class ErrorBoundary extends Component {
  constructor(props){
    super(props)

    this.state = {
      hasError: false,
      errorInfo: {}
    }
  }
  static getDerivedStateFromError(error){
    // 处理需要展示给用户的友好错误提示信息
    console.log('getDerivedStateFromError')
    console.log(error)
    return { hasError: true, errorInfo: error }
  }

  componentDidCatch(error, errorInfo){
    // 做错误事件的上报
    console.log('componentDidCatch')
    console.log(error, errorInfo)
  }

  render(){
    return <div style={{ fontSize: '0.16rem' }}>
      {this.state.hasError ? `请联系网站维护人员处理！ ${this.state.errorInfo.toString()}` : this.props.children}
    </div>
  }
}

// 通常放在在路由层级做统一处理，这样在路由下面的所有子组件在渲染时的错误都会被错误边界捕获，而处理成友好的响应形式
import React, { Component } from 'react'
import { Switch, Route } from 'react-router-dom'
import ErrorBoundary from '../pages/ErrorBoundary'
import Home from '../pages/Home'
import List from '../pages/List'

export default class Router extends Component {
  render() {
    return (
      <ErrorBoundary>
        <Switch>
          <Route path="/" exact component={Home} />
          <Route path="/list" exact component={List} />
        </Switch>
      </ErrorBoundary>
    )
  }
}
```

#### react 常见的错误有
> 自己总结归纳的  

&emsp;&emsp;1、**渲染时的错误**，比如通过数据渲染 UI 时，如数据结构缺失(array.length 的 array 为 undefined)时，这个时候代码报错，默认不处理情况下，**页面会直接白屏**，体验非常不友好(注意，**如果你是在测试环境，那么页面会出现错误，但是如果打包发布到生产环境上就是直接白屏，同一样的错误，测试环境和生产环境的表现会不同**)  

&emsp;&emsp;2、**异步setTimeout 和 事件处理 发生的错误**，这种错误只是 javascript 代码报错，只会在控制台显示报错信息，最多导致点击无响应，并不会触发 React 重新渲染DOM，因而 React 仍然能够知道需要在屏幕上显示什么。  

&emsp;&emsp;3、事件处理 导致的数据变化，触发了 React 重新渲染DOM，要注意**这个时候的错误其实是渲染时的错误，而不是 事件处理的错误**，因此参照 1 即可。

&emsp;&emsp;综上，在自己项目中如何做错误处理，**定义并使用错误边界来处理渲染时的错误，而JavaScript 错误**(异步setTimeout 和 事件处理 发生的错误) **因为不会影响UI的展示而无需做错误边界的同样处理**。









<p align="right"> 2019年11月28日 </p>

[1]:https://zh-hans.reactjs.org/docs/error-boundaries.html#how-about-event-handlers