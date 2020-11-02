# Redux 本质论，第二章 Redux 应用框架
[原文: Redux Essentials, Part 2: Redux App Structure](https://redux.js.org/tutorials/essentials/part-2-app-structure)  
[前一章: Redux 本质论，第一章 Redux 概述及概念](./ReduxEssentials01.md)

1. 典型的 React + Redux 应用框架
2. 使用 Redux DevTools Extension 查看 state 变化

## 示例程序 Counter
来看一个小应用 [counter](https://codesandbox.io/s/github/reduxjs/redux-essentials-counter-example/tree/master/?from-embed)，可以点击加或减按钮来改变数字显示。虽然不大，但通过 action 的使用，展示了 React + Redux 应用的所有重要部分。  
用官方 Redux 模板 create-react-app 来创建项目。这个项目开箱即用，使用标准的 Redux 应用框架，用 Redux Toolkit 创建 Redux store 和逻辑，用 React-Redux 连接 React 组件和 Redux store。  
  
	npx create-react-app redux-essentials-example --template redux

### 把玩 Counter 应用
主要演示 Redux DevTools 的使用，及状态变化，略……

## 应用内容
这个应用由以下关键文件构成:

- /src
	- index.js: 应用入口点
	- App.js: 顶级 React 组件
	- /app
		-store.js: 创建 the Redux store 实例
	- /features
		- /counter
			- Counter.js: React 组件，根据 counter 的特性显示 UI
			- counterSlice.js: 根据 counter 特性实现的 Redux 逻辑

### 创建 Redux store

	{/* app/store.js */}

	import { configureStore } from '@reduxjs/toolkit'
	import counterReducer from '../features/counter/counterSlice'
	
	export default configureStore({
	  reducer: {
	    counter: counterReducer
	  }
	})

用 Redux Toolkit 中的 configureStore 创建 Redux store。 注意，configureStore 需要传入 reducer 参数。  
应用会有很多特性，每个特性可能会对应各自的 reducer 函数。当调用 configureStore 时，会一股脑儿地把这些所有不同的 reducer 放到一个对象中(key-value)，再把这个对象传给 configureStore。这些 reducer 对象对应的 key，也就成了最终 state 值的 key。  
`features/counter/counterSlice.js` 文件中导出的默认函数 slice.reducer 函数，实现了 counter 的逻辑。可以在创建 store 的文件中引入 `counterReducer`(即slice.reducer) 函数。  
当把 {counter: counterReducer} 对象传给 configureStore 函数时，其返回的 Redux state 对象就会包含 state.counter。之后，当接收到 dispatch 传过来的 action 时，counterReducer 函数就可以决定 state.counter 是否需要更新以及如何更新。  
Redux 允许 store 自定义配置各种不同的插件("中间件middleware"和"增强器(enhancers)"。configureStore 函数会自动向 store 配置中加入几个中间件，以提供更好地开发者体验。同时为了让 Redux DevTools Extension 能够侦测到其内容，也做了一些额外的配置。

### Redux Slices
"切片slice"是指一个应用特性对应的 Redux reducer 逻辑和 action 的集合，一般来说这些逻辑和 action 是定义在同一文件中的。切片这个名字源自将根 Redux state 对象拆分成 state 的多个"切片"。  
例如，博客应用中，store 配置可能会是这样的:

	import { configureStore } from '@reduxjs/toolkit'
	import usersReducer from '../features/users/usersSlice'
	import postsReducer from '../features/posts/postsSlice'
	import commentsReducer from '../features/comments/commentsSlice'
	
	export default configureStore({
	  reducer: {
	    users: usersReducer,
	    posts: postsReducer,
	    comments: commentsReducer
	  }
	})

这样，state.users、state.posts 和 state.comments 都是 Redux state 的独立的切片。

> ### 细节解释: reducer 和 state 结构
> Redux store 在创建时，得传入一个"根 reducer"函数。那么，如果有多个不同的 reducer 函数，该拿这个唯一的根 reducer 怎么办？如何定义 Redux store 的 state 内容呢？  
> 如果手动逐个调用所有的切片 reducer，就像下面这样:

	function rootReducer(state = {}, action) {
	  return {
	    users: usersReducer(state.users, action),
	    posts: postsReducer(state.posts, action),
	    comments: commentsReducer(state.comments, action)
	  }
	}

> 这种做法每次切片 reducer 的调用都是独立的，每次都把 Redux state 中特定的切片传递给 reducer，每次调用的返回值都是一个新的独立的 Reduce state 对象。  
> Redux 有个名为 combineReducers 的函数，接受包含多个切片 reducer 的对象作为参数，当接收到 dispatch action 时，返回一个函数调用每一个切片 reducer。combineReducers 会把从每个切片 reducer 得到的结果全部合并到一个 state 对象作为最终结果。用 combineReducers 来改造前面的示例:

	const rootReducer = combineReducers({
	  users: usersReducer,
	  posts: postsReducer,
	  comments: commentsReducer
	})

> 把这个合并后的对象作为根 reducer 传递给 configureStore:

	const store = configureStore({
	  reducer: rootReducer
	})

### 创建切片 reducer 和 action
下面详细研究下 `features/counter/counterSlice.js`

	{/*features/counter/counterSlice.js*/}

	import { createSlice } from '@reduxjs/toolkit'
	
	export const counterSlice = createSlice({
	  name: 'counter',
	  initialState: {
	    value: 0
	  },
	  reducers: {
	    increment: state => {
	      // Redux Toolkit allows us to write "mutating" logic in reducers. It
	      // doesn't actually mutate the state because it uses the immer library,
	      // which detects changes to a "draft state" and produces a brand new
	      // immutable state based off those changes
	      state.value += 1
	    },
	    decrement: state => {
	      state.value -= 1
	    },
	    incrementByAmount: (state, action) => {
	      state.value += action.payload
	    }
	  }
	})
	
	export const { increment, decrement, incrementByAmount } = counterSlice.actions
	
	export default counterSlice.reducer

点击界面中的不同按钮会 dispatch 三种不同的 action 类型:  

- `{type: "counter/increment"}`
- `{type: "counter/decrement"}`
- `{type: "counter/incrementByAmount"}`

action 是一个带有 type 字段的普通对象，type 字段的值是字符串。通常会用 action creator 函数完成 action 的构造工作。那改在何处定义这些 action 对象、type 字段和 action creator 函数呢？  
Redux Toolkit 有个名为 createSlice 的函数，可以自动完成诸如生成 action type 字符串、action creator 函数和 action 对象之类的工作。只要函数调用传入的对象中定义 slice name、编写 reducer 函数即可，这个函数会自动生成相关的 action 代码。其中 slice name 和 reducer 函数的 key 值将作为 action type 的第一、二部分。就是说，name 为 "counter" + "increment" reducer 函数，自动生成的 action type 为 {type: "counter/increment"}。  
除了 name 字段，createSlice 还需要为 reducer 提供一个 initialState 值。  
上面的代码共有三个 reducer 函数，对应着三个不同的 action type。  
createSlice 函数根据我们定义的 reducer 函数，自动生成与其同名的 action creator 函数。可以通过下面的代码来验证:
	
	console.log(counterSlice.actions.increment())
	// {type: "counter/increment"}

同时还自动生成了 slice.reducer 函数，这个函数知道如何根据 action type 调用相应的处理函数(reducer)

	const newState = counterSlice.reducer(
	  { value: 10 },
	  counterSlice.actions.increment()
	)
	console.log(newState)
	// {value: 11}

### reducer 规则
之前提到过，reducer 必须遵循一些特定的规则:

- 只根据取得的 state 和 action 参数计算新的状态，而与其他因素无关(即必须是纯函数)  
- 不得直接修改现有状态。即只能做不可变更新，需生成与现有状态相等的新状态对象，然后根据业务逻辑修改新状态对象
- 不能在函数内执行异步逻辑、计算随机值，或其他会造成"副作用"的操作

> 副作用  
> 如果一个函数修改了自己范围之外的资源，那就叫做有副作用，反之，就是没有副作用。


订立这样的规则有如下原因:

- Redux 的目标之一就是让代码的执行结果更加明确。如果一个函数的输出仅取决于其输入参数。那代码的运作就更加简单易懂，而且方便测试。
- 另外，如果函数依赖于外部变量，或是随机值，那函数的结果就没法预测。
- 如果函数修改了函数参数或函数外的其他值，这可能导致应用无法按预期运行。这可能是造成 bug 的根源，可能会发生"我更新了 state，但 UI 却没有跟着更新"之类的故事。
- reducer 不遵从这些规则，可能会破坏部分 Redux DevTools 的兼容性

"不可变更新"这条规则尤其重要，后面会继续探讨。

### reducer 和不可变更新
之前我们探讨过"可变"(直接修改现有对象/数组值)和"不可变性"(不去直接改变对象值，能直接改也不去改)

> 警告
> 在 Redux 中，reducer 永远也不允许直接去修改原始/当前 state 值!

在不改变原始值的前提下，如何返回更新过的状态呢？

> Tip
> reducer 只有新建一份原始值的副本，然后修改新的副本后，返回副本

前面使用展开操作符(...)来解决新建数组/对象副本的问题，但每次都这么做貌似有点low。  
手写不可变更新逻辑很繁琐，事实上，在 reducer 中直接修改 state 是 Redux 用户最常见的的错误。  
而 Redux Toolkit 的 createSlice 函数提供了不可变更新的简化方案。  
createSlice 里面使用了一个名为 Immer 的库。Immer 用到了一个名为 Proxy 的特殊的JS 工具。Poxy 可以包裹指定数据，让代码来修改包裹的数据。但 Immer 会跟踪所有的修改，然后用记录的修改行为列表来得到一个安全的不可变更新值，就好像手工写完了所有不可变更新逻辑。  
也就是，不必这样做:

	function handwrittenReducer(state, action) {
	  return {
	    ...state,
	    first: {
	      ...state.first,
	      second: {
	        ...state.first.second,
	        [action.someId]: {
	          ...state.first.second[action.someId],
	          fourth: action.someValue
	        }
	      }
	    }
	  }
	}

这样做就好:

	function reducerWithImmer(state, action) {
	  state.first.second[action.someId].fourth = action.someValue
	}

这样也更易读，但有一点必须记住:

> 警告
> 只能在 Redux Toolkit 的 createSlice 和 createReducer 中这样写"可变"逻辑，因为它们内置了 Immer。如果在没有内置 Immer 的 reducer 中这么写，大概率会出问题。


有了前面的说明，继续来看 counter 切片的代码

	{/*features/counter/counterSlice.js*/}

	export const counterSlice = createSlice({
	  name: 'counter',
	  initialState: {
	    value: 0
	  },
	  reducers: {
	    increment: state => {
	      // Redux Toolkit allows us to write "mutating" logic in reducers. It
	      // doesn't actually mutate the state because it uses the immer library,
	      // which detects changes to a "draft state" and produces a brand new
	      // immutable state based off those changes
	      state.value += 1
	    },
	    decrement: state => {
	      state.value -= 1
	    },
	    incrementByAmount: (state, action) => {
	      state.value += action.payload
	    }
	  }
	})

上面代码中的前两个 reducer，由于不需要使用 action 的数据，因此没有在 reducer 中声明。  
在 reducer incrementByAmount 中，由于要使用 action 中的数据，所以在 reducer 定义中声明了 state 和 action 两个参数。此时可知，UI 中的文本框数据将被放到 action.payload 字段中，因此需要将文本框的值放到 state.value 中。    

> 参考  
> [Immutable Update Patterns](https://redux.js.org/recipes/structuring-reducers/immutable-update-patterns)  
> [The Complete Guide to Immutability in React and Redux](https://daveceddia.com/react-redux-immutability-guide/)

### 使用 thunk 编写异步逻辑
到现在为止，所有的应用逻辑都是同步的。dispatch action、store 运行 reducer、计算新 state，直到 dispatch 函数结束。但 JavaScript 是支持异步代码的，从远端 API 获取数据的之类的需求，通常也是通过异步逻辑进行的。那 Redux 应用的异步逻辑放在什么位置呢？    
thunk 是一类特殊的 Redux 函数，可以容纳异步逻辑。编写 thunk 得用到两个函数:  

- 内部 thunk 函数，接收两个参数，一个是 dispatch 函数，另一个 getState 函数
- 外部构造函数，创建并返回 thunk 函数

下面的函数源自 couterSlice，给出了 thunk action creator 的示例:

	{/*features/counter/counterSlice.js*/}

	// The function below is called a thunk and allows us to perform async logic.
	// It can be dispatched like a regular action: `dispatch(incrementAsync(10))`.
	// This will call the thunk with the `dispatch` function as the first argument.
	// Async code can then be executed and other actions can be dispatched
	export const incrementAsync = amount => dispatch => {
	  setTimeout(() => {
	    dispatch(incrementByAmount(amount))
	  }, 1000)
	}

之后，就可以像使用普通 Redux action 一样使用这个异步 action 了:

	store.dispatch(incrementAsync(5))

注意，使用 thunk 需要在 Redux store 创建时传入 redux-thunk 中间件(一种 Redux 插件)。好在 Redux Toolkit 的 configureStore 函数已经自动完成了这个设置，因此在代码中直接使用就好，不必做额外的引入操作。  
使用 AJAX 调用以从服务器获取数据时，可以把这个 AJAX 调用放到 thunk 中。下面这个示例把函数的定义展开，虽然比上面的示例长了不少，但可以搞清楚其定义过程:  

	{/*features/counter/counterSlice.js*/}

	// the outside "thunk creator" function
	const fetchUserById = userId => {
	  // the inside "thunk function"
	  return async (dispatch, getState) => {
	    try {
	      // make an async call in the thunk
	      const user = await userAPI.fetchById(userId)
	      // dispatch an action when we get the response back
	      dispatch(userLoaded(user))
	    } catch (err) {
	      // If something went wrong, handle it here
	    }
	  }
	}


However, using thunks requires that the redux-thunk middleware (a type of plugin for Redux) be added to the Redux store when it's created. Fortunately, Redux Toolkit's configureStore function already sets that up for us automatically, so we can go ahead and use thunks here.

When you need to make AJAX calls to fetch data from the server, you can put that call in a thunk. Here's an example that's written a bit longer, so you can see how it's defined:



