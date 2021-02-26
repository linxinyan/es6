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
### 1、Promise构造函数
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
reject函数的作用是，将Promise对象的状态从“未完成”变为“失败”（即 pending -> rejected），在异步操作`失败`时调用，并`将异步操作报出的错误，作为参数传递出去`。
### 2、then方法
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
### 3、Promise 新建后就会立即执行
    let promise = new Promise(function(resolve, reject) {
      console.log('Promise');
      resolve();
    });

    promise.then(function() {
      console.log('resolved.');
    });

    console.log('Hi!');

    // Promise   Promise 新建后立即执行，所以首先输出的是Promise
    // Hi!
    // resolved  then方法指定的回调函数，将在当前脚本所有同步任务执行完才会执行，所以resolved最后输出。
### 4、resolve函数和reject函数带有参数
如果调用resolve函数和reject函数时带有参数，那么它们的参数会被传递给回调函数。reject函数的参数通常是Error对象的实例，表示抛出的错误；`resolve函数的参数除了正常的值以外，还可能是另一个 Promise 实例`，比如像下面这样。

    const p1 = new Promise(function (resolve, reject) {
      // ...
    });

    const p2 = new Promise(function (resolve, reject) {
      // ...
      resolve(p1);  //p2返回p1的结果
    })
`一个异步操作的结果是返回另一个异步操作`
注意，这时p1的状态就会传递给p2，也就是说，`p1的状态决定了p2的状态`。如果p1的状态是pending，那么p2的回调函数就会等待p1的状态改变；如果p1的状态已经是resolved或者rejected，那么p2的回调函数将会立刻执行。
## 三、Promise.prototype.then()
它的作用是为 Promise 实例添加状态改变时的回调函数。then方法的第一个参数是resolved状态的回调函数，第二个参数是rejected状态的回调函数，`它们都是可选的`。<br>

then方法返回的是一个`新的Promise实例`（注意，不是原来那个Promise实例）。因此可以采用链式写法，即then方法后面再调用另一个then方法。<br>

采用链式的then，可以指定一组按照次序调用的回调函数。这时，前一个回调函数，有可能返回的还是一个Promise对象（即有异步操作），这时后一个回调函数，就会等待该Promise对象的状态发生变化，才会被调用。

    getJSON("/post/1.json").then(function(post) {
      return getJSON(post.commentURL);
    }).then(function (comments) {
      console.log("resolved: ", comments);
    }, function (err){
      console.log("rejected: ", err);
    });
上面代码中，第一个then方法指定的回调函数，返回的是另一个Promise对象。这时，第二个then方法指定的回调函数，就会等待这个新的Promise对象状态发生变化。如果变为resolved，就调用第一个回调函数，如果状态变为rejected，就调用第二个回调函数。<br>

如果采用箭头函数，上面的代码可以写得更简洁。

    getJSON("/post/1.json").then(
      post => getJSON(post.commentURL)
    ).then(
      comments => console.log("resolved: ", comments),
      err => console.log("rejected: ", err)
    );
    
1、finally方法总是会返回原来的值。<br>
2、catch方法，该方法返回的是一个新的 Promise 实例

## 六、Promise.all()
Promise.all()方法用于将多个 Promise 实例，包装成一个新的 Promise 实例。

    const p = Promise.all([p1, p2, p3]);
上面代码中，Promise.all()方法接受一个数组作为参数，p1、p2、p3都是 Promise 实例，如果不是，就会先调用下面讲到的Promise.resolve方法，将参数转为 Promise 实例，再进一步处理。另外，Promise.all()方法的参数可以不是数组，但必须具有 Iterator 接口，且返回的每个成员都是 Promise 实例。

    p的状态由p1、p2、p3决定，分成两种情况。

    （1）只有p1、p2、p3的状态都变成fulfilled，p的状态才会变成fulfilled，此时p1、p2、p3的返回值组成一个数组，传递给p的回调函数。

    （2）只要p1、p2、p3之中有一个被rejected，p的状态就变成rejected，此时第一个被reject的实例的返回值，会传递给p的回调函数。
## 七、Promise.race()
Promise.race()方法同样是将多个 Promise 实例，包装成一个新的 Promise 实例。

    const p = Promise.race([p1, p2, p3]);
上面代码中，`只要p1、p2、p3之中有一个实例率先改变状态，p的状态就跟着改变。那个率先改变的 Promise 实例的返回值，就传递给p的回调函数。`<br>

`Promise.race()方法的参数与Promise.all()方法一样，如果不是 Promise 实例，就会先调用Promise.resolve()方法，将参数转为 Promise 实例，再进一步处理。`<br>

## 八、Promise.allSettled()
Promise.allSettled()方法接受一组 Promise 实例作为参数，包装成一个新的 Promise 实例。只有等到`所有`这些参数实例都返回结果，`不管是fulfilled还是rejected`，包装实例才会结束。<br>

