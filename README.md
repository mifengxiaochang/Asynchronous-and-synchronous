# JS的异步和同步

在研究这个之前要搞懂几个小问题
1. JS是单线程语言
2. JS的执行机制
3. Promise对象
---
- JS为何是单线程语言

JS的初衷是服务于浏览器，而浏览器上要是多线程比如一个process需要edit 一个dom而同时另一个process又要进行delete操作，那么浏览器得疯掉~ 

这么安慰自己应该比较好理解为何要设计出单线程

## JS的运行机制是Event loop
运行和执行不同，对于不同环境下（Node、浏览器）JS的执行方式有些许差异，而运行一般指JavaScript解析引擎，是统一的。

Promise是什么就戳[这里吧](../Promise)

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
              //=>1,3,2

```
上面可以看出,setTimeout里的函数并没有立即执行,而是延迟了一段时间,满足一定条件后,才去执行的。

### setTimeout和setInterval

到这还得仔细瞅瞅这两个异步函数

上学还是小白汐的时候老师介绍setTimeout是隔多少毫秒触发回调函数一次，setInterval是隔多少毫秒触发一次回调函数

后来发现果然老师基本只教一半

其实严谨的说是这样的

XX豪秒后,setTimeout里的函数被会推入event queue,而event queue里的任务,只有在主线程空闲时才会执行。setInterval同理。

所以只有同时满足 (1)1秒后 (2)主线程空闲,才真的是会1秒后执行该函数

#### setInterval 缺点
由此可看出定时器指定的时间间隔，表示的是何时将定时器的代码添加到消息队列，而不是何时执行代码。

所以真正何时执行代码的时间是不能保证的，取决于何时被主线程的事件循环取到，并执行。

综上所述，setInterval有两个缺点：

1. 使用setInterval时，某些间隔会被跳过；
2. 可能多个定时器会连续执行；

---

这也就是一个实现异步的方式。

他主要是通过
**Event loop实现

- 首先判断JS是同步还是异步,同步就进入主进程,异步就进入event table
- 异步任务在event table中注册函数,当满足触发条件后,被推入event queue
- 同步任务进入主线程后一直执行,直到主线程空闲时,才会去event queue中查看是否有可执行的异步任务,如果有就推入主进程中，
- 直到event table为空

这里就能明白有一个大神们常用的，但之前的小白汐不理解的东西：setTimeout(function, 0)，为了将function里的任务异步执行。

0不是立即执行的意思，而是将任务推到消息队列的最后，再由主线程的事件循环去调用它执行。而且HTML5 中规定setTimeout 的最小时间不是0ms，而是4ms。

由此规律我们可以分析下刚才的栗子
```
console.log('1'); 是同步任务,放入主线程里
setTimeout() 是异步任务,被放入event table, 0秒之后被推入event queue里
console.log('3') 是同步任务,放到主线程里 
当 1,3在控制条被打印后,主线程去event queue里查看是否有可执行的函数,执行setTimeout里的函数cosnole.log('2');
```
找到了一个比较直接图片

![EventLoop](./images/EventLoop.png)

图片上对任务有更精细的定义：

**macro-task(宏任务)**：包括整体代码script，setTimeout，setInterval

**micro-task(微任务)**：Promise，process.nextTick，Object.observe，MutationObserver

有一个教科书级定义
```
一个事件循环(event loop)会有一个或多个任务队列(task queue)
task queue 就是 macrotask queue
每一个 event loop 都有一个 microtask queue
task queue == macrotask queue != microtask queue
一个任务 task 可以放入 macrotask queue 也可以放入 microtask queue 中
```

这里的微任务是指Promise和next的是回调函数~~~~

翻译一下：

- 执行一个宏任务，同步就直接执行,异步就进入event table中注册函数,当满足触发条件后,被推入【宏任务的event queue】
- 过程中如果遇到微任务,就将其放到[微任务的event queue]里
- 当前宏任务执行完成后,会查看[微任务的event queue],并将里面全部的微任务依次执行完
- 之后再执行【宏任务的event queue】

看到过一个简单的说法：

整体的js代码宏观先执行，同步代码执行完后有微观执行微观，没有就执行下一个宏观，如此往复循环至结束

举个栗子：
```JS
console.log('start1');
setTimeout(
   ()=>{console.log('step2-setTimeout')} 
    ,1000);
new Promise((resolve)=>{
    console.log('step3');
    resolve();
    }).then(
        ()=>{console.log('then4')}
        );
console.log('end5');

```
首先执行script下的宏任务,遇到setTimeout,将其1秒后放到宏任务的event queue里
 
遇到 new Promise直接执行,打印"step3"
 
遇到then方法,是微任务,将其放到微任务的event queue
 
打印 "end5"
 
本轮宏任务执行完毕,查看本轮的微任务,发现有一个then方法里的函数, 打印"then4"
 
到此,本轮的event loop 全部完成。
开始下一轮循环，由于宏任务的event queue中还有任务
所以执行setTimeout里的函数,打印"step2-setTimeout"

所以输出：

start1
step3
end5
then4
step2-setTimeout

在做测试的时候有几个错误示范举个栗子：
```js
console.log('start1');

//当走到这个宏任务要传到event table的时候注册的就是一个可以执行的语句所以直接打印step2-setTimeout
setTimeout(console.log('step2-setTimeout'),10000);


new Promise(()=>{console.log('step3')}).then(
        ()=>{console.log('then4')}//不执行
        );
console.log('end5');
```
**then 方法的作用是为 Promise 实例添加状态改变时的回调函数**

四不四感觉明白了，从网上看到一个面试题测一下：

```JS
console.log('start');

const interval = setInterval(() => {  
  console.log('setInterval')
}, 0);

setTimeout(() => {  
  console.log('setTimeout 1')
  Promise.resolve()
      .then(() => {
        console.log('promise 3')
      })
      .then(() => {
        console.log('promise 4')
      })
      .then(() => {
        setTimeout(() => {
          console.log('setTimeout 2')
          Promise.resolve()
              .then(() => {
                console.log('promise 5')
              })
              .then(() => {
                console.log('promise 6')
              })
              .then(() => {
                clearInterval(interval)
              })
        }, 0)
      })
}, 0);

Promise.resolve()
    .then(() => {  
        console.log('promise 1')
    })
    .then(() => {
        console.log('promise 2')
    });

```
❤大我要飞了~~







