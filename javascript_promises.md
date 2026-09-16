# Technical Paper 2: Promises and Promise Chaining

## 1. What is a Promise?

A **Promise** is an object that represents the eventual result of an
asynchronous operation.

A promise can be:

-   **Pending** --- operation is still running.
-   **Fulfilled** --- operation completed successfully.
-   **Rejected** --- operation failed.

``` text
Pending
  ├──→ Fulfilled
  └──→ Rejected
```

A settled promise cannot change to another state.

------------------------------------------------------------------------

## 2. Creating a Promise

Use the `Promise` constructor:

``` js
const promise = new Promise((resolve, reject) => {
  const success = true;

  if (success) {
    resolve("Success");
  } else {
    reject("Something went wrong");
  }
});
```

`resolve()` means success.

`reject()` means failure.

Example with asynchronous work:

``` js
const promise = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve("Data received");
  }, 1000);
});
```

------------------------------------------------------------------------

## 3. Promise States

### Pending

``` js
const promise = new Promise(() => {});
```

It remains pending because neither `resolve` nor `reject` is called.

### Fulfilled

``` js
resolve("Success");
```

### Rejected

``` js
reject("Failed");
```

------------------------------------------------------------------------

## 4. Consuming an Existing Promise

Use `.then()` for success:

``` js
promise.then((result) => {
  console.log(result);
});
```

Use `.catch()` for errors:

``` js
promise.catch((error) => {
  console.error(error);
});
```

Use `.finally()` for code that should run after settlement:

``` js
promise.finally(() => {
  console.log("Operation finished");
});
```

------------------------------------------------------------------------

# Promise Chaining

## 5. Chaining with `.then()`

One `.then()` can return another value or promise.

``` js
Promise.resolve(10)
  .then((value) => {
    return value * 2;
  })
  .then((value) => {
    console.log(value);
  });
```

Output:

``` text
20
```

Each `.then()` receives the returned value from the previous `.then()`.

------------------------------------------------------------------------

## 6. Returning a Promise from `.then()`

``` js
getUser()
  .then((user) => {
    return getOrders(user);
  })
  .then((orders) => {
    console.log(orders);
  });
```

The second `.then()` waits for the promise returned by the first
`.then()`.

------------------------------------------------------------------------

## 7. Error Handling with `.catch()`

``` js
getUser()
  .then((user) => {
    return getOrders(user);
  })
  .then((orders) => {
    console.log(orders);
  })
  .catch((error) => {
    console.error(error);
  });
```

A rejection travels down the promise chain until a rejection handler
handles it.

------------------------------------------------------------------------

## 8. Error thrown inside `.then()`

``` js
Promise.resolve("Hello")
  .then((value) => {
    throw new Error("Something failed");
  })
  .catch((error) => {
    console.log(error.message);
  });
```

Output:

``` text
Something failed
```

A thrown error inside `.then()` turns the resulting promise into a
rejected promise.

------------------------------------------------------------------------

## 9. What if there is no `.catch()`?

``` js
Promise.resolve()
  .then(() => {
    throw new Error("Failed");
  });
```

There is no rejection handler in this chain.

The promise remains rejected and the runtime may report an **unhandled
promise rejection**.

Therefore, asynchronous errors should be handled deliberately.

------------------------------------------------------------------------

## 10. Why is `.catch()` usually placed toward the end?

Example:

``` js
getUser()
  .then((user) => getOrders(user))
  .then((orders) => processOrders(orders))
  .catch((error) => {
    console.error(error);
  });
```

Placing one catch near the end can handle failures from earlier parts of
the chain.

However, a `.catch()` can also be placed earlier when you intentionally
want to recover from a particular operation.

So `.catch()` does not have to be at the end; putting it near the end is
a common pattern for handling errors from the whole chain.

------------------------------------------------------------------------

## 11. Consuming multiple promises by chaining

Suppose the second operation needs the result of the first:

