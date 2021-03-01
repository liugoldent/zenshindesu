---
id: card
title: Card 切版
sidebar_label: Card 切版
slug: /card
---
## 準備工具
* Tailwind CSS
* React

## Code
```jsx
import React from 'react'

const Post = ()=>{
  return (
    <div>
    <div className="w-full lg:w-1/2 p-3">
      <div className="
      flex
      flex-col
      lg:flex-row
      rounded
      overflow-hidden
      h-auto
      lg:h-32
      border
      shadow
      shadow-lg">
        <img className="block h-auto w-full lg:w-48 flex-none bg-cover h-24"
             src="https://images.pexels.com/photos/1302883/pexels-photo-1302883.jpeg?auto=compress&cs=tinysrgb&h=750&w=1260" />
          <div
            className="bg-white
            rounded-b
            lg:rounded-b-none
            lg:rounded-r
            p-4
            flex
            flex-col
            justify-between
            leading-normal">
            <div className="text-black font-bold text-xl mb-2 leading-tight">Can coffee make you a bitter developer?
            </div>
            <p className="text-grey-darker text-base">Read more</p>
          </div>
      </div>
    </div>
    </div>
  )
}
```

## 解析
1. 最外層的 `w-full` 代表`width:100%` 的概念
    * 寬度用100%顯示，代表這取決於父級的寬度
2. 第二層 `div` 包含著內容
    * `h-auto`：代表由子內容支撐其高度
3. img 
    * 因為設定為`block` 所以可以設定高度，代表其也會跟著 height 做改變
4. 記得 `auto` 跟子元素有關，100% 跟父元素有關
5. 其他是一些 `flex` & 各元素編框的基本排版 

## Ref
1. [CSS | width、height中auto與100%與固定值有什麼不同](https://www.itread01.com/content/1549818624.html)
