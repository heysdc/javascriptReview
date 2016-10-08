#js异步编程学习

##从线程讲起
- **js是单线程的**
  - 主线程：js引擎中负责解释和执行js代码的线程只有一个（一个页面一个），也可以叫主线程
  - 工作线程：如：处理ajax请求的线程，处理dom事件的线程，定时器线程，读写文件的线程，这些线程可能在js引擎内，也可能在引擎外（如浏览器中）

- **无形的异步**：异步即一段代码分两段执行，执行－等待结果－继续执行，如果按照正常的程序运行过程（同步），由于执行js代码的线程只有一个，整个js的执行就停滞了，在页面进行任何涉及js的操作都没反应，这肯定不行的，所以设计到异步操作（setTimeout）都是自动做了异步处理了，即回调：

  ```js
  setTimeout(() => {
    console.log('ah~')
  }, 2000)
  ```
  setTimeout(callback,  milliseconds)，这就是一个异步过程，js规定了它的写法，这种写法本身就是按照异步来的

- **异步原理**
  按照`执行－等待结果-继续执行`的过程分析上面的代码就是：
  - 执行setTimeout(callback , 2000)，告诉js我要执行一个2秒后运行的一个任务（不管这个任务是干嘛的），同步执行，瞬间完成
  - 等待2秒
  - 执行callback

  从线程上分析：

  - 主线程执行setTimeout(callback, 2000)，setTimeout内置了异步请求，使主线程向工作线程发起了一个异步请求，相应的工作线程接收请求并告知主线程收到请求，主线程可以继续执行后面的代码
  - 工作线程执行计时2秒的任务
  - 主线程收到通知后，调用回调函数，`callback()`
  所以说异步函数一般要接受一个回调函数，具有如下形式`A(...args, callback)`，发起函数A，回调函数callback，也有其它如分离的形式：

  ```js
  var xhr = new XMLHttpRequest();
  xhr.onreadystatechange = xxx; // 添加回调函数
  xhr.open('GET', url);
  xhr.send(); // 发起函数
  ```
- **消息队列和事件循环**
  **消息队列**:一个存放消息的队列
  **事件循环**:主线程不断从消息队列中重复取消息，执行消息，执行了完了之后再取下一个消息
  异步过程的回调函数，一定不在当前这一轮事件循环中执行。
  **事件**：消息队列中每条消息都对应着一个事件，事件机制实际上就是异步过程的通知机制，工作线程执行完通知主线程就形成了一个`事件`
  工作线程是生产者，主线程是唯一的消费者，工作线程执行异步任务，完成后把对应的回调函数封装成一条消息放到消息队列中，主线程不断从消息队列中取消息执行。

##Promise
- **promise基本用法**: 对异步过程的一种封装，特点：不受外界影响以及一旦状态更改无法改变，特点不重要，关键理解如何封装异步。
  如下代码，上面的代码和下面代码的区别为，下面代码把回调函数定义在了then方法的第一个参数
  promise有三种装pending,resolved,rejected从正在进行分别变为完成失败，执行了resolve就完成了前者，执行reject就完成了后者，状态一旦更改无法改变
  promise的串联，即传给resolve的参数为一个promiseB,如果这个promiseB的resoleve执行了，则将其参数传过来，如果没有，就等着，必须promiseB状态变了后面才能执行

  ```js

  setTimeout(() => {
    console.log('a')
  }, 2000)

  var timeout = (time) => {
    return new Promise((resolve) => {
      setTimeout(resolve, time)
    })
  }
  timeout(2000).then(() => {
    console.log('a')
  })


  // 串联
  var a = (time) => {
    var tmp = new Promise((resolve) => {
      setTimeout(resolve, 5000, 'sss')
    })
    return new Promise((resolve) => {
      setTimeout(resolve, time, tmp)
    })
  }
  a(1000).then((value) => {
    console.log('sb', value) // sss
  })

  // 串联2
  var a = () => {
    return new Promise((resolve) => {
      setTimeout(resolve, 2000)
    })
  }
  a()
  .then(() => { // then返回的是一个promise，所以可以串联
    return 'sb'
  })
  .then((val) => {
    console.log(val) // 2s后输出sb
  })
  
  ```
- **catch**:.catch(callback) ＝ .then(null, callback)的别名，专门用于捕获错误
  Promise错误具有冒泡性质，会一直传递直到被捕获为止
  nodejs有一个unhandledRejection事件，专门监听未捕获的Rejected错误

- **Promise.all()**:用于将多个Promise实例包装成一个新的Promise实例
  Promise.all([promise1, promise2, ...]):参数可以不是数组，但必须有iterator接口，里面元素可以不是promise，那就调用promise.resolve()对其进行promise化
  结果：都变为resolved，则返回一个数组传给回调函数，有一个为rejected，则返回第一个为rejected的错误

  ```js
  for (var i = 0; i <= 10; i++) { // 有的时候会有这种奇怪的需求
    Api //调用接口，不知道哪个先完成
  }

  var promise1 = new Promise((resolve, reject) => {
    var client = new XMLHttpRequest()
    ...
    client.onreadystatechange = handler
    client.send()

    function handler() {
      if (this.status === 200) {
        resolve(this.responce)
      } else {
        reject(new Error(this.statusText))
      }
    }
  })

  var promise2 = ...
  var promises = Promise.all([promise1, promise2...]) // 同时调用多个接口（进行多个异步操作），等都完成了再处理
  .then((promses) => {})
  .catch((err) => {})
  ```