``` js
getUser()
  .then((user) => {
    return getOrders(user.id);
  })
  .then((orders) => {
    return getPayment(orders[0].id);
  })
  .then((payment) => {
    console.log(payment);
  })
  .catch((error) => {
    console.error(error);
  });
```

The operations happen in sequence.

------------------------------------------------------------------------

# Promise Utility Methods

## 12. `Promise.resolve()`

Creates an already fulfilled promise.

``` js
Promise.resolve("Hello")
  .then((value) => console.log(value));
```

Output:

``` text
Hello
```

------------------------------------------------------------------------

## 13. `Promise.reject()`

Creates an already rejected promise.

``` js
Promise.reject(new Error("Failed"))
  .catch((error) => {
    console.log(error.message);
  });
```

------------------------------------------------------------------------

## 14. `Promise.all()`

Use when multiple promises must succeed.

``` js
const p1 = Promise.resolve("A");
const p2 = Promise.resolve("B");
const p3 = Promise.resolve("C");

Promise.all([p1, p2, p3])
  .then((results) => {
    console.log(results);
  });
```

Output:

``` text
["A", "B", "C"]
```

If one promise rejects, `Promise.all()` rejects.

``` js
Promise.all([p1, Promise.reject("Failed"), p3])
  .catch((error) => console.log(error));
```

Use it when **all results are required**.

------------------------------------------------------------------------

## 15. `Promise.allSettled()`

Waits for all promises, whether they succeed or fail.

``` js
Promise.allSettled([
  Promise.resolve("A"),
  Promise.reject("Failed")
]).then((results) => {
  console.log(results);
});
```

It provides the status of each promise.

Useful when you want the outcome of every operation.

------------------------------------------------------------------------

## 16. `Promise.any()`

Returns when the **first promise fulfills**.

``` js
Promise.any([
  Promise.reject("Failed"),
  Promise.resolve("Success"),
  Promise.resolve("Another")
]).then((result) => {
  console.log(result);
});
```

Output:

``` text
Success
```

If every promise rejects, `Promise.any()` rejects with an
`AggregateError`.

------------------------------------------------------------------------

## 17. `Promise.race()`

Settles when the **first promise settles**, whether fulfilled or
rejected.

``` js
const p1 = new Promise((resolve) => {
  setTimeout(() => resolve("First"), 1000);
});

const p2 = new Promise((resolve) => {
  setTimeout(() => resolve("Second"), 2000);
});

Promise.race([p1, p2])
  .then((result) => console.log(result));
```

Output:

``` text
First
```

------------------------------------------------------------------------

## Quick Comparison

  ------------------------------------------------------------------------
  Method                   Finishes when           If one rejects
  ------------------------ ----------------------- -----------------------
  `Promise.all()`          All fulfill             Rejects

  `Promise.allSettled()`   All settle              Does not reject because
                                                   of member rejection

  `Promise.any()`          First fulfills          Rejects only if all
                                                   reject

  `Promise.race()`         First settles           Result depends on first
                                                   settled promise
  ------------------------------------------------------------------------

------------------------------------------------------------------------

## 18. Error Handling with Promises

A common pattern:

``` js
fetch("/users")
  .then((response) => {
    if (!response.ok) {
      throw new Error("Request failed");
    }

    return response.json();
  })
  .then((users) => {
    console.log(users);
  })
  .catch((error) => {
    console.error(error);
  })
  .finally(() => {
    console.log("Request finished");
  });
```

Error handling is important because asynchronous operations can fail due
to:

-   Network errors
-   Invalid data
-   Server errors
-   File errors
-   Exceptions in callback/`.then()` code

Without handling failures, applications can behave unpredictably or
produce unhandled rejections.

------------------------------------------------------------------------

## Key idea

``` text
Promise
   ↓
pending
   ↓
fulfilled / rejected
   ↓
.then() / .catch() / .finally()
```

Promises provide a structured way to represent and handle asynchronous
results.