`该方法返回的新的 Promise 实例，一旦结束，状态总是fulfilled，`不会变成rejected。状态变成fulfilled后，Promise 的监听函数接收到的参数是一个`数组`，每个成员对应一个传入Promise.allSettled()的 Promise 实例。<br>

    const resolved = Promise.resolve(42);
    const rejected = Promise.reject(-1);

    const allSettledPromise = Promise.allSettled([resolved, rejected]); 
     //该方法返回的新的 Promise 实例（即allSettledPromise），状态只能是fulfilled

    allSettledPromise.then(function (results) {
      console.log(results);
      //Promise 的监听函数接收到的参数是一个数组，每个成员对应一个传入Promise.allSettled()的 Promise 实例
    });
    // [
    //    { status: 'fulfilled', value: 42 },
    //    { status: 'rejected', reason: -1 }
    //每个对象都有status属性，该属性的值只可能是字符串fulfilled或字符串rejected。
    //fulfilled时，对象有value属性，rejected时有reason属性，对应两种状态的返回值。
    // ]

有时候，我们不关心异步操作的结果，只关心`这些操作有没有结束`。这时，`Promise.allSettled()方法`就很有用。如果没有这个方法，想要确保所有操作都结束，就很麻烦
## 九、Promise.any()
ES2021 引入了Promise.any()方法。该方法接受一组 Promise 实例作为参数，包装成一个新的 Promise 实例返回。只要参数实例有一个变成fulfilled状态，包装实例就会变成fulfilled状态；如果所有参数实例都变成rejected状态，包装实例就会变成rejected状态。<br>

Promise.any()跟Promise.race()方法很像，只有一点不同，就是不会因为某个 Promise 变成rejected状态而结束。<br>

Promise.any()抛出的错误，不是一个一般的错误，而是一个 AggregateError 实例。它相当于一个数组，每个成员对应一个被rejected的操作所抛出的错误。
十、Promise.resolve()
将现有对象转为 Promise 对象

    Promise.resolve('foo')
    // 等价于
    new Promise(resolve => resolve('foo'))
Promise.resolve()方法的参数分成四种情况。<br>
（1）参数是一个 Promise 实例 <br>
如果参数是 Promise 实例，那么Promise.resolve将不做任何修改、原封不动地返回这个实例。 <br>
（2）参数是一个thenable对象 <br>
thenable对象指的是具有then方法的对象，比如下面这个对象。

    let thenable = {
      then: function(resolve, reject) {
        resolve(42);
      }
    };
Promise.resolve()方法会将这个对象转为 Promise 对象，执行Promise 对象的then()方法就会执行thenable对象的then()方法。<br>
（3）参数不是具有then()方法的对象，或根本就不是对象 <br>
如果参数是一个原始值，或者是一个不具有then()方法的对象，则Promise.resolve()方法返回一个新的 Promise 对象，`状态为resolved`。

    const p = Promise.resolve('Hello');

    p.then(function (s) {
      console.log(s)
    });
    // Hello
由于字符串Hello不属于异步操作（`判断方法是字符串对象不具有 then 方法`），返回 Promise 实例的状态从一生成就是resolved，所以回调函数会立即执行。Promise.resolve()方法的参数，会同时传给回调函数。<br>
（4）不带有任何参数 <br>
Promise.resolve()方法允许调用时不带参数，直接返回一个resolved状态的 Promise 对象。所以，如果希望得到一个 Promise 对象，比较方便的方法就是直接调用Promise.resolve()方法。
需要注意的是，`立即resolve()的 Promise 对象，是在本轮“事件循环”（event loop）的结束时执行，而不是在下一轮“事件循环”的开始时。`<br>

    setTimeout(function () {
      console.log('three');
    }, 0);

    Promise.resolve().then(function () {
      console.log('two');
    });

    console.log('one');

    // one
    // two
    // three
上面代码中，setTimeout(fn, 0)在下一轮“事件循环”开始时执行，Promise.resolve()在本轮“事件循环”结束时执行，console.log('one')则是立即执行，因此最先输出。
## 十一、Promise.reject()
Promise.reject(reason)方法会返回一个新的 Promise 实例，该实例的状态为rejected。reason参数作为reject的理由，变成后续方法的参数。

    Promise.reject('出错了')
    .catch(e => {
      console.log(e === '出错了')
    })
    // true
上面代码中，Promise.reject()方法的参数是一个字符串，`后面catch()方法的参数e就是这个字符串（reason参数）`。
## 十二、应用
### 1、加载图片
我们可以将图片的加载写成一个Promise，一旦加载完成，Promise的状态就发生变化。

    const preloadImage = function (path) {
      return new Promise(function (resolve, reject) {
        const image = new Image();
        image.onload  = resolve;
        image.onerror = reject;
        image.src = path;
      });
    };
### 2、Generator 函数与 Promise 的结合
使用 Generator 函数管理流程，遇到异步操作的时候，通常返回一个Promise对象。
## 十三、Promise.try()
事实上，Promise.try就是模拟try代码块，就像promise.catch模拟的是catch代码块。
