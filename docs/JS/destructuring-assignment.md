---
id: destructuring-assignment
title: Destructuring-assignment
sidebar_label: Destructuring-assignment
slug: /destructuring-assignment
---
```javascript
function test(par1, {par2, par3}){
  console.log(par1)
  console.log(par2)
  console.log(par3)
}
test(1,{
  par2: 2,
  par3: 3
})
```
