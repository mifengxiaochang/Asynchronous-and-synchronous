# JS的异步和同步
在研究这个之前要搞懂几个小问题
1. JS是单线程语言
2. JS的执行机制
3. Promise对象

- JS为何是单线程语言

JS的初衷是服务于浏览器，而浏览器上要是多线程比如一个process需要edit 一个dom而同时另一个process又要进行delete操作，那么浏览器得疯掉~ 

这么安慰自己应该比较好理解为何要设计出单线程

## JS的运行机制是Event loop
运行和执行不同，对于不同环境下（Node、浏览器）JS的执行方式有些许差异，而运行一般指JavaScript解析引擎，是统一的。

Promise是什么就戳[这里吧](./)

## 线程和进程

其实写前端而言本身对这个概念并不太明确，看了很多资料就用阮一峰大神的栗子说明吧。
```
有一个大型工厂
工厂里有若干车间，每次只能有一个车间在作业
每个车间里有若干房间，有若干工人在流水线作业

那么：

一个工厂对应的就是计算机的一个CPU，平时讲的多核就代表多个工厂
每个工厂里的车间，就是进程，意味着同一时刻一个CPU只运行一个进程，其余进程在怠工
这个运行的车间（进程）里的工人，就是线程，可以有多个工人（线程）协同完成一个任务
车间（进程）里的房间，代表内存。

再深入点：

车间（进程）里工人可以随意在多个房间（内存）之间走动，意味着一个进程里，多个线程可以共享内存
部分房间（内存）有限，只允许一个工人（线程）使用，此时其他工人（线程）要等待
房间里有工人进去后上锁，其他工人需要等房间（内存）里的工人（线程）开锁出来后，才能才进去，这就是互斥锁（Mutual exclusion，缩写 Mutex）
有些房间只能容纳部分的人，意味着部分内存只能给有限的线程

再再深入:

如果同时有多个车间作业，就是多进程
如果一个车间里有多个工人协同作业，就是多线程
当然不同车间之间的工人也可以有相互协作，就需要协调机制
```

## 同步(synchronous)

假如一个函数返回时，调用者就能够得到预期结果(即拿到了预期的返回值或者看到了预期的效果)。

栗子：
```JS
console.log('1');
console.log('2');
console.log('3');
                  \\=>1,2,3
```
## 异步(asynchronous)
假如一个函数返回时，调用者不能得到预期结果，需要通过一定手段才能获得。

栗子：
```JS
coonsle.log('1');
setTimeout(function() {
   console.log('2');
}, 1000);
console.log('3');
              \\=>1,3,2

```
上面可以看出,setTimeout里的函数并没有立即执行,而是延迟了一段时间,满足一定条件后,才去执行的。

JS是单线程那实现异步？
-》通过Event loop：

- 首先判断JS是同步还是异步,同步就进入主进程,异步就进入event table
- 异步任务在event table中注册函数,当满足触发条件后,被推入event queue
- 同步任务进入主线程后一直执行,直到主线程空闲时,才会去event queue中查看是否有可执行的异步任务,如果有就推入主进程中

由此规律我们可以分析下上面的栗子
```
console.log('1'); 是同步任务,放入主线程里
setTimeout() 是异步任务,被放入event table, 0秒之后被推入event queue里
console.log('3') 是同步任务,放到主线程里 
当 1,3在控制条被打印后,主线程去event queue里查看是否有可执行的函数,执行setTimeout里的函数cosnole.log('2');

```