- **Promise.race()**:与Promise.all()传的参数一样，区别：哪个promise先返回结果了就返回哪个
  常用前端控制调用时间，几秒内调用借口没反应就不掉了，报个错，可以一顶成都弥补promise无法取消的缺点：

  ```js
  var promise1 = new Promise((resolve, reject) => {
    var client = new XMLHttpRequest()
    ...
    client.onreadystatechange = handler
    client.send()

    function handler() {
      if (this.status === 200) {
        resolve(this.responce)
      } else {
        reject(new Error(this.statusText))
      }
    }
  })

  var promise2 = new Promise((resolve, reject) => {
    setTimeout(() => {
      reject(new Error(''链接超时，请检查网络''))
    }, 10000)
  })
  var promises = Promise.race([promise1, promise2]) // 调借口10秒内没反应，setTimeout结果返回，报个错
  .then((promses) => {})
  .catch((err) => {
    throw err
  })
  ```

- **Promise.resolve**/**Promise.reject**：将现有对象转化为promise对象，参数如果是promise对象，原封不动返回；如果是其它,则相当于`new Promise((resolve) => resolve(args))`, 基本就是个立即执行的异步操作
  `.then(() => {return a})`,这里的return a应该就是Promise.resolve(a),所以then里第一个参数可以返回一个promise，这个then后面的得等这个promise里resolve触发了才能继续进行

##Generator
- 什么是**generator**
  **generator**函数是es6提供的一种**异步编程解决方案**，内部有多种状态，使用yield定义不同的内部状态，执行generator函数会返回这些状态构成的遍历器对象（而不会立即运行），可以通过调用遍历接口依次遍历generator函数内部的每一个状态
  每次遍历都得到一个状态对象，其中value为yield或者return后面跟随表达式的值，没有return函数执行完毕为undefined

  ```js
  function* cons () {
    yield 'hello'
    return 'world'
  }

  var result = cons()
  result.next() // {value: 'hello', done: false}
  reuslt.next() // {value: 'world', done: true}
  ```

  - **yield**:不能用在普通函数中；在表达式中用要放括号里面，如`console.log(11, (yield 33))`；用作函数参数或者用于赋值表达式右边可以不用加括号
  yield语句本身是没有返回值的，`var a = yield 1`a的值为undefined，但如果给next()传一个参数，就能赋给a一只值

  ```js
  function* a () {
    var b = yield 233
    console.log(b)
  }
  var c = a()
  c.next()
  c.next() // undefined

  var d = a()
  d.next()
  d.next(3222) // 3222
  ```

- **throw()**方法
  - generator函数返回的遍历器对象有一个throw方法，可以在函数体外抛出错误，在函数体内捕获

    ```js
    var gene = function * () {
      try {
        yield 1
      } catch (e) {
        console.log('nei')
      }
    }
    var obj = gene()
    obj.next()
    try {
      obj.throw('') // 'nei'
    } catch (e) {
      console.log('wai')
    }
    ```

  - 如果generator函数内没有部署try……catch函数，则错误会被外部捕获

  - 如果用遍历器对象的throw方法，则遍历器终止

- **return()**方法: 立即终止函数，除非有try..finally
  
  ```js
  var gene = function * () {
    yield 1
    yield 2
    yield 3
  }
  var res = gene()
  res.return(666) // {value: 666, done: true}
  ```

- **yield \* **语句：在generator函数内部调用另一个generator函数

  ```js
  function a * () {
    yield 1
    yield 2
  }
  function b * () {
    yield 3
    yield * a()
    yield 4
  }
  for (let i of b()) {
    console.log(i) // 3124
  }
  // b相当于
  function b * () {
    yield 3
    for (let i of a()) { // 即后面接一个遍历器对象，如果遍历器对象的生成函数为generator且有返回值，则yield *可以被赋值
      yield i
    }
    yield 4
  }
  // 写个函数取出嵌套数组的值
  function * getValueOfArr (arr) {
    if (Array.isArray(arr)) {
      for (let i of arr) {
        yield * getValueOfArr(i)
      }
    } else {
      yield i
    }
  }
  ```

- 作为对象属性的generator函数,两种形式都可以
  
  ```js
  var a = {
    * b () {
    },
    c: function * () {
    }
  }
  ```


####参考
[JavaScript：彻底理解同步、异步和事件循环(Event Loop)](https://segmentfault.com/a/1190000004322358)
[阮一峰的es6教程异步相关](http://es6.ruanyifeng.com/#docs/promise)