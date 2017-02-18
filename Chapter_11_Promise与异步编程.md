# Chapter 11 Promises and asynchronous Programming

One of the most powerful aspects of JavaScript is how easily it handles asynchro-
nous programming. As a language created for the web, JavaScript needed to be able to
respond to asynchronous user interactions, such as clicks and key presses, from the beginning. 
Node.js made asynchronous programming in JavaScript more popular by using callbacks as an alternative to events. But as more programs started using asynchronous programming, events and callbacks weren’t powerful enough to support everything developers wanted to do. Promises are the solution to this problem.  

Promises are another option for asynchronous programming, and they work like futures and deferreds do in other languages. Like events and call- backs, a promise speci es some code to be executed later, but promises
also explicitly indicate whether the code succeeded or failed. You can chain promises together based on success or failure in ways that make your code easier to understand and debug.  

This chapter shows you how promises work. However, for a complete understanding, it’s important to understand some of the basic concepts upon which promises are built.  

## Asynchronous Programming Background
JavaScript engines are built on the concept of a single-threaded event loop. Single-threaded means that only one piece of code is executed at a time. Contrast this with languages like Java or C++, where threads can allow mul- tiple different pieces of code to execute at the same time. Maintaining and protecting state when multiple pieces of code can access and change that state is a dif cult problem and a frequent source of bugs in thread-based software.  

JavaScript engines can execute only one piece of code at a time, so they need to keep track of code that is meant to run. That code is kept in a job queue. Whenever a piece of code is ready to be executed, it is added to the job queue. When the JavaScript engine is  nished executing code, the event loop executes the next job in the queue. The event loop is a process inside the JavaScript engine that monitors code execution and manages the job queue. Keep in mind that as a queue, job execution runs from the  rst job in the queue to the last.  

### The Event Model 事件模型
When a user clicks a button or presses a key on the keyboard, an event like onclick is triggered. That event might respond to the interaction by adding a new job to the back of the job queue. This is JavaScript’s most basic form of asynchronous programming. The event handler code doesn’t execute until the event  res, and when it does execute, it has the appropriate con- text. For example:  

当用户点击按钮或者在键盘上按压按键时，就会触发onclick这样的事件。这个时间可以

```js
let button = document.getElementById("my-btn");
             button.onclick = function(event) {
                 console.log("Clicked");
             };
```

In this code, console.log("Clicked") will not be executed until button is clicked. When button is clicked, the function assigned to onclick is added to the back of the job queue and will be executed when all other jobs ahead of it are complete.
Events work well for simple interactions, but chaining multiple sepa- rate asynchronous calls together is more complicated because you must keep track of the event target (button in this example) for each event. Additionally, you need to ensure that all appropriate event handlers are added before the  rst time an event occurs. For instance, if button is clicked before onclick is assigned, nothing will happen. So although events are use- ful for responding to user interactions and similar infrequent functionality, they aren’t very flexible for more complex needs.  

### The Callback Pattern 回调模式
Node.js advanced the asynchronous programming model by popularizing callbacks. The callback pattern is similar to the event model because the asynchronous code doesn’t execute until a later point in time. It’s different because the function to call is passed in as an argument, as shown here:  

Node.js通过让回调的流行改进了异步编程模型。回调模型类似于实践模型，因为异步编程代码直到某个时间点后才会真正执行。这里的区别是因为需要调用的函数是作为参数被传递的，一如以下所示：  

```js
readFile("example.txt", function(err, contents) {
    if (err) {
        throw err; 
    }
    console.log(contents);
});
console.log("Hi!");
```

This example uses the traditional Node.js error- rst callback style. The readFile() function is intended to read from a  le on disk (speci ed as the  rst argument) and then execute the callback (the second argument) when complete. If there’s an error, the err argument of the callback is an error object; otherwise, the contents argument contains the file contents as a string.  

本例子使用的是传统的Node.js中的错误优先回调风格。readFile()函数被用作从磁盘（指定为第一个参数）读取文件并再完成时执行回调（第二个参数）。如果有错误，则回调的err参数是个错误对象；否则，contents参数将包含内容为字符串的文件。  

Using the callback pattern, readFile() begins executing immediately and pauses when it starts reading from the disk. That means console.log("Hi!") is output immediately after readFile() is called, before console.log(contents) prints anything. When readFile()  nishes, it adds a new job to the end of the job queue with the callback function and its arguments. That job executes upon completion of all other jobs ahead of it.  

在使用回调模式时，readFile()开始立即执行并在它开始从磁盘读取时暂停。这就意味着，console.log("Hi!")在readFile()调用之后，在console.log(contents)打印任何内容之前被立即输出。当readFile()结束时，

The callback pattern is more  exible than events because chaining multiple calls together is easier with callbacks. Here’s an example:  

回调模式比事件更为灵活是因为你可以在回调中把多次调用放到一起。这里就有一个例子：   

