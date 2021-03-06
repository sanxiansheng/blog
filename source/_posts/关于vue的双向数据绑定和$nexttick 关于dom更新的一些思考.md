---
title: 关于vue的双向数据绑定的简单理解和$nexttick 关于dom更新的一些理解
tags: vue
---

# vue的双向数据绑定
__vue采用的方式属于数据劫持+发布订阅模式：__
通过Object.defineProperty()来劫持各个属性的setter，getter，在数据变动时发布消息给订阅者，触发相应的监听回调。
所以实现这个过程大概需要一下以下几点:

	1 创建一个数据监听器 监听数据对象包含的所有属性 如果有数据变动，那就去通知订阅者

	2 实现一个指令解析器 对每个元素节点的指令进行扫描，根据指令替换数据(相互对应)，以及绑定相应的更新函数

	3 实现一个数据监听器和指令解析器之间的桥梁，让解析器收到监听器的数据变动的通知，然后去更新视图

<img src='http://sanasan.top/images/mvvm.png' >


# 关于Vue this.$nextTick 和DOM的更新

官方文档说明:

__用法：在下次 DOM 更新循环结束之后执行延迟回调。在修改数据之后立即使用这个方法，获取更新后的 DOM__
__想法:从绑定的数据发生改变，到更新视图，这个过程是异步的？__

## 关于异步
异步运行机制:
	(1)所有同步任务都在主线程上执行,形成一个执行栈
	(2)主线程之外，还存在一个“任务队列”。只要异步任务有了运行结果，就在“任务队列”中放置一个事件。
	(3)一旦“执行栈”中的所有同步任务执行完毕，系统就会读取“任务队列”，看看 还有那些时间，那些对应的异步任务这时候哦就结束了等待状态，进入执行栈，开始执行。
	(4)主线程不断重复（3）
<img src='http://sanasan.top/images/js_event_loop.png' >
ajax异步的过程
<img src='http://sanasan.top/images/js_event_loop2.png' >

__vue的响应式并不是数据改变后，DOM立即变化 而是按照一点的策略进行DOM更新__ 例子:

	//改变数据
	vm.message = 'changed'

	//想要立即使用更新后的DOM。这样不行，因为设置message后DOM还没有更新
	console.log(vm.$el.textContent) // 并不会得到'changed'

	//这样可以，nextTick里面的代码会在DOM更新后执行
	Vue.nextTick(function(){
    console.log(vm.$el.textContent) //可以得到'changed'
	})

简单总结下这块的过程：


__同步代码执行 -> 查找异步队列，推入执行栈，执行Vue.nextTick[事件循环1] ->查找异步队列，推入执行栈，执行Vue.nextTick[事件循环2]... 总之，异步是单独的一个tick，不会和同步在一个 tick 里发生，也是 DOM 不会马上改变的原因。__
