## 微任务、Promise

### 1、以下代码的执行顺序

```
setTimeout(() => console.log('A'), 0) // 1
setTimeout(() => console.log('B'), 1000) // 2
Promise.resolve() // 3
  .then(() => { // 4
    setTimeout(() => console.log('C'), 0) // 5
    setTimeout(() => console.log('D'), 1000) // 6
    console.log('E') // 7
    Promise.resolve() // 8
      .then(() => console.log('F')) // 9
  }) // 10
  .then(() => console.log('G')) // 11

setTimeout(() => console.log('H'), 0) // 12
setTimeout(() => console.log('I'), 1000) // 13
```

输出结果为  E、F、G、A、H、C、B、I、D

解析  
通过函数调用栈，消息队列，webApi队列，微任务队列来解释上述代码的执行结果。这里用注释中数字n表示所在行代码，F(n)代表所在行对应的匿名函数
+ 首先 1、2 中的两个延时任务交给webapi, F(1)立即进入消息队列
+ 3 立即执行，then之后的内容进入微任务队列
+ 12、13 两个延时任务交给webapi，F(12)立即进入消息队列
+ 主线程脚本执行结束，微任务队列函数4内容进入主线程
+ 5、6 两个延时任务交给webapi，F(5)立即进入消息队列
+ 7 立即执行，打印  E
+ 8 立即执行，F(9) 进入微任务队列
+ F(11) 进入微任务队列
+ 新一轮主线程执行完毕，微任务队列F(9)，F(11)进入主线程立即执行，打印F、G
+ 主线程微任务队列没有待执行任务，事件循环开始起作用，从消息队列首部取出任务 F(1)，F(12)，F(5)，打印A、H、C
+ 1秒后webApi将F(2)、F(13)、F(6)交给消息队列，随后一次进入主线程执行，打印B、I、D