```js
readFile("example.txt", function(err, contents) {
    if (err) {
        throw err; 
    }
    writeFile("example.txt", function(err) {
        if (err) {
            throw err; 
        }
        console.log("File was written!");
    });
});
```

In this code, a successful call to readFile() results in another asynchro- nous call, this time to the writeFile() function. Note that the same basic pattern of checking err is present in both functions. When readFile() is com- plete, it adds a job to the job queue that calls the writeFile() function if there are no errors. Then, writeFile() adds a job to the job queue when it finishes.  

在这段代码中，成功在另一个异步调用中调用了readFile()结果，这一次则是writeFile()函数。注意在两个函数中都出现了相同的err检查基本模式。当readFile()完成时，如果没有错误，它会添加一个任务到调用writeFile()的函数的任务队列。然后，writeFile()在完成时添加一个任务到任务队列。  

This pattern works fairly well, but you can quickly  nd yourself in callback hell. Callback hell occurs when you nest too many callbacks, like this:  

这种模式用起来相当不错，但是很快你会发现自己身处回调地狱之中。回调地狱出现在你嵌套多个回调之时，就像这样：  

```js
method1(function(err, result) {
    if (err) {
        throw err;
    }
    method2(function(err, result) {
        if (err) {
            throw err;
        }
        method3(function(err, result) {
            if (err) {
                throw err;
            }
            method4(function(err, result) {
                if (err) {
                    throw err;
                }
                method5(result);
            });
        }); 
    });
});
```

Nesting multiple method calls, as this example does, creates a tangled web of code that is dif cult to understand and debug. Callbacks also pres- ent problems when you want to implement more complex functionality. What if you want two asynchronous operations to run in parallel and notify you when they’re both complete? What if you want to start two asynchro- nous operations at the same time but only take the result of the  rst one
to complete? In these cases, you’d need to track multiple callbacks and cleanup operations, and promises greatly improve such situations.  

嵌套多个方法调用，就像本例所做的那样，

## Promise Basics Promise基础
A promise is a placeholder for the result of an asynchronous operation. Instead of subscribing to an event or passing a callback to a function, the function can return a promise, as shown here:  

Promise是一个用于异步操作结果容器。与订阅一个事件或者传递一个回调到函数不同，该函数可以反悔一个Promise，一如此处所示：  

```js
// readFile promises to complete at some point in the future
let promise = readFile("example.txt");
```

In this code, readFile() doesn’t start reading the  le immediately: that will happen later. Instead, the function returns a promise object represent- ing the asynchronous read operation so you can work with it in the future. Exactly when you’ll be able to work with that result depends entirely on how the promise’s life cycle concludes.  

在本代码中，readFile()并没有开始立即读取文件：而是在稍后发生。相反，函数返回一个代表着异步操作的promise对象，这样你就可以在之后使用它了。当然了，在你能够使用这个结果时完全依赖于promise的是被如何约定的。  

### The Promise Life Cycle Promise的生命周期
Each promise goes through a short life cycle starting in the pending state, which indicates that the asynchronous operation hasn’t completed yet.  

每个promise在挂起状态中都有一个短暂的生命周期，它表示异步操作还未完成。  

A pending promise is considered unsettled. The promise in the previous example is in the pending state as soon as the readFile() function returns it. Once the asynchronous operation completes, the promise is considered settled and enters one of two possible states:  

挂起中的promise被认为是不稳定的。之前例子中promise处于尽快readFile()函数结果返回的挂起状态。一旦异步操作完成，

- Fulfilled: The promise’s asynchronous operation has completed successfully.
- Rejected: The promise’s asynchronous operation didn’t complete successfully due to either an error or some other cause.

- Fulfilled： promise的异步操作已经成功完成。
- Rejected：promise的异步操作出于错误或者其他原因并没有完全成功。

An internal [[PromiseState]] property is set to "pending", "fulfilled",
or "rejected" to re ect the promise’s state. This property isn’t exposed on promise objects, so you can’t determine which state the promise is in programmatically. But you can take a speci c action when a promise changes state by using the then() method.  

[[PromiseState]]内部的属性被设置为“挂起”，”完成“，或者”拒绝“以映射promise的状态。该属性并不会暴露出promise对象，所以你不能够

The then() method is present on all promises and takes two arguments. The  rst argument is a function to call when the promise is ful lled. Any additional data related to the asynchronous operation is passed to this ful llment function. The second argument is a function to call when the promise is rejected. Similar to the ful llment function, the rejection func- tion is passed any additional data related to the rejection.  

then()方法出现在所有promise对象上，并接受两个参数。第一个参数是在promise完成时调用的函数。

>#### Note 
>Any object that implements the then() method as described in the preceding para- graph is called a thenable. All promises are thenables, but all thenables are not promises.

>任何实现在

Both arguments to then() are optional, so you can listen for any combina- tion of ful llment and rejection. For example, consider this set of then() calls:  

这两个参数对于then()都是可选的，所以你可以

