## 一、Promise的含义
 Promise，简单说就是一个容器，里面保存着某个未来才会结束的事件（通常是`一个异步操作`）的结果。从语法上说，Promise 是一个对象，从它可以获取异步操作的消息
 ### 1、Promise对象的两个特点。

（1）对象的状态不受外界影响。Promise对象代表`一个异步操作`，有三种状态：pending（进行中）、fulfilled（已成功）和rejected（已失败）。只有异步操作的结果，可以决定当前是哪一种状态，任何其他操作都无法改变这个状态。

（2）Promise对象的状态改变，只有两种可能：pending->fulfilled和pending->rejected。只要这两种情况发生，`状态就凝固了，不会再变了`，会一直保持这个结果，这时就称为 resolved（已定型）。如果改变已经发生了，你再对Promise对象添加回调函数，也会立即得到这个结果。这与事件（Event）完全不同，事件的特点是，如果你错过了它，再去监听，是得不到结果的。

### 2、Promise优点 
有了Promise对象，就可以将异步操作以同步操作的流程表达出来，避免了层层嵌套的回调函数。此外，Promise对象提供统一的接口，使得控制异步操作更加容易。

### 3、Promise缺点。
首先，无法取消Promise，一旦新建它就会立即执行，无法中途取消。其次，如果不设置回调函数，Promise内部抛出的错误，不会反应到外部。第三，当处于pending状态时，无法得知目前进展到哪一个阶段（刚刚开始还是即将完成）。
#### 4、`如果某些事件不断地反复发生，一般来说，使用 Stream 模式是比部署Promise更好的选择。`

## 二、基本用法
ES6 规定，`Promise对象是一个构造函数`，用来生成Promise实例。

    const promise = new Promise(function(resolve, reject) {
      // ... some code

      if (/* 异步操作成功 */){
        resolve(value);
      } else {
        reject(error);
      }
    });
Promise构造函数接受一个`函数作为参数`，该函数的两个参数分别是resolve和reject。它们是两个`函数`，由 JavaScript 引擎提供，不用自己部署。<br>

resolve函数的作用是，将Promise对象的状态从“未完成”变为“成功”（即 pending -> resolved），在异步操作`成功`时调用，并`将异步操作的结果，作为参数传递出去`。<br>
reject函数的作用是，将Promise对象的状态从“未完成”变为“失败”（即 pending -> rejected），在异步操作`失败`时调用，并`将异步操作报出的错误，作为参数传递出去`。<br>

Promise实例生成以后，可以用then方法分别指定resolved状态和rejected状态的回调函数。

    promise.then(function(value) {  //value：promise对象异步操作成功的结果
      // success
    }, function(error) {  //error：promise对象异步操作失败的错误
      // failure
    });
then方法可以接受两个`回调函数`作为参数。第一个回调函数是Promise对象的状态变为resolved时调用，第二个回调函数是Promise对象的状态变为rejected时调用。这两个函数都是可选的，不一定要提供。它们都接受Promise对象传出的值作为参数。<br>

下面是一个Promise对象的简单例子。

    function timeout(ms) {
      return new Promise((resolve, reject) => {    //Promise构造函数的参数是一个函数，该函数的参数是resolve函数和reject函数
        setTimeout(resolve, ms, 'done');  //setTimeout函数的第三个及其之后的参数作为第一个参数（函数）的参数传入该函数
       //这里把'done'字符串作为resolve函数的参数传入，并在ms参数时间后执行resolve函数
      });
    }

    timeout(100).then((value) => {  
      console.log(value);
    });
上面代码中，timeout方法返回一个Promise实例，表示一段时间以后才会发生的结果。过了指定的时间（ms参数）以后，Promise实例的状态变为resolved，就会触发then方法绑定的回调函数。
