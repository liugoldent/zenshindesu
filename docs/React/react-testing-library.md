---
id: react-testing-library
title: Testing Library 常用語法
sidebar_label: Testing Library 常用語法
slug: /react-testing-library
---
## 生命週期執行
### `beforeAll`
所在區域內會第一個執行
### `beforeEach`
每一次的測試前會先執行
### `afterAll`
所在區域內會最後一個執行
### `afterEach`
每一次測試後會馬上執行

## Basic
### `it or test`
用於描述測試本身，其包含兩個引數，第一個是該測試的敘述，第二個是執行測試的函式。
### `render`
用於渲染component使用
### `expect`
表示測試需要通過的條件，他將接收到的引數，與matcher比較
### `matcher`
預想要的結果
### `cleanup`
目的為每個測試完成後清除所有內容，避免記憶體洩漏
```javascript
import { render, cleanup } from "@testing-library/react";
// 定義每個測試完後，清除內容
afterEach(cleanup)
it("我是一個描述", () => {
                         // render函數，負責render component
  const { asFragment } = render(<App />);
  // toMatchSnapshot 為匹配器
  // 預期asFragment(<App />)結果，與toMatchSnapshot匹配
  expect(asFragment(<App />)).toMatchSnapshot();
});
```

## 測試使用的Component
```jsx
// 定義Component
const FirstTest = props =>{
  const [counter, setCounter] = useState(0)

  return (
    <div>
      {/*這邊定義testId，以利測試時取得DOM*/}
      <h1 data-testid="h1counter">{counter}</h1>
      <button data-testid="buttonAdd" onClick={()=>setCounter(counter+1)}>+</button>
      <button
        disabled
        date-testid="buttonDown"
        onClick={()=> setCounter(counter-1)}
        >-</button>
    </div>
  )
}
```

## 測試DOM
我們可以利用一些getTestId or ...來取得DOM位置
### `getByTestId`
直接用Id取得DOM元素
### `getByText`
去取得DOM呈現的內容，然後toBe來預測為何
### `container`
等於取得整包DOM物件，甚至是能夠直接對它使用`querySelector`來取得節點。
```jsx
import React, {useState} from 'react'
import { render, cleanup} from "@testing-library/react";
import '@testing-library/jest-dom'
import FirstTest from './FirstTest'

it('test counter', ()=>{
  // 用解構的方式取得TestId這個function
  const { getByTestId } = render(<FirstTest />)

  // 利用getByTestId取得節點後                    // 然後讓其toBe為何
  expect(getByTestId('h1counter').textContent).toBe("0")

  // 測試這個button是否為disabled（注意這個toBeDisabled是來自於jest/dom）
  expect(getByTestId('buttonDown')).toBeDisabled()
})
```

## 測試事件
### `fireEvent`
來觸發DOM事件，包含`onClick`、`onChange`等  
### [fireEvent API資料](https://github.com/testing-library/dom-testing-library/blob/master/src/event-map.js)
```js
import React, {useState} from 'react'
import { render, cleanup, fireEvent} from "@testing-library/react";
import '@testing-library/jest-dom'

afterEach(cleanup)
it('increment counter', ()=>{
  // 一樣先render出component
  const { getByTestId } = render(<FirstTest />)
  // 在這邊我們用fireEvent來觸發button
  fireEvent.click(getByTestId('buttonAdd'))
  // 然後我們預期這個testId的結果為1
  expect(getByTestId('h1counter').textContent).toBe("1")
})

it('decrease counter', ()=>{
  const { getByTestId } = render(<FirstTest />)
  fireEvent.click(getByTestId('buttonDown'))

  // 注意這邊，因為是連續的測試，所以這邊的數字是上面的結果"1" ，減掉 "1" 後剩下"0"
  expect(getByTestId('h1counter').textContent).toBe("0")
})

```

