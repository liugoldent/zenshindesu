---
id: emit-event
title: emit 事件
sidebar_label: emit 事件
slug: /emit-event 
---
## code
```jsx
const { useState } = React;

// 主要父組件 Component
function PageComponent() {
  const [count, setCount] = useState(0);
  const increment = () => {
    setCount(count + 1)
  }

  return (
    <div className="App">
      {/*在這邊綁定子組件認識的onClick，為父組件的increment*/}
      <ChildComponent onClick={increment} count={count} />         
      <h2>count {count}</h2>
      (count should be updated from child)
    </div>
  );
}

const ChildComponent = ({ onClick, count }) => {
  return (
    // 前面的onClick 代表 react 事件，後面的onClick代表從父組件傳進來的func
    <button onClick={onClick}>
       Click me {count}
    </button>
  )
};
```

## 解析
1. 在子組件
    * 左邊的`onClick` 為react事件
    * 右邊的`onClick` 為父組件的function
2. 按下子組件的onClick後，即可觸發父組件綁定的`increment`事件
