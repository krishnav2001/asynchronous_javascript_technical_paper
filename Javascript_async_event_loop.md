# Technical Paper 1: JavaScript Execution, Sync/Async, Web APIs and Event Loop

## 1. How does JavaScript execute code?

JavaScript is generally **single-threaded**, meaning JavaScript code is
executed by one main thread at a time.

The JavaScript runtime has important parts:

-   **Call Stack** --- keeps track of currently executing functions.
-   **Web APIs** --- browser-provided features that can perform
    asynchronous work.
-   **Task/Callback Queue** --- stores callbacks that are ready to run.
-   **Microtask Queue** --- stores promise callbacks such as `.then()`
    and `.catch()`.
-   **Event Loop** --- moves ready callbacks to the call stack when the
    stack is empty.

Example:

``` js
console.log("A");

function greet() {
  console.log("B");
}

greet();

console.log("C");
```

Output:

``` text
A
B
C
```

JavaScript executes statements in order. A function call is pushed onto
the call stack, executed, and then removed.

### Call Stack

For:

``` js
function one() {
  two();
}

function two() {
  console.log("Hello");
}

one();
```

The stack roughly works like:

``` text
one()
  ↓
two()
  ↓
console.log()
```

After each function finishes, it is removed from the stack.

------------------------------------------------------------------------

## 2. Synchronous vs Asynchronous

### Synchronous

Synchronous code runs **one operation at a time** and waits for the
current operation to finish.

``` js
console.log("A");
console.log("B");
console.log("C");
```

Output:

``` text
A
B
C
```

### Asynchronous

Asynchronous code allows an operation to be started without blocking the
rest of the JavaScript code.

``` js
console.log("A");

setTimeout(() => {
  console.log("B");
}, 2000);

console.log("C");
```

Output:

``` text
A
C
B
```

The timer does not block the execution of `console.log("C")`.

### Difference

  -----------------------------------------------------------------------
  Synchronous                         Asynchronous
  ----------------------------------- -----------------------------------
  Executes step by step               Allows work to complete later

  Blocks the current flow             Can avoid blocking the main flow

  Result is available immediately in  Result is handled later
  the normal flow                     

  Simple to understand                Useful for I/O and waiting
  -----------------------------------------------------------------------

------------------------------------------------------------------------

## 3. Ways to make code asynchronous

Common mechanisms include:

### `setTimeout`

``` js
setTimeout(() => {
  console.log("Done");
}, 1000);
```

### Browser APIs

Examples include:

``` js
fetch("/users");
```

and:

``` js
setInterval(() => {
  console.log("Running");
}, 1000);
```

### Callbacks

``` js
function getData(callback) {
  setTimeout(() => {
    callback("Data received");
  }, 1000);
}

getData((data) => {
  console.log(data);
});
```

### Promises

``` js
fetch("/users")
  .then((response) => response.json())
  .then((data) => console.log(data));
```

### `async/await`

``` js
async function getUsers() {
  const response = await fetch("/users");
  const data = await response.json();
  console.log(data);
}
```

`async/await` is built on top of promises.

------------------------------------------------------------------------

## 4. What are Web Browser APIs?

Web APIs are features provided by the browser environment, not by the
JavaScript language itself.

Examples:

-   `setTimeout`
-   `fetch`
-   DOM APIs
-   `localStorage`
-   `setInterval`
-   Geolocation
-   Event handling

Example:

``` js
setTimeout(() => {
  console.log("Hello");
}, 1000);
```

The timer is handled by the browser environment. JavaScript does not sit
on the call stack waiting for one second.

Another example:

``` js
fetch("/users");
```

The browser handles the network operation and later makes the result
available to JavaScript.

------------------------------------------------------------------------

## 5. What is the Event Loop?

The **event loop** coordinates asynchronous callbacks and JavaScript
execution.

Consider:

``` js
console.log("Start");

setTimeout(() => {
  console.log("Timer");
}, 0);

console.log("End");
```

Output:

``` text
Start
End
Timer
```

Why?

1.  `"Start"` executes.
2.  `setTimeout` is registered with the runtime.
3.  `"End"` executes.
4.  The timer callback becomes ready.
5.  When the call stack is empty, the event loop allows the callback to
    execute.

### Simplified model

``` text
JavaScript
    ↓
Call Stack
    ↓
Event Loop
    ↓
Queues
    ↓
Callback execution
```

### Important: Microtasks

Promise callbacks use the **microtask queue**.

``` js
console.log("A");

setTimeout(() => {
  console.log("B");
}, 0);

Promise.resolve().then(() => {
  console.log("C");
});

console.log("D");
```

Output:

``` text
A
D
C
B
```

Promise microtasks are processed before timer/task callbacks after the
current synchronous code finishes.

------------------------------------------------------------------------

## 6. Callback Hell

Callback hell happens when many asynchronous operations depend on each
other and callbacks become deeply nested.

``` js
getUser((user) => {
  getOrders(user, (orders) => {
    getPayment(orders, (payment) => {
      getAddress(payment, (address) => {
        console.log(address);
      });
    });
  });
});
```

Problems:

-   Difficult to read
-   Difficult to maintain
-   Error handling becomes repetitive
-   Hard to understand the flow

Promises help flatten this structure.

------------------------------------------------------------------------

## 7. Inversion of Control in Callbacks

When you pass a callback to another function, you give that function
control over **when and how your callback is executed**.

``` js
function processData(callback) {
  // process something
  callback();
}

processData(() => {
  console.log("Done");
});
```

You do not directly control when `callback()` is called. `processData`
does.

This is called **inversion of control**.

Potential problems:

-   Callback might never be called.
-   It might be called multiple times.
-   It might be called with unexpected arguments.
-   Error handling can become difficult.

Promises provide a more controlled way to represent one eventual result.