## 測試非同步事件
### `waitFor`
注意的事：
1. 你必須先`import '@babel/polyfill'`，因為一般語法不支援`async` `await`
2. 
```javascript
import '@babel/polyfill';
import React, {useState, useEffect} from 'react'
import { render, cleanup, fireEvent, waitFor} from "@testing-library/react";
import '@testing-library/jest-dom'

const FirstTest = () =>{
  const [counter, setCounter] = useState(0)
  // 我們在這邊假設一個非同步事件
  const DelayFunc = function(){
    setTimeout(()=>{
      setCounter(counter+10)
    }, 5000)
  }
  return (
    <div>
      <h1 data-testid="h1counter">{counter}</h1>
      <button data-testid="buttonAdd" onClick={DelayFunc}>+</button>
      <button
        disabled
        data-testid="buttonDown"
        onClick={()=> setCounter(counter-1)}
        >-</button>
    </div>
  )
}
afterEach(cleanup)
it('delay counter', async() =>{
  const { getByTestId, getByText } = render(<FirstTest />)
  fireEvent.click(getByTestId('buttonAdd'))
  // 上方都一樣，render後，click button

  // 下方使用waitFor來等待結果
  await waitFor(() => {
    expect(getByTestId('h1counter').textContent).toBe('10')
  }, {
    // 注意這邊是因為testing/library預設是1000ms，而在這個範例中我們使用了5000ms
    timeout:5000
  });
})
```

## 測試 Redux
0. 建立一個reducer.js的檔案
```javascript
const INCREMENT = 'INCREMENT';

const initState = {
  count: 0,
};

const counterReducer = (state = initState, action) => {
  switch (action.type) {
    case INCREMENT:
      return {
        count: state.count + 1,
      };
    default:
      return state;
  }
};

export default counterReducer
```
1. 首先我們需要把組件資料都改成`redux`版本
```jsx
import React from 'react'
import { Provider, useSelector, useDispatch } from 'react-redux'
import { createStore} from "redux";
import counterReducer from "../src/store/counterReducer";
import { render, cleanup, fireEvent } from "@testing-library/react";
import '@testing-library/jest-dom'
import '@testing-library/jest-dom/extend-expect';

const ReduxTest = () =>{
  // 使用useDispatch 取出 dispatch 事件
  const dispatch = useDispatch()
  const increment = () => dispatch({type: "INCREMENT"})
  const decrement = () => dispatch({type: "DECREMENT"})

  // 使用useSelector 得到 redux 資料
  const counter = useSelector(state => state.count)
  return (
    <div>
      // 注意這邊的 counter 已經變成 redux 資料了
      <h1 data-testid="h1counter">{counter}</h1>
      <button data-testid="buttonAdd" onClick={increment}>+</button>
      <button
        disabled
        data-testid="buttonDown"
        onClick={decrement}
        >-</button>
    </div>
  )
}
```
2. 測試方法與之前相同
```jsx
it('delay counter', () =>{
  // 首先要先createStore
  const store = createStore(counterReducer)
  
  // 再來將store 放入 Provider 中
  const { getByTestId } = render(
    <Provider store={store}>
      <ReduxTest />
    </Provider>
  )

  // 一樣的事件規劃方式
  fireEvent.click(getByTestId('buttonAdd'))
  expect(getByTestId('h1counter').textContent).toBe('1')
})
```

## 測試Context
1. 首先跟redux很像的，我們先建立一個`Provider`
```jsx
const CounterContext = React.createContext()
const CounterProvider = ()=>{
  // 這裡跟一般component設定很像，不過我們就是把它提到Provider層
  const [ counter, setCounter ] = React.useState(0)
  const increment = () => setCounter(counter+1)
  const decrement = () => setCounter(counter-1)

  return(
    <CounterContext.Provider value={{counter, increment, decrement}}>
      <Counter />
    </CounterContext.Provider>
  )
}
```
2. 使用一般組件
```jsx
const Counter = () =>{
  // 這邊呼應到上面所傳的物件                     // 特別的是，我們在這邊使用useContext來取得資料
  const { counter , increment, decrement } = useContext(CounterContext)
  return (
    // 下方寫法相同
    <div>
      <h1 data-testid="h1counter">{counter}</h1>
      <button data-testid="buttonAdd" onClick={increment}>+</button>
      <button
        disabled
        data-testid="buttonDown"
        onClick={decrement}
        >-</button>
    </div>
  )
}
```
3. 另外我們傳出一個接口來render組件
```jsx
// 傳入一個組件進去，進行render
const renderWithContext = (component) =>{
  return {
    ...render(
      <CounterProvider value={CounterContext}>
        { component }
      </CounterProvider>
    )
  }
}
```
4. 測試函式
```jsx
afterEach(cleanup)
it('delay counter', () =>{
  // 這邊我們使用轉接器renderWithContext 來 render 組件
  const { getByTestId } = renderWithContext(<Counter />)

  // 下面相同
  fireEvent.click(getByTestId('buttonAdd'))
  expect(getByTestId('h1counter').textContent).toBe('1')
})
```
## 參考資料
[jest-dom](https://github.com/testing-library/jest-dom)
[再一次測試你的Component](https://ppt.cc/f2lssx)
