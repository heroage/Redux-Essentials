# Redux 本质论，第一章 Redux 概述及概念
[原文链接: Redux Essentials, Part 1 Redux Overview and Concepts](https://redux.js.org/tutorials/essentials/part-1-overview-concepts)  

1. Redux 的概念、用途
2. 关键术语、概念
3. Redux 应用的数据流向

## 介绍
本教程将介绍 Redux，并教你如何利用推荐的工具和最好的技巧正确使用。完成教程后，应该可以使用学到的工具、模式构建自己的 Redux 应用。
本教程共分六章，本章涉及 Redux 的应知应会的关键概念和术语。第二章来看看如何将 React+Redux 应用捏合到一起。第三章将用学到的知识构建一个有点用的社交媒体反馈程序，来看看如何在实践中，让这些部件一起工作，同时会讨论一些 Redux 使用过程中的重要模式和原则。

### 如何阅读这份教程
在学习本教程前，需要了解以下知识

- Familiarity with HTML & CSS.
- Familiarity with ES6 syntax and features
- Knowledge of React terminology: JSX, State, Function Components, Props, and Hooks
- Knowledge of asynchronous JavaScript and making AJAX requests

在浏览器中安装以下插件

- React DevTools Extension:
    - [React DevTools Extension for Chrome](https://chrome.google.com/webstore/detail/react-developer-tools/fmkadmapgofadopljbjfkapdkoienihi?hl=en)
    - [React DevTools Extension for Firefox](https://addons.mozilla.org/en-US/firefox/addon/react-devtools/)
- Redux DevTools Extension:
    - [Redux DevTools Extension for Chrome](https://chrome.google.com/webstore/detail/redux-devtools/lmhkpmbekcpmknklioeibfkpmmfibljd?hl=en)
    - [Redux DevTools Extension for Firefox](https://addons.mozilla.org/en-US/firefox/addon/reduxdevtools/)

## Redux 是什么
Redux 是一个模式，也是一个库，通过名为 "action" 的事件，来管理、更新应用程序的状态。它为全局 state 维护了一个中心化 store，以便贯穿整个应用的始终，通过规则确保只能以预先规定的方式更新 state。

### Redux 的必要性
Redux 帮你管理"全局" state，这样 state 就可以根据需要在应用的各个部分便捷地共享。  
Redux 提供的模式和工具让应用 state 的更新变得容易理解(何时、何地、为何、如何)，同时状态改变后应用逻辑应该如何作用也变得更加清晰。Redux 让(state变更)代码的行为更加明确、方便测试，让你对应用的良好工作更有信心。

### 何时使用 Redux
- 应用中随处都需要用到大量的 application state
- application state 随时需要频繁更新
- 更新 state 的逻辑复杂
- 应用由多人维护的众多代码库构成

### Redux 库和工具
Redux 是独立的 JS 库，但通常也需要与其他包协同工作

- [React-Redux](https://react-redux.js.org/)
- [Redux Toolkit](https://redux-toolkit.js.org/)
- [Redux DevTools Extension](https://github.com/zalmoxisus/redux-devtools-extension)


## Redux 术语和概念
### 状态管理(State Management)
先看一个 React counter 组件。其组件状态跟踪了一个数字，按下按钮，数字会加1:


    function Counter() {
      // State: a counter value
      const [counter, setCounter] = useState(0)
    
      // Action: code that causes an update to the state when something happens
      const increment = () =>{
        setCounter(prevCounter => prevCounter + 1)
      }
    
      // View: the UI definition
      return (
        <div>
          Value: {counter} <button onClick={increment}>Increment</button>
        </div>
      )
    }
这是一个自包含的应用，可分为以下几部分:

1. state，是驱动应用的本源
2. view，基于当前 state，通过声明描述了用户界面(UI)
3. actions，应用中的事件基于用户输入，输入触发 state 更新


  
下图是"单向数据流"的一个简单示例:
![单向数据流](./images/part1-one-way-data-flow.png)

- state 描述了应用在特定时间点的状态
- 基于 state 渲染 UI
- 当某事发生时(如点击按钮)，state 根据触发事件更新
- 根据新的 state 重新渲染 UI

上面的一切看上去简单明了，但当复杂度增加，有多个组件需要共享相同的 state，尤其是这些组件分布在应用的不同部分时，这种简单性就被打破了。很多时候，需要通过"状态提升"来解决 state 共享的目的，但往往单纯的状态提升还不足以解决问题。
解决问题的方案之一是，从组件中抽离需要共享的 state，把共享状态集中放到组件树外面的一个位置。如此这般，我们的组件树就成了一个大单纯的"view"，组件树中的任何组件都可以方便地到 state 的集中存放点去存取 state 或是触发 action。
通过定义和切割 state 管理的相关概念，同时为了维护 view 和 state 之间的独立性，制定了一些强制规则，这样使得我们的代码变得更具结构性和可维护性。
这就是 Redux 背后的基本目的: 找个独立集中的地方存放应用的全局 state，同时依照规定的模式来更新 state，这样代码就更清晰，代码的执行结果也更易于预测。

### 不变性(Immutability)
"Mutable"是"可变的"。那如果有东西是"immutable"的，那这东西就不能被改变。
Javascript 对象和数组默认都是可变的。即使创建一个 const 对象，也可以改变其中字段的内容。

    const obj = { a: 1, b: 2 }
    // still the same object outside, but the contents have changed
    obj.b = 3
    
    const arr = ['a', 'b']
    // In the same way, we can change the contents of this array
    arr.push('c')
    arr[1] = 'd'

以上的例子就叫做可变对象或可变数组。就是说，内存中对象或数组的引用没有改变，但其中的内容已经改变。
要以不可变方式更新值，代码就得复制现有的对象/数组，然后修改新复制的对象/数组。
我们可以简单地使用 JavaScript 的数组/对象展开(...)运算符，这样就可以得到原来数组/对象的新副本，而不是原数组/对象的引用。

    const obj = {
      a: {
        // To safely update obj.a.c, we have to copy each piece
        c: 3
      },
      b: 2
    }
    
    const obj2 = {
      // copy obj
      ...obj,
      // overwrite a
      a: {
        // copy obj.a
        ...obj.a,
        // overwrite c
        c: 42
      }
    }
    
    const arr = ['a', 'b']
    // Create a new copy of arr, with "c" appended to the end
    const arr2 = arr.concat('c')
    
    // or, we can make a copy of the original array:
    const arr3 = arr.slice()
    // and mutate the copy:
    arr3.push('c')
Redux 希望所有的 state 更新都是不可变的。我们稍后就会看到在何处做，以及这一点的重要性，同时也有一些便捷地方法来实现不可变更新逻辑。   

>延伸阅读  
[A Visual Guide to References in JavaScript](https://daveceddia.com/javascript-references/)  
[Immutability in React and Redux: The Complete Guide](https://daveceddia.com/react-redux-immutability-guide/)

## 术语
### action
action 是一个普通的 JavaScript 对象，包含一个 type 字段。可以把 action 认为是一个事件，它描述了应用中发生的一些情况。
type 字段是字符串，其值给出了对 action 的描述性名称，诸如 "todos/todoAdded" 之类。我们通常会为 type 指定类似于 "domain/eventName" 的值，第一部分是 action 的特征或所属的分类，第二部分是发生的特定事件。
action 对象可以用其他字段来描述发生的额外信息。习惯上，我们把这个字段定义为 "payload"。
典型的 action 对象如下:  

    const addTodoAction = {
      type: 'todos/todoAdded',
      payload: 'Buy milk'
    }

### Action Creator
actoin creator 是一个函数，用来创建并返回 action 对象。我们使用这些函数获取 action 对象，而不必每次都手工来写一遍 action 对象:  

    const addTodo = text => {
      return {
        type: 'todos/todoAdded',
        payload: text
      }
    }

### Reducers
reducer 是一个函数，接受当前状态和一个 action 对象作为参数，决定在需要时如何更新 state，并返回新 state。其函数原型为: (state, action) => newState。可以把 reducer 看做是一个事件监听器，它会根据接收到的 action(事件) 类型对事件做相应的处理。

> "reducer" 函数之所以得名，是因为其原型跟传给 Array.reduce() 方法的回调函数类似。



reducer 必须遵循以下特定规则:

- 只根据当前取得的 state 和 action 参数计算新的状态，而与其他因素无关(即必须是纯函数)  
- 不得直接修改当前状态。即只能做不可变更新，需生成与当前状态相等的新状态对象，然后根据业务逻辑修改新状态对象
- 不能在 reducer 内执行异步逻辑、计算随机值，或其他会造成"副作用"的操作

> 副作用  
> 如果一个函数修改了自己范围之外的资源，那就叫做有副作用，反之，就是没有副作用。

稍后会继续讨论 reducer 规则，包括其为何如此重要，以及如何正确地遵从这些规则。  
reducer 函数中的逻辑通常遵循以下步骤:  

- 检查 reducer 是否需要处理这个 action
	- 如果需要处理这个 action，就复制当前 state 得到新的状态对象，然后更新新对象中的值后，返回新对象作为新状态
- 直接返回现有 state

下面是一个 reducer 的示例，演示了 reducer 应遵循的步骤:

	const initialState = { value: 0 }

	function counterReducer(state = initialState, action) {
	  // Check to see if the reducer cares about this action
	  if (action.type === 'counter/increment') {
	    // If so, make a copy of `state`
	    return {
	      ...state,
	      // and update the copy with the new value
	      value: state.value + 1
	    }
	  }
	  // otherwise return the existing state unchanged
	  return state
	}

reducer 函数中可以使用 if/else、switch、loops 在内的任何逻辑

> ### 详细解释: 缘何名为 reducer?
> Array.reduce() 方法遍历数组的每个值，每次只处理数组的一项，最终返回唯一的最终结果。你可以认为这是"将数组缩减为一个值"。  
> Array.reduce() 接受一个回调函数作为参数，在遍历过程中对数组的每一项都调用这个回调函数。这个回调函数接受两个参数:
>   
> - previousResult: 回调函数上次的返回值
> - currentItem: 当前遍历值 
>
> 数组第一次遍历，回调函数被调用时，previousResult 是没有值的，因此需要传给它一个初始值，以便回调函数能正常调用。
> 下面的代码演示了如何用 reduce 回调函数实现数组值的累加:

	const numbers = [2, 5, 8]
	
	const addNumbers = (previousResult, currentItem) => {
	  console.log({ previousResult, currentItem })
	  return previousResult + currentItem
	}
	
	const initialValue = 0
	
	const total = numbers.reduce(addNumbers, initialValue)
	// {previousResult: 0, currentItem: 2}
	// {previousResult: 2, currentItem: 5}
	// {previousResult: 7, currentItem: 8}
	
	console.log(total)
	// 15

> 注意，addNumber 函数是一个 reduce 回调函数，addNumber 函数本身不必关心遍历过程。函数接受 previousResult 和 currentItem 这两个参数，运算后返回新的结果值。
> Redux reducer 函数与 reduce 回调函数完全相同。它也接受两个参数，一个是"之前的结果"(即 state)，另一个是"当前项"(即 action 对象)，这两个参数决定了新 state 值，reducer 函数也返回这个新 state 值。
> 如果创建一个 Redux action 数组，然后将一个 Redux reducer 传给 Array.reduce()，也同样能得到最终结果:

	const actions = [
	  { type: 'counter/increment' },
	  { type: 'counter/increment' },
	  { type: 'counter/increment' }
	]
	
	const initialState = { value: 0 }
	
	const finalResult = actions.reduce(counterReducer, initialState)
	console.log(finalResult)
	// {value: 3}

> 可以说 Redux reducer 将一堆 action 缩减(reduce)到一个唯一的状态。二者的不同在于，Array.reduce() 一次性完成缩减，而 Redux reducer 则需要在应用程序的整个生命周期中不断执行。

### store
当前 Redux 的 application state 存储在一个叫做 store 的对象中。  
创建 store 需要传进 reducer 作为参数，其 getState 方法返回当前 state 值


	import { configureStore } from '@reduxjs/toolkit'
	
	const store = configureStore({ reducer: counterReducer })
	
	console.log(store.getState())
	// {value: 0}

### Dispatch
Redux store 有一个名为 dispatch 的方法。更新 state 的唯一方法就是调用 store.dispatch()，并把 action 对象作为参数传给 dispatch 方法。接下来，store 会运行其 reducer 函数，并在内部保存新的 state 值，之后，可以用 getState() 方法取得更新后的 state 值:

	store.dispatch({ type: 'counter/increment' })
	
	console.log(store.getState())
	// {value: 1}

在应用中，可以把 dispatch action 看做"触发事件"，而 store 需要通过 action 了解发生事件的信息。reducer 的作用类似于事件监听器，当它们监听到需要它们处理的 action，它们会在反馈中返回更新的状态。  
通常调用 action creator 来 dispatch 相应的 action:

	const increment = () => {
	  return {
	    type: 'counter/increment'
	  }
	}
	
	store.dispatch(increment())
	
	console.log(store.getState())
	// {value: 2}

### Selector
selector 是一个函数，用以提取并返回 store 中 state 的特定信息。当应用变大时，应用这个函数可以避免在应用的不同部分，重复编写读取同一数据的逻辑代码。

	const selectCounterValue = state => state.value
	
	const currentValue = selectCounterValue(store.getState())
	console.log(currentValue)
	// 2

### Redux Application Data Flow
前面回顾 React 概念时，曾提到过"单向数据流"，它描述了 application state 更新的先后次序:

- state 描述了应用在特定时间点的状态
- 基于 state 渲染 UI
- 当某事发生时(如点击按钮)，state 根据触发事件更新
- 根据新的 state 重新渲染 UI

下面我们针对 Redux 来继续细化这些步骤:

- 初始化设置:
	- 创建 Redux store 时，以根 reducer 函数作为参数传入
	- store 一旦调用了根 reducer，就保存根 reducer 的返回值作为其初始状态
	- 当第一次渲染 UI 时，UI 组件会访问 Redux store 的当前 state，使用其中的数据进行渲染。UI 组件会 subscribe 相关的 store 更新，以便在 state 更新时获取通知。
- 更新:
	- 应用事件发生，诸如点击按钮之类
	- 应用代码 dispatch action 到 Redux store ，诸如 dispatch({type: 'counter/increment'}) 之类
	- store 运行 reducer 函数，一遍遍地传入 previous state 和 current action，并返回计算得到的新 state
	- store 通知所有 subscribe 了 store 更新的所有 UI 组件
	- 得到更新通知的每个 UI 组件，主动向 store 要求检查其需要的 state 数据是否已经发生改变
	- 每个发现数据已经改变的组件，强制用新数据重新渲染，然后更新结果就呈现在屏幕上了


下面是 Redux 数据流图示:  
![Redux 数据流图示](./images/part1-ReduxDataFlowDiagram.gif)


## 总结
Redux 有大量的新术语和概念需要记住。前面提到的相关内容总结如下:  
  
- Redux 是一个库，用以管理应用的全局状态
	- React-Redux 库将 Redux 和 React 整合到一起
	- 建议使用 Redux Toolkit 编写 Redux 逻辑
- Redux 使用"单向数据流"的应用框架
	- state 描述了特定时间点的应用状态，UI 是基于 state 渲染出来的
	- 当应用事件发生时:
		- UI 组件 dispatch action
		- store 运行 reducer，state 根据发生了的事件(action)进行更新
		- store 通知 UI 组件 state 已经更新
	- UI 根据新的 state 重新进行渲染
- Redux 的几种代码
	- action: 普通的对象，必须带有 type 字段，用以描述应用中到底"发生了什么"
	- reducer: (纯)函数，基于 previous state 和 action 计算 new state 
	- 每当 dispatch action 时，store 就会调用根 reducer


[下一章: Redux Essentials, Part 2 Redux App Structure](./ReduxEssentials02.md)