```js
let promise = readFile("example.txt");
promise.then(function(contents) {
    // fulfillment
    console.log(contents);
}, function(err) {
// rejection
    console.error(err.message);
});
promise.then(function(contents) {
    // fulfillment
    console.log(contents);
});
promise.then(null, function(err) {
    // rejection
    console.error(err.message);
});
```

All three then() calls operate on the same promise. The  rst call listens for ful llment and rejection. The second only listens for ful llment; errors won’t be reported. The third just listens for rejection and doesn’t report success.
Promises also have a catch() method that behaves the same as then() when only a rejection handler is passed. For example, the following catch() and then() calls are functionally equivalent:  

这三个then()调用全部操作的是相同的promise。

```js
promise.catch(function(err) {
    // rejection
    console.error(err.message);
});
// is the same as:
promise.then(null, function(err) {
    // rejection
    console.error(err.message);
});
```

The then() and catch() methods are intended to be used in combination to properly handle the result of asynchronous operations. This system is better than using events and callbacks because it clearly indicates whether the operation succeeded or failed completely. (Events tend not to  re when there’s an error, and in callbacks you must always remember to check the error argument.) Just know that if you don’t attach a rejection handler to a promise, all failures will happen silently. Always attach a rejection handler, even if the handler just logs the failure.  

A ful llment or rejection handler will still be executed even if it is added to the job queue after the promise is already settled. This allows you to add new ful llment and rejection handlers at any time and guarantee that they will be called. For example:  

```js
let promise = readFile("example.txt");
// original fulfillment handler
promise.then(function(contents) {
    console.log(contents);
    // now add another
    promise.then(function(contents) {
        console.log(contents);
    });
});
```

In this code, the ful llment handler adds another ful llment handler to the same promise. The promise is already ful lled at this point, so the new ful llment handler is added to the job queue and called when all other preceding jobs on the queue are complete. Rejection handlers work the same way.  

>#### Note
>Each call to then() or catch() creates a new job to be executed when the promise is resolved. But these jobs end up in a separate job queue that is reserved strictly for promises. The precise details of this second job queue aren’t important for under- standing how to use promises as long as you understand how job queues work in general.


### Creating Unsettled Promises
New promises are created using the Promise constructor. This constructor accepts a single argument: a function called the executor, which contains the code to initialize the promise. The executor is passed two functions named resolve() and reject() as arguments. The resolve() function is called when the executor has  nished successfully to signal that the promise is ready to be resolved, whereas the reject() function indicates that the executor has failed.  

Here’s an example that uses a promise in Node.js to implement the readFile() function you saw earlier in this chapter:  

```js
// Node.js example
let fs = require("fs");
function readFile(filename) {
    return new Promise(function(resolve, reject) {
        // trigger the asynchronous operation
        fs.readFile(filename, { encoding: "utf8" }, function(err, contents) {
            // check for errors
            if (err) {
                reject(err);
return; }
            // the read succeeded
            resolve(contents);
}); });
}
let promise = readFile("example.txt");
// listen for both fulfillment and rejection
promise.then(function(contents) {
    // fulfillment
    console.log(contents);
}, function(err) {
// rejection
    console.error(err.message);
});
```

In this example, the native Node.js fs.readFile() asynchronous call is wrapped in a promise. The executor either passes the error object to the reject() function or passes the  le contents to the resolve() function.  

Keep in mind that the executor runs immediately when readFile() is called. When either resolve() or reject() is called inside the executor, a job is added to the job queue to resolve the promise. This is called job sched- uling, and if you’ve ever used the setTimeout() or setInterval() functions, you’re already familiar with it. In job scheduling, you add a new job to the job queue to say, “Don’t execute this right now, but execute it later.” For instance, the setTimeout() function lets you specify a delay before a job is added to the queue:  

```js
// add this function to the job queue after 500 ms have passed
setTimeout(function() {
    console.log("Timeout");
}, 500)
console.log("Hi!");
```

This code schedules a job to be added to the job queue after 500 ms. The two console.log() calls produce the following output:  

```js
Hi! 
Timeout
```

Thanks to the 500 ms delay, the output that the function passed to setTimeout() was shown after the output from the console.log("Hi!") call.  

Promises work similarly. The promise executor executes immediately, before anything that appears after it in the source code. For instance:  

```js
let promise = new Promise(function(resolve, reject) {
    console.log("Promise");
    resolve();
});
console.log("Hi!");
```

The output for this code is:  

```js
Promise Hi!
```

Calling resolve() triggers an asynchronous operation. Functions passed to then() and catch() are executed asynchronously, because these are also added to the job queue. Here’s an example:  

```js
let promise = new Promise(function(resolve, reject) {
    console.log("Promise");
    resolve();
});
promise.then(function() {
    console.log("Resolved.");
});
console.log("Hi!");
```

The output for this example is:  

```js
Promise
Hi!
Resolved
```

Note that even though the call to then() appears before the line console.log("Hi!"), it doesn’t actually execute until later (unlike the exe- cutor). The reason is that ful llment and rejection handlers are always added to the end of the job queue after the executor has completed.  





