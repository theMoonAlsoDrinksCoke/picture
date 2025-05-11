---
title: 'Two Forms of Pre-rendering'
date: '2020-01-01'
---

## 数据更新驱动
![图片](https://gcore.jsdelivr.net/gh/theMoonAlsoDrinksCoke/picture@main/static/20250506212713241.png)
### useState

```react
// 函数组件通过 useState 可以让组件重新渲染，更新视图。
// state，目的提供给 UI ，作为渲染视图的数据源。
// dispatchAction 改变 state 的函数，可以理解为推动函数组件渲染的渲染函数。
// initData 有两种情况，第一种情况是非函数，将作为 state 初始化的值。第二种情况是函数，函数的返回值作为 useState 初始化的值。

const [ ①state , ②dispatch ] = useState(③initData)
更改后需要重新useEffect后数据才会更新
```

### useReducer

```react
// 更新之后的 state 值。
// 派发更新的 dispatchAction 函数, 本质上和 useState 的 dispatchAction 是一样的。
// 一个函数 reducer ，我们可以认为它就是一个 redux 中的 reducer , reducer的参数就是常规reducer里面的state和action, 返回改变后的state, 这里有一个需要注意的点就是：如果返回的 state 和之前的 state ，内存指向相同，那么组件将不会更新。

const [ ①state , ②dispatch ] = useReducer(③reducer)


const DemoUseReducer = ()=>{
    /* number为更新后的state值,  dispatchNumbner 为当前的派发函数 */
   const [ number , dispatchNumbner ] = useReducer((state,action)=>{
       const { payload , name  } = action
       /* return的值为新的state */
       switch(name){
           case 'add':
               return state + 1
           case 'sub':
               return state - 1
           case 'reset':
             return payload
       }
       return state
   },0)
   return <div>
      当前值：{ number }
      { /* 派发更新 */ }
      <button onClick={()=>dispatchNumbner({ name:'add' })} >增加</button>
      <button onClick={()=>dispatchNumbner({ name:'sub' })} >减少</button>
      <button onClick={()=>dispatchNumbner({ name:'reset' ,payload:666 })} >赋值</button>
      { /* 把dispatch 和 state 传递给子组件  */ }
      <MyChildren  dispatch={ dispatchNumbner } State={{ number }} />
   </div>
}
```

### useSyncExternalStore

```react
// subscribe 为订阅函数，当数据改变的时候，会触发 subscribe，在 useSyncExternalStore 会通过带有记忆性的 getSnapshot 来判别数据是否发生变化，如果发生变化，那么会强制更新数据。
// getSnapshot 可以理解成一个带有记忆功能的选择器。当 store 变化的时候，会通过 getSnapshot 生成新的状态值，这个状态值可提供给组件作为数据源使用，getSnapshot 可以检查订阅的值是否改变，改变的话那么会触发更新。
// getServerSnapshot 用于 hydration 模式下的 getSnapshot。

useSyncExternalStore( subscribe, getSnapshot, getServerSnapshot )

// 例
import { combineReducers , createStore  } from 'redux'

/* number Reducer */
function numberReducer(state=1,action){
    switch (action.type){
      case 'ADD':
        return state + 1
      case 'DEL':
        return state - 1
      default:
        return state
    }
}

/* 注册reducer */
const rootReducer = combineReducers({ number:numberReducer  })
/* 创建 store */
const store = createStore(rootReducer,{ number:1  })

function Index(){
    /* 订阅外部数据源 */
    const state = useSyncExternalStore(store.subscribe,() => store.getState().number)
    console.log(state)
    return <div>
        {state}
        <button onClick={() => store.dispatch({ type:'ADD' })} >点击</button>
    </div>
}
```

### useTransition (v18)

```react
// useTransition 执行返回一个数组。数组有两个状态值：

//     第一个是，当处于过渡状态的标志——isPending。

//     第二个是一个方法，可以理解为上述的 startTransition。可以把里面的更新任务变成过渡任务。

import { useTransition } from 'react'
/* 使用 */
const  [ isPending , startTransition ] = useTransition ()

// 例子

 const App = () => {
      const { useTransition, useState } = React
      const [active, setActive] = useState('tab1') //需要立即响应的任务，立即更新任务
      const [renderData, setRenderData] = useState(tab[active]) //不需要立即响应的任务，过渡任务
      const [isPending, startTransition] = useTransition()
      const handleChangeTab = (activeItem) => {
        setActive(activeItem) // 立即更新
        startTransition(() => { // startTransition 里面的任务优先级低
          setRenderData(tab[activeItem])
        })
      }
      return (
        <div>1
          <div className='tab' >
            {Object.keys(tab).map((item) => <span className={active === item && 'active'} onClick={() => handleChangeTab(item)} >{item}</span>)}
          </div>
          <ul className='content' >
            {isPending && <div> loading... </div>}
            {renderData.map(item => <li key={item} >{item}</li>)}
          </ul>
        </div>
      )
    }

    ReactDOM.render(<App></App>, document.getElementById('app'))

```

### useDeferredValue (v18)

```react
// React 18 提供了 useDeferredValue 可以让状态滞后派生。useDeferredValue 的实现效果也类似于 transtion，当迫切的任务执行后，再得到新的状态，而这个新的状态就称之为 DeferredValue。
// useDeferredValue 基础介绍：
// useDeferredValue 和上述 useTransition 本质上有什么异同呢？
// 相同点： useDeferredValue 本质上和内部实现与 useTransition 一样都是标记成了过渡更新任务。
// 不同点： useTransition 是把 startTransition 内部的更新任务变成了过渡任务transtion,而 useDeferredValue 是把原值通过过渡任务得到新的值，这个值作为延时状态。一个是处理一段逻辑，另一个是生产一个新的状态。
// useDeferredValue 接受一个参数 value ，一般为可变的 state , 返回一个延时状态 deferrredValue。

const deferrredValue = React.useDeferredValue(value)

// 例子
/* 模拟数据 */
    const mockList1 = new Array(10000).fill('tab1').map((item, index) => item + '--' + index)
    const mockList2 = new Array(10000).fill('tab2').map((item, index) => item + '--' + index)
    const mockList3 = new Array(10000).fill('tab3').map((item, index) => item + '--' + index)

    const tab = {
      tab1: mockList1,
      tab2: mockList2,
      tab3: mockList3
    }
    const App = () => {
      const [active, setActive] = React.useState('tab1') //需要立即响应的任务，立即更新任务
      const deferActive = React.useDeferredValue(active) // 把状态延时更新，类似于过渡任务
      const handleChangeTab = (activeItem) => {
        setActive(activeItem) // 立即更新
      }
      const renderData = tab[deferActive] // 使用滞后状态
      return (
        <div>1
          <div className='tab' >
            {Object.keys(tab).map((item) => <span className={active === item && 'active'} onClick={() => handleChangeTab(item)} >{item}</span>)}
          </div>
          <ul className='content' >
            {renderData.map(item => <li key={item} >{item}</li>)}
          </ul>
        </div>
      )
    }

    ReactDOM.render(<App></App>, document.getElementById('app'))
// 当切换 tab 的时候，产生了两个优先级任务，第一个任务是 setActive 控制 tab active 状态的改变，第二个任务为 setRenderData 控制渲染的长列表数据 （在现实场景下长列表可能是一些数据量大的可视化图表）。
```

## hooks 副作用函数

### useEffect

```react
// useEffect是异步执行，类似于setTimeout放入任务队列，，等到主线程任务完成，DOM 更新，js 执行完成，视图绘制完毕，才执行。所以 useEffect 回调函数不会阻塞浏览器绘制视图
// 相当于函数组件的生命周期，第二个参数不传时，每次render都会执行callback。
//为空数组时仅在页面挂载和卸载是执行，有值时，值改变则触发

useEffect(()=>{},[])
```

### useLayoutEffect

```react
// 类似于useEffect，但是是同步执行，
// useLayoutEffect 在 DOM 更新之后，浏览器绘制之前执行
// 可以方便修改 DOM，获取 DOM 信息，这样浏览器只会绘制一次，如果修改 DOM 布局放在 useEffect ，那 useEffect 执行是在浏览器绘制视图之后，接下来又改 DOM ，就可能会导致浏览器再次回流和重绘。而且由于两次绘制，视图上可能会造成闪现突兀的效果。

useLayoutEffect(()=>{},[])
```

### useInsertionEffect (v18 新增)

```react
// 类似于useEffect，但是是同步执行，
// useInsertionEffect 执行 -> useLayoutEffect 执行 -> useEffect 执行
// useInsertionEffect 的执行的时候，DOM 还没有更新
// 主要用于动态的引入js,css。

useInsertionEffect(()=>{},[])
```

### useContext

```react
// 用于接收上层组件传的值
// 和props类似，但是props只能接收父组件的值
// useContext可以接收所有所有包含关系的上层的值。例如：父->子->孙，孙可以直接接收父的值

// 最上层组件需要被 AppContext.Provider 包裹 并在上面赋值、
// 上层组件
const AppContext = React.createContext({});
<AppContext.Provider value={{userName:'hello'}}>
      <div>
        <下层组件>
          <下层组件></下层组件>
        </下层>
        <下层组件></下层组件>
      </div>
    </AppContext.Provider>

// 下层组件
const value = useContext(Context)
```

### useRef

```react
// useRef 获取dom。useRef只有一个参数current
// useRef 对象的 current 值改变不会导致重新渲染(重要)，useState改变值会重新render

const getDom = React.useRef(initState)
<div ref= {getDom}></div>

// 保存状态  当前组件不被销毁，状态就会一直存在
status= React.useRef(false)

// 修改状态
status.current = true

```

## 少见但是重要

### forwardRef

```react
// forwardRef高阶函数，接收一个函数组件做参数，函数组件可以到接收父组件传递的ref
// 可以通过forwardRef实现父子组件相互调用方法与useImperativeHandle配合最好

代码实例看useImperativeHandle

```

### useImperativeHandle

```react
// useImperativeHandle 需要结合forwardRef使用
// useImperativeHandle(ref,callback,[]) 三个形参
// 第一个参数是父组件传过来的ref
// 第二个参数是个处理函数，它的返回值就是我们要传给父组件的方法和属性；返回的是一个对象
// 第三个参数是依赖项，表示只有依赖项中的值发生改变，才会把最新的属性和方法传给父组件；如果没有依赖项，则表示只要子组件render都会把属性和方法传给父组件。

    const Son =React.forwardRef((props,ref) =>{
      const {useEffect,useImperativeHandle} = React
      useImperativeHandle(ref,()=>{
        return {
          css
        }
      },[])
      useEffect(()=>{
        console.log(ref,'fas');
      },[])
      const css = ()=>{
        console.log('子组件');
      }
      return(
        <div>
          <button ref={ref} onClick={()=>{ref.current.cs()}}>点我</button>
      </div>
      )
    })
    const App =()=> {
      const child = React.useRef();
      const {useEffect,useState} = React
      const [num ,setNum] = useState('测试')
      useEffect(()=>{
        console.log(child,'child');
        child.current.cs = cs
      },[])
      const cs = ()=>{
        setNum('父组件')
        console.log("测试");
      }
        return (
          <div>1
            <div onClick={()=>{child.current.css()}}>{num}</div>
            <Son ref={child}/>
          </div>
        )
      }

    ReactDOM.render(<App></App>, document.getElementById('app'))

```

## 状态的缓存与保持

### useMemo

```react
// create：第一个参数为一个函数，函数的返回值作为缓存值，如上 demo 中把 Children 对应的 element 对象，缓存起来。
// deps：第二个参数为一个数组，存放当前 useMemo 的依赖项，在函数组件下一次执行的时候，会对比 deps 依赖项里面的状态，是否有改变，如果有改变重新执行 create ，得到新的缓存值。
// acheSomething：返回值，执行 create 的返回值。如果 deps 中有依赖项改变，返回的重新执行 create 产生的值，否则取上一次缓存值。

const cacheSomething = useMemo(create,deps)

```

### useCallback

```react
// useMemo 和 useCallback 接收的参数都是一样，都是在其依赖项发生变化后才执行，都是返回缓存的值，区别在于 useMemo 返回的是函数运行的结果，useCallback 返回的是函数，这个回调函数是经过处理后的也就是说父组件传递一个函数给子组件的时候，由于是无状态组件每一次都会重新生成新的 props 函数，这样就使得每一次传递给子组件的函数都发生了变化，这时候就会触发子组件的更新，这些更新是没有必要的，此时我们就可以通过 usecallback 来处理此函数，然后作为 props 传递给子组件。

const cacheSomething = useCallback(()=>{},[])

```

## 自定义 hooks

```react
// 自定义 Hooks 是一个函数，约定函数名称必须以 use 开头，React 就是通过函数名称是否以 use 开头来判断是不是 Hooks

// Hooks 只能在函数组件中或其他自定义 Hooks 中使用，否则，会报错！

// 自定义 Hooks 用来提取组件的状态逻辑，根据不同功能可以有不同的参数和返回值（就像使用普通函数一样）

 // 自定义hooks
    const useFilter = (props,ref) => {
      const { useEffect } = React
      useEffect(() => {
        console.log(props,'props',ref);
      }, [])
      return 1
    }
    const App = () => {

      return (
        <div>
          {useFilter(1,2)}
        </div>
      )
    }

    ReactDOM.render(<App></App>, document.getElementById('app'))
```


