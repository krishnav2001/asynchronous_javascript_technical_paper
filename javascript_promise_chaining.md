# Technical Paper 3: Promise Chaining

## 1. What is Promisification?

**Promisification** means converting a callback-based asynchronous
function into a function that returns a Promise.

Callback-based function:

``` js
function getData(callback) {
  setTimeout(() => {
    callback(null, "Data");
  }, 1000);
}
```

Promisified version:

``` js
function getData() {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve("Data");
    }, 1000);
  });
}
```

Now it can be consumed with:

``` js
getData()
  .then((data) => console.log(data))
  .catch((error) => console.error(error));
```

------------------------------------------------------------------------

# 2. Promisifying `setTimeout`

`setTimeout` itself does not return a promise.

We can create one:

``` js
function delay(ms) {
  return new Promise((resolve) => {
    setTimeout(resolve, ms);
  });
}
```

Usage:

``` js
delay(1000)
  .then(() => {
    console.log("One second completed");
  });
```

With `async/await`:

``` js
async function run() {
  await delay(1000);
  console.log("Done");
}

run();
```

------------------------------------------------------------------------

# 3. Promisifying `fs.readFile`

Node.js provides callback-based APIs such as:

``` js
const fs = require("fs");

fs.readFile("data.txt", "utf8", (error, data) => {
  if (error) {
    console.error(error);
    return;
  }

  console.log(data);
});
```

A promise-based version can be created manually:

``` js
const fs = require("fs");

function readFilePromise(file) {
  return new Promise((resolve, reject) => {
    fs.readFile(file, "utf8", (error, data) => {
      if (error) {
        reject(error);
        return;
      }

      resolve(data);
    });
  });
}
```

Usage:

``` js
readFilePromise("data.txt")
  .then((data) => console.log(data))
  .catch((error) => console.error(error));
```

In modern Node.js, promise-based APIs are also available directly:

``` js
const fs = require("fs/promises");

fs.readFile("data.txt", "utf8")
  .then((data) => console.log(data))
  .catch((error) => console.error(error));
```

------------------------------------------------------------------------

# 4. Sequential vs Parallel Promises

## Sequential

Use chaining when the next operation depends on the previous result.

``` js
getUser()
  .then((user) => getOrders(user.id))
  .then((orders) => getPayment(orders[0].id))
  .then((payment) => console.log(payment));
```

Flow:

``` text
getUser
   ↓
getOrders
   ↓
getPayment
```

## Parallel

If operations are independent, they can be started together.

``` js
const users = getUsers();
const products = getProducts();

Promise.all([users, products])
  .then(([users, products]) => {
    console.log(users);
    console.log(products);
  });
```

This can avoid unnecessary waiting.

------------------------------------------------------------------------

# 5. `Promise.resolve()`

Use it to create a fulfilled promise.

``` js
const promise = Promise.resolve(100);

promise.then((value) => {
  console.log(value);
});
```

Output:

``` text
100
```

It is also useful when a function may return either a normal value or a
promise:

``` js
Promise.resolve(getData())
  .then((data) => console.log(data));
```

------------------------------------------------------------------------

# 6. `Promise.reject()`

Creates a rejected promise.

``` js
const promise = Promise.reject(new Error("Invalid data"));

promise.catch((error) => {
  console.log(error.message);
});
```

Output:

``` text
Invalid data
```

------------------------------------------------------------------------

# 7. `Promise.all()`

Use when **every operation is required**.

``` js
const userPromise = getUser();
const ordersPromise = getOrders();
const productsPromise = getProducts();

Promise.all([
  userPromise,
  ordersPromise,
  productsPromise
])
  .then(([user, orders, products]) => {
    console.log(user);
    console.log(orders);
    console.log(products);
  })
  .catch((error) => {
    console.error(error);
  });
```

Important:

-   Results keep the same order as the input promises.
-   It rejects if any input promise rejects.

------------------------------------------------------------------------

# 8. `Promise.allSettled()`

Use when you want the result of **every operation**, even when some
fail.

``` js
Promise.allSettled([
  getUsers(),
  getProducts(),
  getOrders()
]).then((results) => {
  console.log(results);
});
```

Possible result structure:

``` js
[
  { status: "fulfilled", value: "users" },
  { status: "rejected", reason: "Server error" },
  { status: "fulfilled", value: "orders" }
]
```

