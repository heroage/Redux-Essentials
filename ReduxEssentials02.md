# Redux 本质论，第二章 Redux 应用框架
[原文: Redux Essentials, Part 2: Redux App Structure](https://redux.js.org/tutorials/essentials/part-2-app-structure)  
[前一章: Redux 本质论，第一章 Redux 概述及概念](./ReduxEssentials01.md)

1. 典型的 React + Redux 应用框架
2. 使用 Redux DevTools Extension 查看 state 变化

## 示例程序 Counter
来看一个小应用 [counter](https://codesandbox.io/s/github/reduxjs/redux-essentials-counter-example/tree/master/?from-embed)，可以点击加或减按钮来改变数字显示。虽然不大，但其通过 action 的使用，展示了 React + Redux 应用的所有重要部分。  
用官方 Redux 模板 create-react-app 来创建项目。这个项目开箱即用，使用了标准的 Redux 应用框架，用 Redux Toolkit 创建 Redux store 和逻辑，用 React-Redux 连接 React 组件和 Redux store。  

```bash
npx create-react-app redux-essentials-example --template redux
```

### 把玩 Counter 应用
主要演示 Redux DevTools 的使用，及状态变化，略……

## 应用内容
这个应用由以下关键文件构成:

- /src
	- index.js: 应用入口点
	- App.js: 顶级 React 组件
	- /app
		- store.js: 创建 the Redux store 实例
	- /features
		- /counter
			- Counter.js: React 组件，根据 counter 的特性显示 UI
			- counterSlice.js: 根据 counter 特性实现的 Redux 逻辑

### 创建 Redux store

```javascript
{/* app/store.js */}

import { configureStore } from '@reduxjs/toolkit'
import counterReducer from '../features/counter/counterSlice'

export default configureStore({
  reducer: {
    counter: counterReducer
  }
})
```

用 Redux Toolkit 中的 configureStore 创建 Redux store。 注意，configureStore 需要传入 reducer 参数。  
应用会有很多特性，每个特性可能会对应各自的 reducer 函数。当调用 configureStore 时，会一股脑儿地把这些所有不同的 reducer 放到一个对象中({reducer:{key1: reducer1, key2: reducer2}})，再把这个对象传给 configureStore。这些 reducer 对象对应的 key，也就是最终 state 值的 key。  
`features/counter/counterSlice.js` 文件中导出的默认函数 slice.reducer 函数，实现了 counter 的逻辑。可以在创建 store 的文件中引入 `counterReducer`(即slice.reducer) 函数。  
当把 {counter: counterReducer} 对象传给 configureStore 函数时，其返回的 Redux state 对象就会包含 state.counter。之后，当接收到 dispatch 传过来的 action 时，counterReducer 函数就可以决定 state.counter 是否需要更新以及如何更新。  
Redux 允许 store 自定义配置各种不同的插件("中间件middleware"和"增强器(enhancers)"。为了提供更好地开发者体验，configureStore 函数会自动向 store 配置中加入几个中间件。同时为了让 Redux DevTools Extension 能够侦测到 store 的内容，也做了一些额外的配置。

### Redux Slices
"切片slice"是指一个应用特性对应的 Redux reducer 逻辑和 action 的集合，一般来说这些逻辑和 action 是定义在同一文件中的。切片这个名字源自将根 Redux state 对象拆分成 state 的多个"切片"。  
例如，博客应用中，store 配置可能会是这样的:

```javascript
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
```

这样，state.users、state.posts 和 state.comments 都是 Redux state 的独立的切片。

> ### 详细解释: reducer 和 state 结构
> Redux store 在创建时，得传入一个"根 reducer"函数。那么，如果有多个不同的 reducer 函数，该拿这个唯一的根 reducer 怎么办？如何定义 Redux store 的 state 内容呢？  
> 如果手动逐个调用所有的切片 reducer，就像下面这样:

```javascript
function rootReducer(state = {}, action) {
  return {
    users: usersReducer(state.users, action),
    posts: postsReducer(state.posts, action),
    comments: commentsReducer(state.comments, action)
  }
}
```

> 这种做法每次切片 reducer 的调用都是独立的，每次都把 Redux state 中特定的切片传递给 reducer，每次调用的返回值都是一个新的独立的 Reduce state 对象{users: userState, posts: postsState, comments: commentsState}。  
> Redux 有个名为 combineReducers 的函数，接受包含多个切片 reducer 的对象作为参数，当接收到 dispatch action 时，返回一个函数调用每一个切片 reducer。combineReducers 会把从每个切片 reducer 得到的结果全部合并到一个 state 对象作为最终结果。用 combineReducers 来改造前面的示例:

```javascript
const rootReducer = combineReducers({
  users: usersReducer,
  posts: postsReducer,
  comments: commentsReducer
})
```

> 把这个合并后的对象作为根 reducer 传递给 configureStore:

```javascript
const store = configureStore({
  reducer: rootReducer
})
```

### 创建切片 reducer 和 action
下面详细研究 `features/counter/counterSlice.js`

```javascript
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
      // Redux Toolkit 允许在 reducer 中编写"可变"逻辑。但由于使用了 immer 库，所以其实
      // 并没有真的去改变 state。代码会把检测到的更新动作写到一个"草稿 state"中，然后基于更改
      // 构造一个全新的不可变 state
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
```

点击 [counter](https://codesandbox.io/s/github/reduxjs/redux-essentials-counter-example/tree/master/?from-embed) 界面中的不同按钮会 dispatch 三种不同的 action 类型:  

- {type: "counter/increment"}`
- {type: "counter/decrement"}`
- {type: "counter/incrementByAmount"}`

action 是一个带有 type 字段的普通对象，type 字段的值是字符串。通常会用 action creator 函数完成 action 的构造工作。那该在何处定义这些 action 对象、type 字段和 action creator 函数呢？  
Redux Toolkit 有个名为 createSlice 的函数，可以自动完成诸如生成 action type 字符串、action creator 函数和 action 对象之类的工作。只要函数调用传入的对象中定义 slice name、编写 reducer 函数即可，这个函数会自动生成相关的 action。其中 slice name 和 reducer 函数的 key 值将作为 action type 的第一、二部分。例如，name 为 "counter" + key 为 "increment" 的 reducer 函数，自动生成的 action type 为 {type: "counter/increment"}。  
除了 name 字段，createSlice 还需要为 reducer 提供一个 initialState 值。  
上面的代码共有三个 reducer 函数，对应着三个不同的 action type。  
createSlice 函数根据我们定义的 reducer 函数，自动生成与其同名的 action creator 函数。可以通过下面的代码来验证:	

```javascript
console.log(counterSlice.actions.increment())
// {type: "counter/increment"}
```

同时还自动生成了 slice.reducer 函数，这个函数知道如何根据 action type 调用相应的处理函数(reducer)

```javascript
const newState = counterSlice.reducer(
  { value: 10 },
  counterSlice.actions.increment()
)
console.log(newState)
// {value: 11}
```

### reducer 规则
之前提到过，reducer 必须遵循一些特定的规则:

- 只根据取得的 state 和 action 参数计算新的状态，而与其他因素无关(即必须是纯函数)  
- 不得直接修改现有状态。即只能做不可变更新，需生成与现有状态相等的新状态对象，然后根据业务逻辑修改新状态对象
- 不能在函数内执行异步逻辑、计算随机值，或其他会造成"副作用"的操作

> 副作用  
> 如果一个函数修改了自己范围之外的资源，那就叫做有副作用，反之，就是没有副作用。


订立这样的规则有如下原因:

- Redux 的目标之一就是让代码的执行结果更加明确。如果一个函数的输出仅取决于其输入参数。那代码的运作就更加简单易懂，而且方便测试。
- 另外，如果函数依赖于外部变量，或是随机值，那函数的结果就无法预测。
- 如果函数修改了函数参数在内的其他值，这可能导致应用无法按预期运行。这可能是造成 bug 的根源，可能会发生"我更新了 state，但 UI 却没有跟着更新"之类的故事。
- reducer 不遵从这些规则，可能会破坏部分 Redux DevTools 的兼容性

"不可变更新"这条规则尤其重要，后面会继续探讨。

### reducer 和不可变更新
之前我们探讨过"可变"(直接修改现有对象/数组值)和"不可变性"(把对象当作不能改变的东西，不去直接改变对象值，能改也不改)

> 警告
> 在 Redux 中，reducer 永远也不允许直接去修改原始/当前 state 值!

```javascript
// ❌ Illegal - don't do this in a normal reducer!
state.value = 123
```

禁止在 Redux 直接修改 state 的原因如下:

- 会造成 bug，比如 UI 无法根据最新的状态值正确更新
- state 更新的原因和方式难以理解
- 难以编写测试
- 破坏"time-travel debugging"的能力
- 与 Redux 的基本思想和使用模式背道而驰

在不改变原始值的前提下，如何返回更新过的状态呢？

> Tip
> reducer 只有新建一份原始值的副本，然后修改新的副本后，返回副本

```javascript
// ✅ This is safe, because we made a copy
return {
  ...state,
  value: 123
}
```

前面使用展开操作符(...)来解决新建数组/对象副本的问题，但每次都这么做貌似有点low。  
手写不可变更新逻辑很繁琐，事实上，在 reducer 中直接修改 state 是 Redux 用户最常见的的错误。  
而 Redux Toolkit 的 createSlice 函数提供了不可变更新的简化方案。  
createSlice 里面使用了一个名为 Immer 的库。Immer 用到了一个名为 Proxy 的特殊的JS 工具。Poxy 可以包裹指定数据，让代码来修改包裹的数据。但 Immer 会跟踪所有的修改，然后用记录的修改行为列表来得到一个安全的不可变更新值，就好像手工写完了所有不可变更新逻辑。  
也就是不必费劲周折写成这样:

```javascript
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
```

只要这样写就好:

	function reducerWithImmer(state, action) {
	  state.first.second[action.someId].fourth = action.someValue
	}

这样也更易读，但有一点必须记住:

> 警告
> 只能在 Redux Toolkit 的 createSlice 和 createReducer 中这样写"可变"逻辑，因为它们内置了 Immer。如果在没有内置 Immer 的 reducer 中这么写，就会直接改变 state，并造成 bug。


有了前面的说明，继续来看 counter 切片的代码

```javascript
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
```

上面代码中的前两个 reducer，由于不需要使用 action 的数据，因此没有在 reducer 函数中声明。  
在 reducer 函数 incrementByAmount 中，由于要使用 action 中的数据，所以在 reducer 定义中声明了 state 和 action 两个参数。此时可知，输入到文本框的值会被放到 action.payload 字段中，也就会放到 state.value 中。    

> 参考  
> [Immutable Update Patterns](https://redux.js.org/recipes/structuring-reducers/immutable-update-patterns)  
> [The Complete Guide to Immutability in React and Redux](https://daveceddia.com/react-redux-immutability-guide/)

### 使用 thunk 编写异步逻辑
到现在为止，所有的应用逻辑都是同步的。dispatch action、store 运行 reducer、计算新 state，直到 dispatch 函数结束。而 JavaScript 是支持异步代码的，从远端 API 获取数据的之类的需求，通常也是通过异步逻辑进行的。那 Redux 应用的异步逻辑放在什么位置呢？    
thunk 是一类特殊的 Redux 函数，其中可以容纳异步逻辑。编写 thunk 得用到两个函数:  

- 内部 thunk 函数，接收两个参数，一个是 dispatch 函数，另一个 getState 函数
- 外部构造函数，创建并返回 thunk 函数

下面的函数源自 couterSlice，给出了 thunk action creator 的示例:

```javascript
{/*features/counter/counterSlice.js*/}

// The function below is called a thunk and allows us to perform async logic.
// It can be dispatched like a regular action: `dispatch(incrementAsync(10))`.
// This will call the thunk with the `dispatch` function as the first argument.
// Async code can then be executed and other actions can be dispatched
// 下面的函数称为 thunk，我们可以在其中执行异步逻辑。
// 这个函数可以像普通的 action 一样被 dispatch: `dispatch(incrementAsync(10))`。
// 代码中把需要调用的 thunk 作为 `dispatch` 函数的第一个参数。
// 这样就可以用 dispatch 执行异步代码了，接着就可以继续 dispatch 其他 action，
// 而不必等待异步代码执行结束
export const incrementAsync = amount => dispatch => {
  setTimeout(() => {
    dispatch(incrementByAmount(amount))
  }, 1000)
}
```

之后，就可以像使用普通 Redux action 一样使用这个异步 action 了:

```javascript
store.dispatch(incrementAsync(5))
```

注意，使用 thunk 需要在 Redux store 创建时传入 redux-thunk 中间件(一种 Redux 插件)。还好，Redux Toolkit 的 configureStore 函数会自动完成对 redux-thunk 中间件的相关设置，因此在代码中直接使用就好。  
使用 AJAX 调用从服务器获取数据时，可以把相关的 AJAX 调用放到 thunk 中。下面这个示例把函数的定义展开，虽然下面的例子比上面的长了不少，但可以更好地搞清楚定义过程:  

```javascript
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
```

在[第五章: 异步逻辑和数据获取](./ReduxEssentials05.md)中会了解 thunk 的使用。

> ### 详细解释: thunk 和异步逻辑
> 我们知道，reducer 中不能有任何形式的异步逻辑。但有时又必须需要调用异步逻辑。  
> 如果可以访问 Redux store，这样通过异步方式调用 store.dispatch():  

	```javascript
	const store = configureStore({ reducer: counterReducer })
	
	setTimeout(() => {
	  store.dispatch(increment())
	}, 250)
	```
	
	但在真实的Redux 应用中，是不允许把 store 引入到其他文件中的，尤其不能在 React 组建文件中引入 store。因为这样，会造成代码更难以测试和复用。
	
	另外，常常还需要为某些 store 写些异步逻辑，如果运行将 store 引入到其他文件中，最终根本没法分清引用的是哪个 store。
	
	Redux store 可以用"中间件"拓展，中间件是一种可以加入额外功能的 add-on 或插件。使用中间件最常见的原因就是，需要在代码中使用异步逻辑的同时，还能跟 store 继续通信。中间件的使用还可以改变 store，使其调用 `dispatch()` 时，不仅可以以普通的 action 对象为参数，还可以使用函数或是 Promise 作为参数。
	
	Redux Thunk 中间件通过修改 store 后，就可以将函数传给 `dispatch`。实际上，这个中间件真的很短，直接贴在下面:
	
	```javascript
	const thunkMiddleware = ({ dispatch, getState }) => next => action => {
	  if (typeof action === 'function') {
	    return action(dispatch, getState, extraArgument)
	  }
	  return next(action)
	}
	```
	
	中间件会检查传给 dispatch 的 "action" 是函数还是一个普通对象。如果是函数，则调用该函数，并返回结果。否则，认为 action 参数是一个 action 对象，就继续将其向前传递给 store。
	
	这为我们提供了一种编写任何同步或异步代码的方法，而同时仍然可以访问 `dispatch` 和 `getState`。

这个文件(counterSlice.js)中还有一个函数没有讲到，等到研究 `<Counter>` UI 组件时，再顺带说一句就行。

> 相关材料
>
> [the Redux Thunk docs](https://github.com/reduxjs/redux-thunk)
>
> [What the heck is a thunk?](https://daveceddia.com/what-is-a-thunk/)
>
>  [Redux FAQ entry on "why do we use middleware for async?"](https://redux.js.org/faq/actions#how-can-i-represent-side-effects-such-as-ajax-calls-why-do-we-need-things-like-action-creators-thunks-and-middleware-to-do-async-behavior)



### React Counter 组件

之前我们见过独立的 React `<Counter>`如何运作。其实 React+Redux 应用类似于 `<Counter>`组件，只有稍许不同。

下面来看下 `Counter.js`组件文件:

```javascript
{/* features/counter/Counter.js */}

import React, { useState } from 'react'
import { useSelector, useDispatch } from 'react-redux'
import {
  decrement,
  increment,
  incrementByAmount,
  incrementAsync,
  selectCount
} from './counterSlice'
import styles from './Counter.module.css'


export function Counter() {
  const count = useSelector(selectCount)
  const dispatch = useDispatch()
  const [incrementAmount, setIncrementAmount] = useState('2')
  return (
    <div>
      <div className={styles.row}>
        <button
          className={styles.button}
          aria-label="Increment value"
          onClick={() => dispatch(increment())}
        \>
          +
        </button>
        <span className={styles.value}>{count}</span>
        <button
          className={styles.button}
          aria-label="Decrement value"
          onClick={() => dispatch(decrement())}
        \>
          \-
        </button>
      </div>
      {/* omit additional rendering output here */}
    </div>
  )
}
```

就像之前的普通 React 实例一样，文件中包含一个函数组件，名为`Counter`，它在 useState hook 中存了些数据。

但看上去我们的组件好像没有把实际的当前 couter 值作为  state 来存储。代码中有个名为`count`的 变量，但这个变量并非源自`useState`  hook。

而 React 中包含几个内置 hook，诸如 `useState`和`useEffect`之类，其他库可以创建他们自己的[自定义 hook](https://reactjs.org/docs/hooks-custom.html) 使用 React hook 来构建自定义逻辑。

[React-Redux 库](https://react-redux.js.org/)就有一系列自定义的 [hook](https://react-redux.js.org/api/hooks) 以便 React 组件可以与 Redux store 交互。





Like with the earlier plain React example, we have a function component called `Counter`, that stores some data in a `useState` hook.

However, in our component, it doesn't look like we're storing the actual current counter value as state. There *is* a variable called `count`, but it's not coming from a `useState` hook.

While React includes several built-in hooks like `useState` and `useEffect`, other libraries can create their own [custom hooks](https://reactjs.org/docs/hooks-custom.html) that use React's hooks to build custom logic.

The [React-Redux library](https://react-redux.js.org/) has [a set of custom hooks that allow your React component to interact with a Redux store](https://react-redux.js.org/api/hooks).

#### 