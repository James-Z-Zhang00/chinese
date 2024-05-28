---
标题: JavaScript Promises 工作原理 - 初学者手册
作者: 乔·阿塔迪（Joe Attardi）
作者页面: https://www.freecodecamp.org/news/author/joeattardi/
原始页面: https://www.freecodecamp.org/news/the-javascript-promises-handbook/
译者: "James Z. Zhang"
reviewer: ""
---

2024 年 2 月 13 日 / [#JavaScript][1]

<!-- more -->

# JavaScript Promises 工作原理 - 初学者手册

![Joe Attardi](https://www.freecodecamp.org/news/content/images/size/w60/2023/10/5.png)

[乔·阿塔迪（Joe Attardi）][2]

  ![How JavaScript Promises Work – Handbook for Beginners](https://www.freecodecamp.org/news/content/images/size/w2000/2024/02/How-JavaScript-Promises-Work-Cover.png)

许多操作，如网络请求，本质上是异步的。在处理异步代码时，Promise 是最有用和最强大的工具之一。在这本手册中，您将了解有关 JavaScript Promises 的所有信息以及如何使用它们。

## 目录

1.  [Promise 是什么][3]
2.  [Promises 与其他异步模式的比较][4]
3.  [如何创建一个 Promise][5]
4.  [如何获取 Promise 的结果][6]
5.  [如何使用`then`处理错误][7]
6.  [Promise 链接][8]
7.  [如何创建立即满足或拒绝的 Promises][9]
8.  [如何使用`async`和`await`][10]
9.  [反模式 Promise][11]
10. [总结][12]

## Promise 是什么

首先，让我们了解一下什么是 Promise。

简单来说，Promise 是一个代表异步操作的对象。这个对象可以告诉你操作何时成功，或者何时失败。

当你调用一个基于 Promise 的 API 时，函数返回一个 Promise 对象来告诉你最终的操作结果。

### Promise 的状态

在其生命周期中，Promise 可以处于以下三种状态之一：

-   **待定（Pending）**: 当操作仍在进行中时，Promise 处于待定状态。它处于闲置状态，等待最终结果或错误。
-   **已实现（Fulfilled）**: 返回 Promise 的异步任务成功完成。Promise 被一个值实现，这个值是操作的结果。
-   **已拒绝（Rejected）**: 如果异步操作失败，Promise 被认为是拒绝的。Promise 会有一个_拒绝理由_。这通常是一个错误（Error）对象，也可以是任何错误值——甚至是一个简单的数字或字符串！

A Promise starts out in the pending state, then depending on the result, will transition to either the fulfilled or rejected state. A Promise is said to be _settled_ once it reaches either the fulfilled or rejected state.
Promise 一开始处于待定（pending）状态，然后根据结果，会过渡到实现（fulfilled）或拒绝（rejected）状态。一旦 Promise 达到实现或拒绝状态，就被认为是 _已解决（settled）_。

Of course, there is no guarantee that the asynchronous task will ever complete. It's completely possible for a `Promise` to remain in the pending state forever, though this would be because of a bug in the asynchronous task's code.
当然，不是所有的异步任务都会最终完成。一个 `Promise` 可能永远保持在待定状态，尽管这通常是由于异步任务的代码中存在错误。

## Comparing Promises to Other Async Patterns
## Promises 与其他异步模式的比较

Promises behave a little differently from other asynchronous patterns in JavaScript. Before diving deeper into Promises, let's briefly compare Promises to these other techniques.
Promise 与 JavaScript 中的其他异步模式有些不同。在深入了解 Promise 之前，让我们简要比较一下 Promise 和这些其他技术。

### 回调函数

A callback function is a function that you pass to another function. When the function you call has finished its work, it will execute your callback function with the result.
回调函数是你传递给另一个函数的函数。当你调用的函数完成其工作后，它将执行你的回调函数，并提供结果。

Imagine a function called `getUsers` which will make a network request to get an array of users. You can pass a callback function to `getUsers`, which will be called with the array of users once the network request is complete:
想象一个名为 `getUsers` 的函数，它将发起网络请求以获取用户数组。你可以将一个回调函数传递给 `getUsers`，一旦网络请求完成，这个回调函数就会被调用，并传入用户数组：

```javascript
console.log('Preparing to get users');
getUsers(users => {
  console.log('Got users:', users);
});
console.log('Users request sent');
```

An example of a callback function
回调函数的一个例子

First, the above code will print "Preparing to get users". Then it calls `getUsers` which will initiate the network request. But JavaScript doesn't wait for the request to complete. Instead, it immediately executes the next `console.log` statement.
首先，上述代码将打印（在 console 中显示，译者注）“准备获取用户”。然后它调用 `getUsers`，这将启动网络请求。但是 JavaScript 不会等待请求完成。相反，它会立即执行下一个 console.log 语句。

Later, once the users have been loaded, your callback will be executed and "Got users" will be printed.
稍后，一旦用户被加载，你的回调函数将被执行，屏幕上将会显示“已获取用户”。

Some callback-based APIs, such as many Node.js APIs, use _error-first callbacks_. These callback functions take two arguments. The first argument is an error, and the second is the result.
一些基于回调的 API，如许多 Node.js API，使用 _错误优先的回调（error-first callbacks）_。这些回调函数有两个参数。第一个参数是错误（error），第二个是结果（result）。

Typically, only one of these will have a value, depending on the outcome of the operation. This is similar to the fulfilled and rejected Promise states.
通常，这两者中只有一个会有值，这取决于操作的结果。这与 Promise 的已实现和已拒绝状态类似。

The trouble with callback APIs is that of nesting. If you need to make multiple asynchronous calls in sequence, you’ll end up with nested function calls and callbacks.
回调 API 的问题在于嵌套。如果你需要按顺序进行多个异步调用，你将结束于嵌套的函数调用和回调。

Imagine you want to read a file, process some data from that file, then write a new file. All three of these tasks are asynchronous and use an imaginary callback based API.
想象一下，你想读取一个文件，处理该文件中的一些数据，然后写一个新文件。这三个任务都是异步的，并使用一个假想的基于回调的 API。

```javascript
readFile('sourceData.json', data => {
	processData(data, result => {
		writeFile(result, 'processedData.json', () => {
			console.log('Done processing');
		});
	});
});
```

A sequence of nested callbacks
回调序列嵌套

It gets even more unwieldy with error handling. Imagine these functions used error-first callbacks:
在错误处理方面，情况变得更加复杂。想象一下，这些函数使用错误优先的回调：

```javascript
readFile('sourceData.json', (error, data) => {
	if (error) {
		console.error('Error reading file:', error);
		return;
	}
	
	processData(data, (error, result) => {
		if (error) {
			console.error('Error processing data:', error);
			return;
		}
		
		writeFile(result, 'processedData.json', error => {
			if (error) {
				console.error('Error writing file:', error);
				return;
			}
			
			console.log('Done processing');
		});
	});
});
```

A sequence of nested error-first callbacks
错误优先的回调序列嵌套

Callback functions aren't typically used directly as an asynchronous mechanism in modern APIs, but as you'll soon see, they are the foundation for other types of asynchronous tools such as Promises.
现代 API 中的异步机制通常不直接使用回调函数，但正如你很快会看到的，它们是其他类型异步工具（如 Promise）的基础。

### Events
### 事件（Events）

An event is something that you can listen for and respond to. Some objects in JavaScript are event _emitters_, which means you can register event listeners on them.
事件是你可以监听并做出响应的东西。在 JavaScript 中，一些对象是事件的 _发射器（emitters）_，这意味着你可以在它们上注册事件监听器（event listener，用来监视软件的变化，比如用户点击了鼠标。译者注）。

In the DOM, many elements implement the `EventTarget` interface which provides `addEventListener` and `removeEventListener` methods.
在 DOM 中，许多元素实现了 `EventTarget` 接口，它提供了 `addEventListener` 和 `removeEventListener` 方法。 ------(more accurate translation needed)

A given type of event can occur more than once. For example, you can listen for the click event on a button:
一个给定类型的事件可以发生多次。例如，你可以监听按钮上的点击事件（click event）：

```javascript
myButton.addEventListener('click', () => {
   console.log('button was clicked!'); 
});
```

Listening for a click on a button
监听按钮的点击 ------(more accurate translation needed)

Every time the button is clicked, the text "button was clicked!" will be printed to the console.
每次按钮被点击时，控制台都会打印“button was clicked!”（按钮被点击了！）。

`addEventListener` itself accepts a callback function. Whenever the event occurs, the callback is executed.
`addEventListener` 本身接受一个回调函数。每当事件发生时，回调函数就会被执行。

An object can emit multiple types of events. Consider an image object. If the image at the specified URL is loaded successfully, the `load` event is triggered. If there was an error, this event is not triggered and instead the `error` event is triggered.
一个对象可以发出多种类型的事件。考虑一个图像对象。如果指定 URL 的图像成功加载，`加载事件（load）` 将被触发。如果发生错误，这个事件不会触发，而是触发 `错误事件（error）`。

```javascript
myImage.addEventListener('load', () => {
    console.log('Image was loaded');
});

myImage.addEventListener('error', error => {
   console.error('Image failed to load:', error); 
});
```

Listening for an image's load and error events
监听图像的加载和错误事件

Suppose the image already completed loading before you added the event listener. What do you think would happen? Nothing! One drawback of event-based APIs is that if you add an event listener after an event, your callback won't be executed. This makes sense, after all – you wouldn't want to receive all past click events when you add a click listener to a button.
假设在你添加事件监听器之前，图像已经完成了加载。你认为会发生什么呢？什么也不会发生！基于事件的 API 的一个缺点是，如果你在事件发生后添加事件监听器，你的回调将不会被执行。这是有道理的——毕竟，当你向按钮添加点击监听器时，你不会想接收所有过去的点击事件。

Now that we've explored callbacks and events, let's take a closer look at Promises.
现在我们已经探讨了回调和事件，让我们更仔细地看看 Promise。

## How to Create a Promise
## 如何创建一个 Promiose
You can create a Promise using the `new` keyword with the `Promise` constructor. The `Promise` constructor takes a callback function that takes two arguments, called `resolve` and `reject`. Each of these arguments is a function provided by the Promise, which are used to transition the Promise to either the fulfilled or rejected state.
你可以使用 `new` 关键字和 `Promise` 构造器来创建一个 `Promise`。`Promise` 构造器接受一个回调函数，该函数带有两个参数，称为 `resolve` 和 `reject`。这两个参数都是由 `Promise` 提供的函数，用于将 `Promise` 转换到已实现或已拒绝状态。

Inside your callback, you perform your asynchronous work. If the task is successful, you call the `resolve` function with the final result. If there was an error, you call the `reject` function with the error.
在你的回调中，你执行你的异步工作。如果任务成功，你用最终结果调用 `resolve` 函数。如果发生错误，你用错误调用 `reject` 函数。

Here's an example of creating a Promise that wraps the browser's `setTimeout` function:
以下是一个创建 Promise 的示例，该示例包装了浏览器的 `setTimeout` 函数：

```javascript
function wait(duration) {
	return new Promise(resolve => {
        setTimeout(resolve, duration);
    });
}
```

将 setTimeout 包装在 Promise 中 ------(more accurate translation needed)

The `resolve` function is passed as the first argument to `setTimeout`. After the time specified by `duration` has passed, the browser calls the `resolve` function which fulfills the Promise.
`resolve` 函数作为第一个参数传递给 `setTimeout`。在由 `duration` 指定的时间过后，浏览器调用 `resolve` 函数，这会实现 Promise。

Note: In this example, the delay before the `resolve` function is called may be longer than the duration passed to the function. This is because `setTimeout` does not guarantee execution at the specified time.
注意：在这个示例中，调用 `resolve` 函数之前的延迟可能比 duration 指定的时间要长。这是因为 `setTimeout` 并不保证在指定时间执行。------(more accurate translation needed)

It's important to note that often times, you won't actually need to construct your own Promise by hand. You will typically be working with Promises returned by other APIs.
重要的是要注意，通常情况下，你实际上不需要手动构建自己的 Promise。你通常会使用其他 API 返回的 Promise。

## How to Get the Result of a Promise
## 如何获取 Promise 的结果

We've seen how to create a Promise, but how do you actually get the result of the asynchronous operation? To do this, you call `then` on the Promise object itself. `then` takes a callback function as its argument. When the Promise is fulfilled, the callback is executed with the result.
我们已经看到了如何创建一个 Promise，但你如何实际获取异步操作的结果呢？为此，你需要在 Promise 对象本身上调用 `then`。`then` 接受一个回调函数作为其参数。当 Promise 实现时，会执行带有结果的回调函数。

Let's see an example of this in action. Imagine a function called `getUsers` that asynchronously loads a list of user objects and returns a Promise. You can get the list of users by calling `then` on the Promise returned by `getUsers`.
让我们看一个这样的例子。想象一个名为 `getUsers` 的函数，它异步加载用户对象列表并返回一个 Promise。你可以通过在 `getUsers` 返回的 Promise 上调用 `then` 来获取用户列表。

```javascript
getUsers()
  .then(users => {
    console.log('Got users:', users);
  });
```

Calling `then` on a Promise
在 Promise 上调用 `then`

Just like with events or callback based APIs, your code will continue executing without waiting for the result. Some time later, when the users have been loaded, your callback is scheduled for execution.
就像处理事件或基于回调的 API 一样，你的代码将继续执行而不是等待结果。稍后，当用户数据加载完毕时，你的回调函数将被安排执行。

```javascript
console.log('Loading users');
getUsers()
  .then(users => {
    console.log('Got users:', users);
  });
console.log('Continuing on');
```

In the above example, "Loading users" will be printed first. The next thing that is printed will be "Continuing on", because the `getUsers` call is still loading the users. Later, you'll see "Got users" printed.
在上面的例子中，首先将打印“正在加载用户”。因为 `getUsers` 调用仍在加载用户，所以接下来打印的将是“继续进行”。稍后，你将看到打印出“已获取用户”。

## How to Handle Errors with `then`
## 如何使用 `then` 处理错误

We've seen how to use `then` to get the result provided to the Promise, but what about errors? What happens if we fail to load the user list?
我们已经看到了如何使用 `then` 来获取承诺提供的结果，但如果有错误怎么办？如果我们加载用户列表失败会发生什么？

The `then` function actually takes a second argument, another callback. This is the error handler. If the Promise is rejected, this callback is executed with the rejection value.
`then` 函数实际上接受第二个参数，另一个回调。这是错误处理器（error handler）。如果 Promise 被拒绝，这个回调将会用拒绝值执行。

```javascript
getUsers()
  .then(users => {
    console.log('Got users:', users);
  }, error => {
    console.error('Failed to load users:', error);  
  });
```

Since a Promise can only ever be either fulfilled or rejected, but not both, only one of these callback functions will be executed.
由于一个 Promise 只能被实现（fulfilled）或被拒绝（rejected），而不能同时是两者，所以只有其中一个回调函数会被执行。

It's important to always handle errors when working with Promises. If you have a Promise rejection that isn't handled by an error callback, you'll get an exception in your console about an unhandled rejection, which can cause issues for your users at runtime.
在使用 Promise 时，始终处理错误总是非常重要。如果你有一个 Promise 拒绝没有通过错误回调处理，你的控制台会出现一个未处理的拒绝异常，这可能会在运行时为你的用户造成问题。

## Promise Chaining
## Promise 链接

What if you need to work with multiple Promises in series? Consider the earlier example where we loaded some data from a file, did some processing, and wrote the result to a new file. Suppose the `readFile`, `processData`, and `writeFile` functions used Promises instead of callbacks.
如果你需要连续处理多个 Promise 怎么办？考虑之前的例子，我们从一个文件加载了一些数据，进行了一些处理，然后将结果写入一个新文件。假设 `readFile`，`processData` 和 `writeFile` 函数使用 Promise 而不是回调。

You might try something like this:
你可能会尝试像这样做：

```javascript
readFile('sourceData.json')
  .then(data => {
    processData(data)
      .then(result => {
        writeFile(result, 'processedData.json')
          .then(() => {
            console.log('Done processing');
          });
      });
  });
```

Nested promises
Promise 的嵌套

This doesn't look great, and we still have the nesting issue that we had with the callback approach. Thankfully, there is a better way. You can chain Promises together in a flat sequence.
这看起来并不理想，我们仍然面临着我们在回调方法中遇到的嵌套问题。幸运的是，有一个更好的方法。你可以将 promise 以平面序列方式链接起来。

To see how this works, let's look deeper at how `then` works. The key idea is this: the `then` method returns _another Promise_. Whatever value you return from your `then` callback becomes the fulfilled value of this new Promise.
为了理解这是如何工作的，让我们更深入地看看 `then` 是如何工作的。关键思想是：`then` 方法返回 _另一个 promise_。你从 `then` 回调中返回的任何值都会成为这个新承诺的实现值。

Consider a `getUsers` function that returns a Promise that gets fulfilled with an array of user objects. Suppose we call `then` on this Promise, and in the callback, return the first user in the array (`users[0]`):
考虑一个返回承诺的 `getUsers` 函数，该 promise 实现时带有用户对象数组。假设我们在这个 promise 上调用 `then`，并在回调中返回数组中的第一个用户（`users[0]`）：

```javascript
getUsers().then(users => users[0]);
```

This whole expression, then, results in a new Promise that will be fulfilled with the first user object!

```javascript
getUsers()
  .then(users => users[0])
  .then(firstUser => {
    console.log('First user:', firstUser.username);
  });
```

Returning a value from the `then` handler

This process of returning a Promise, calling `then`, and returning another value, resulting in another Promise, is called chaining.

Let's extend this idea. What if, instead of returning a value from the `then` handler, we returned another Promise? Consider again the file-processing example, where `readFile` and `processData` are both asynchronous functions that return Promises:

```javascript
readFile('sourceData.json')
  .then(data => processData(data));
```

Returning another Promise from `then`

The `then` handler calls `processData`, returning the resulting Promise. As before, this returns a new Promise. In this case, the new Promise will become fulfilled when the Promise returned by `processData` is fulfilled, giving you the same value. So the code in the above example would return a Promise that will be fulfilled with the processed data.

You can chain multiple Promises, one after the other, until you get to the final value you need:

```javascript
readFile('sourceData.json')
  .then(data => processData(data))
  .then(result => writeFile(result, 'processedData.json'))
  .then(() => console.log('Done processing'));
```

Chaining multiple promises

In the above example, the whole expression will result in a Promise that won't be fulfilled until after the processed data is written to a file. "Done processing!" will be printed to the console, and then the final Promise will become fulfilled.

### Error handling in Promise chains

In our file-processing example, an error can occur at any stage in the process. You can handle an error from any step in the Promise chain by using the Promise's `catch` method.

```javascript
readFile('sourceData.json')
  .then(data => processData(data))
  .then(result => writeFile(result, 'processedData.json'))
  .then(() => console.log('Done processing'))
  .catch(error => console.log('Error while processing:', error));
```

Handling errors with `catch`

If one of the Promises in the chain is rejected, the callback function passed to `catch` will execute and the rest of the chain is skipped.

### How to use `finally`

You might have some code you want to execute regardless of the Promise result. For example, maybe you want to close a database or a file.

```javascript
openDatabase()
  .then(data => processData(data))
  .catch(error => console.error('Error'))
  .finally(() => closeDatabase());
```

### How to use `Promise.all`

Promise chains let you run multiple tasks in sequence, but what if you want to run multiple tasks at the same time, and wait until they all complete? The `Promise.all` method lets you do just that.

`Promise.all` takes an array of Promises, and returns a new Promise. This Promise will be fulfilled once all of the other Promises are fulfilled. The fulfillment value is an array containing the fulfillment values of each Promise in the input array.

Suppose you have a function `loadUserProfile` that loads a user's profile data, and  another function `loadUserPosts` that loads a user's posts. They both take a user ID as the argument. There's a third function, `renderUserPage`, that needs both the profile and list of posts.

```javascript
const userId = 100;

const profilePromise = loadUserProfile(userId);
const postsPromise = loadUserPosts(userId);

Promise.all([profilePromise, postsPromise])
  .then(results => {
    const [profile, posts] = results;
    renderUserPage(profile, posts);
  });
```

Waiting for multiple promises with `Promise.all`

What about errors? If any of the Promises passed to `Promise.all` is rejected with an error, the resulting Promise is also rejected with that error. If any of the other Promises are fulfilled, those values are lost.

### How to use `Promise.allSettled`

The `Promise.allSettled` method works similarly to `Promise.all`. The main difference is that the Promise returned by `Promise.allSettled` will never be rejected.

Instead, it is fulfilled with an array of objects, whose order corresponds to the order of the Promises in the input array. Each object has a `status` property which is either "fulfilled" or "rejected", depending on the result.

If `status` is "fulfilled", the object will also have a `value` property indicating the Promise's fulfillment value. If `status` is "rejected", the object will instead have a `reason` property which is the error or other object the Promise was rejected with.

Consider again a `getUser` function that takes a user ID and returns a Promise that is fulfilled with the user having that ID. You can use `Promise.allSettled` to load these in parallel, making sure to get all users that were loaded successfully.

```javascript
Promise.allSettled([
  getUser(1),
  getUser(2),
  getUser(3)
]).then(results => {
   const users = results
     .filter(result => result.status === 'fulfilled')
     .map(result => result.value);
   console.log('Got users:', users);
});
```

Attempting to load three users, and showing the ones that were successfully loaded

You can make a general purpose `loadUsers` function that loads users, in parallel, given an array of user IDs. The function returns a Promise that is fulfilled with an array of all users that were successfully loaded.

```javascript
function getUsers(userIds) {
  return Promise.allSettled(userIds.map(id => getUser(id)))
    .then(results => {
      return results
        .filter(result => result.status === 'fulfilled')
        .map(result => result.value);
    });
}
```

A helper function to load multiple users in parallel, filtering out any requests that failed.

Then, you can just call `getUsers` with an array of user IDs:

```javascript
getUsers([1, 2, 3])
	.then(users => console.log('Got users:', users));
```

Using the `getUsers` helper function

## How to Create Immediately Fulfilled or Rejected Promises

Sometimes, you may want to wrap a value in a fulfilled Promise. For example, maybe you have an asynchronous function that returns a Promise, but there is a base case where you know the value ahead of time and you don't need to do any asynchronous work.

To do this, you can call `Promise.resolve` with a value. This returns a Promise that is immediately fulfilled with the value you specified:

```javascript
Promise.resolve('hello')
  .then(result => {
    console.log(result); // prints "hello"
  });
```

Using `Promise.resolve`

This is more or less equivalent to the following:

```javascript
new Promise(resolve => {
   resolve('hello'); 
}).then(result => {
    console.log(result); // also prints "hello"
});
```

To make your API more consistent, you can create an immediately fulfilled Promise and return that in such cases. This way, the code that calls your function knows to always expect a Promise, no matter what.

For example, consider the `getUsers` function defined earlier. If the array of user IDs is empty, you could simply return an empty array because no users will be loaded.

```javascript
function getUsers(userIds) {
  // immediately return the empty array
  if (userIds.length === 0) {
    return Promise.resolve([]);
  }
    
  return Promise.allSettled(userIds.map(id => getUser(id)))
    .then(results => {
      return results
        .filter(result => result.status === 'fulfilled')
        .map(result => result.value);
    });
}
```

Adding an early return to the `getUsers` helper function

Another use for `Promise.resolve` is to handle the case where you are given a value that may or may not be a Promise, but you want to always treat it as a Promise.

You can safely call `Promise.resolve` on any value. If it was already a Promise, you'll just get another Promise that will have the same fulfillment or rejection value. If it was not a Promise, it will be wrapped in an immediately fulfilled Promise.

The benefit of this approach is you don't have to do something like this:

```javascript
function getResult(result) {
  if (result.then) {
     result.then(value => {
         console.log('Result:', value);
     });
  } else {
      console.log('Result:', result);
  }
}
```

Conditionally calling `then` based on whether or not something is a Promise

Similarly, you can create an immediately rejected Promise with `Promise.reject`. Returning once again to the `getUsers` function, maybe we want to immediately reject if the user ID array is `null`, `undefined`, or not an array.

```javascript
function getUsers(userIds) {
  if (userIds == null || !Array.isArray(userIds)) {
    return Promise.reject(new Error('User IDs must be an array'));
  }
    
  // immediately return the empty array
  if (userIds.length === 0) {
    return Promise.resolve([]);
  }
    
  return Promise.allSettled(userIds.map(id => getUser(id)))
    .then(results => {
      return results
        .filter(result => result.status === 'fulfilled')
        .map(result => result.value);
    });
}
```

Returning an error if the argument is not a valid array

### How to use `Promise.race`

Just like `Promise.all` or `Promise.allSettled`, the `Promise.race` static method takes an array of Promises, and returns a new Promise. As the name implies, though, it works somewhat differently.

The Promise returned by `Promise.race` will wait until the first of the given Promises is fulfilled or rejected, and then that Promise will also be fulfilled or rejected, with the same value. When this happens, the fulfilled or rejected values of the other Promises are lost.

### How to use `Promise.any`

`Promise.any` works similarly to `Promise.race` with one key difference – where `Promise.race` will be done as soon as any Promise is fulfilled or rejected, `Promise.any` waits for the first _fulfilled_ Promise.

## How to Use `async` and `await`

`async` and `await` are special keywords that simplify working with Promises. They remove the need for callback functions and calls to `then` or `catch`. They work with try-catch blocks, as well.

Here's how it works. Instead of calling `then` on a Promise, you `await` it by putting the `await` keyword before it. This effectively "pauses" execution of the function until the Promise is fulfilled.

Here's an example using standard Promises:

```javascript
getUsers().then(users => {
    console.log('Got users:', users);
});
```

Awaiting a promise with `then`

Here's the equivalent code using the `await` keyword:

```javascript
const users = await getUsers();
console.log('Got users:', users);
```

Awaiting a promise with `await`

Promise chains are a little cleaner, too:

```javascript
const data = await readFile('sourceData.json');
const result = await processData(data);
await writeFile(result, 'processedData.json');
```

Chaining promises with `await`

Remember that each usage of `await` will pause execution of the rest of the function until the Promise you are awaiting becomes fulfilled. If you want to await several Promises that run in parallel, you can use `Promise.all`:

```javascript
const users = await Promise.all([getUser(1), getUser(2), getUser(3)]);
```

Using Promise.all with `await`

To use the `await` keyword, your function must be marked as an async function. You can do this by placing the `async` keyword before your function:

```javascript
async function processData(sourceFile, outputFile) {
  const data = await readFile(sourceFile);
  const result = await processData(data);
  writeFile(result, outputFile);
}
```

Marking a function as `async`

Adding the `async` keyword also has another important effect on the function. Async functions always implicitly return a Promise. If you return a value from an async function, the function will actually return a Promise that is fulfilled with that value.

```javascript
async function add(a, b) {
  return a + b;   
}

add(2, 3).then(sum => {
   console.log('Sum is:', sum); 
});
```

An `async` function to add two numbers

In the above example, the function is returning the sum of the two arguments `a` and `b`. But since it's an `async` function, it doesn't return the sum but rather a Promise that is fulfilled with the sum.

### Error handling with `async` and `await`

We use `await` to wait for Promise to be fulfilled, but what about handling errors? If you are awaiting a Promise, and it is rejected, an error will be thrown. This means to handle the error, you can put it in a try-catch block:

```javascript
try {
    const data = await readFile(sourceFile);
    const result = await processData(data);
    await writeFile(result, outputFile);
} catch (error) {
    console.error('Error occurred while processing:', error);
}
```

Error handling with a try-catch block

## Promise Anti-Patterns

### Unnecessarily creating a new Promise

Sometimes there's no getting around creating a new Promise. But if you are already working with Promises returned by an API, you usually shouldn't need to create your own Promise:

```javascript
function getUsers() {
  return new Promise(resolve => {
     fetch('https://example.com/api/users')
       .then(result => result.json())
       .then(data => resolve(data))
  });
}
```

An example of unnecessary Promise creation

In this example, we're creating a new Promise to wrap the Fetch API, which already returns Promises. This is unnecessary. Instead, just return the Promise chain from the Fetch API directly:

```javascript
function getUsers() {
  return fetch('https://example.com/api/users')
    .then(result => result.json());
}
```

Using the existing Fetch promise

In both cases, the code calling `getUsers` looks the same:

```javascript
getUsers()
  .then(users => console.log('Got users:', users))
  .catch(error => console.error('Error fetching users:', error));
   
```

Client code for either version of the `getUsers` function

### Swallowing errors

Consider this version of a `getUsers` function:

```javascript
function getUsers() {
    return fetch('https://example.com/api/users')
    	.then(result => result.json())
    	.catch(error => console.error('Error loading users:', error));
}
```

Swallowing the fetch error

Error handling is good, right? You might be surprised by the result if we call this `getUsers` function:

```javascript
getUsers()
  .then(users => console.log('Got users:', users))
  .catch(error => console.error('error:', error);)
```

Calling `getUsers`

You might expect this to print "error", but it will actually print "Got users: undefined". This is because the `catch` call "swallows" the error and returns a new Promise that is fulfilled with the return value of the `catch` callback, which is `undefined` (`console.error` returns `undefined`). You'll still see the "Error loading users" log message from `getUsers`, but the returned Promise will be fulfilled, not rejected.

If you want to catch the error inside the `getUsers` function and still reject the returned Promise, the `catch` handler needs to return a rejected Promise. You can do this by using `Promise.reject`.

```javascript
function getUsers() {
  return fetch('https://example.com/api/users')
    .then(result => result.json())
    .catch(error => {
      console.error('Error loading users:', error);
      return Promise.reject(error);
    });
}
```

Returning a rejected Promise after handling the error

Now you'll still get the "Error loading users" message, but the returned Promise will also be rejected with the error.

### Nesting Promises

Avoid nesting Promise code. Instead, try to use flattened Promise chains.

Instead of this:

```javascript
readFile(sourceFile)
  .then(data => {
    processData(data)
      .then(result => {
        writeFile(result, outputFile)
          .then(() => console.log('done');
      });
  });
```

Do this:

```javascript
readFile(sourceFile)
  .then(data => processData(data))
  .then(result => writeFile(result, outputFile))
  .then(() => console.log('done'));
```

## Summary

Here are the key points for working with Promises:

-   A Promise can be pending, fulfilled, or rejected
-   A Promise is settled if it is either fulfilled or rejected
-   Use `then` to get the fulfilled value of a Promise
-   Use `catch` to handle errors
-   Use `finally` to perform cleanup logic that you need in either the success or error case
-   Chain Promises together to perform asynchronous tasks in sequence
-   Use `Promise.all` to get a Promise that is fulfilled when all given Promises are fulfilled, or rejects when one of those Promises rejects
-   Use `Promise.allSettled` to get a Promise that is fulfilled when all given Promises are either fulfilled or rejected
-   Use `Promise.race` to get a Promise that is fulfilled or rejected when the first of the given Promises is either fulfilled or rejected
-   Use `Promise.any` to get a Promise that is fulfilled when the first of the given Promises is fulfilled
-   Use the `await` keyword to wait for the fulfillment value of a Promise
-   Use a try-catch block to handle errors when using the `await` keyword
-   A function that uses `await` inside of it must use the `async` keyword

Thank you for reading this deep dive on Promises. I hope you learned something new!

---

![Joe Attardi](https://www.freecodecamp.org/news/content/images/size/w60/2023/10/5.png)

[Joe Attardi][13]

Read [more posts][14].

---

If you read this far, thank the author to show them you care. Say Thanks

Learn to code for free. freeCodeCamp's open source curriculum has helped more than 40,000 people get jobs as developers. [Get started][15]

[1]: /news/tag/javascript/
[2]: /news/author/joeattardi/
[3]: #Promise是什么
[4]: #Promises-与其他异步模式的比较
[5]: #如何创建一个-Promiose
[6]: #如何获取-Promise-的结果
[7]: #如何使用-then-处理错误
[8]: #Promise-链接
[9]: #how-to-create-immediately-fulfilled-or-rejected-promises
[10]: #how-to-use-async-and-await
[11]: #promise-anti-patterns
[12]: #summary
[13]: /news/author/joeattardi/
[14]: /news/author/joeattardi/
[15]: https://www.freecodecamp.org/learn/