This is useful for dashboards where one failed request should not hide
the results of other requests.

------------------------------------------------------------------------

# 9. `Promise.any()`

Use when you need **the first successful result**.

``` js
Promise.any([
  fetchFromServer1(),
  fetchFromServer2(),
  fetchFromServer3()
])
  .then((result) => {
    console.log(result);
  })
  .catch((error) => {
    console.log(error);
  });
```

If server 1 fails but server 2 succeeds, the result from server 2 can be
used.

If all promises reject, it rejects with `AggregateError`.

------------------------------------------------------------------------

# 10. `Promise.race()`

Use when you care about whichever promise settles first.

``` js
Promise.race([
  fetchData(),
  delay(5000).then(() => {
    throw new Error("Timeout");
  })
])
  .then((result) => console.log(result))
  .catch((error) => console.error(error));
```

This pattern can be used to implement a timeout around an asynchronous
operation.

------------------------------------------------------------------------

# 11. `finally()`

`finally()` runs after a promise settles.

``` js
fetch("/users")
  .then((response) => response.json())
  .catch((error) => {
    console.error(error);
  })
  .finally(() => {
    console.log("Loading finished");
  });
```

It is useful for cleanup:

-   Hide loading indicators
-   Close resources
-   Reset UI state
-   Stop timers

The `finally()` callback normally does not receive the fulfilled value
or rejection reason.

------------------------------------------------------------------------

# 12. Error Handling Rules

### Handle expected failures

``` js
getData()
  .then((data) => console.log(data))
  .catch((error) => console.error(error));
```

### Throw meaningful errors

``` js
.then((response) => {
  if (!response.ok) {
    throw new Error("Request failed");
  }

  return response.json();
})
```

### Do not silently ignore errors

Avoid:

``` js
.catch(() => {});
```

unless intentionally ignoring the error.

### Recover when appropriate

``` js
getData()
  .catch(() => {
    return "Default data";
  })
  .then((data) => {
    console.log(data);
  });
```

Here, the catch converts the failure into a successful value.

------------------------------------------------------------------------

# 13. Common Promise Mistake: Forgetting `return`

Incorrect:

``` js
getUser()
  .then((user) => {
    getOrders(user.id);
  })
  .then((orders) => {
    console.log(orders);
  });
```

The first `.then()` does not return the `getOrders()` promise.

Correct:

``` js
getUser()
  .then((user) => {
    return getOrders(user.id);
  })
  .then((orders) => {
    console.log(orders);
  });
```

Or with an arrow function:

``` js
getUser()
  .then((user) => getOrders(user.id))
  .then((orders) => {
    console.log(orders);
  });
```

------------------------------------------------------------------------

# 14. Promise Chaining Mental Model

Remember:

``` js
promise
  .then(...)
  .then(...)
  .catch(...)
  .finally(...);
```

Think of it as:

``` text
Start
  ↓
.then()
  ↓
.then()
  ↓
.catch() if something fails
  ↓
.finally()
```

Each `.then()` returns a **new promise**.

------------------------------------------------------------------------

# 15. Quick Decision Guide

  Requirement                               Use
  ----------------------------------------- ---------------------------
  One async result                          `.then()` / `async-await`
  Sequential dependent operations           Promise chaining
  All independent operations must succeed   `Promise.all()`
  Need every result, including failures     `Promise.allSettled()`
  First successful result                   `Promise.any()`
  First settled result                      `Promise.race()`
  Always perform cleanup                    `.finally()`
  Convert callback API to Promise API       Promisification
  Create successful promise                 `Promise.resolve()`
  Create failed promise                     `Promise.reject()`

------------------------------------------------------------------------

# Final Example

``` js
function delay(ms) {
  return new Promise((resolve) => {
    setTimeout(resolve, ms);
  });
}

async function main() {
  try {
    const results = await Promise.all([
      delay(1000).then(() => "User"),
      delay(500).then(() => "Products")
    ]);

    console.log(results);
  } catch (error) {
    console.error(error);
  } finally {
    console.log("Finished");
  }
}

main();
```

The important concepts are:

``` text
Callbacks
   ↓
Callback Hell / Inversion of Control
   ↓
Promises
   ↓
Promise Chaining
   ↓
Promise Utility Methods
   ↓
async/await
```

Understanding this flow makes asynchronous JavaScript much easier to
reason about.
