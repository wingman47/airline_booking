# JavaScript

## falsy values
false
0 and -0
"" (empty string)
null
undefined
NaN
0n (BigInt zero)

## hoisting
means variable and function declarations are moved to the top of their scope before code execution.
```js
console.log(a); // ❌ ReferenceError
let a = 10;
```
var declarations are hoisted and initialized with undefined, leading to potential bugs. 
let and const are also hoisted, but they are not initialized, existing in a "Temporal Dead Zone" until their declaration is reached.

Function declarations are fully hoisted and can be called before their definition in the code. Function expressions are not hoisted; only the variable storing the function is hoisted (with a value of undefined).

## var vs let
var is function-scoped, while let is block-scoped.
```js
function testScope() {
  if (true) {
    var x = "I am var";
    let y = "I am let";
  }

  console.log(x); // works, because var is function-scoped
  console.log(y); // ❌ ReferenceError, because let is block-scoped
}
testScope();
```

## == vs ===
== checks values after type conversion
=== checks values without type conversion
console.log(5 == '5'); (true) vs. console.log(5 === '5'); (false)

## default and rest params
```js
function intro(greeting = "Hi", ...names) {
  console.log(greeting, names);
}

intro();                 // Hi []
intro("Hello", "A", "B"); // Hello [ 'A', 'B' ]
```

## high order functions -  implement
Here’s a quick breakdown of the four commonly used array methods in JavaScript:

### `map()`
* Transforms each element and returns a **new array**.

```js
let nums = [1, 2, 3];
let doubled = nums.map(n => n * 2);
console.log(doubled); // [2, 4, 6]
```

### `filter()`
* Returns a **new array** with elements that pass the condition.

```js
let nums = [1, 2, 3, 4];
let evens = nums.filter(n => n % 2 === 0);
console.log(evens); // [2, 4]
```

### `reduce()`
* Reduces array to a **single value** (sum, product, etc).

```js
let nums = [1, 2, 3, 4];
let sum = nums.reduce((acc, n) => acc + n, 0);
console.log(sum); // 10
```

### `forEach()`
* Loops through array elements but **doesn’t return a new array**.

```js
let nums = [1, 2, 3];
nums.forEach(n => console.log(n * 2));
// 2, 4, 6
```

👉 In short:

* `map` → transform
* `filter` → select
* `reduce` → accumulate
* `forEach` → iterate

### `spread` operator
```js
const numbers = [1, 2];
const newNumbers = [...numbers, 3, 4]; // [1, 2, 3, 4]
const frontNumbers = [0, ...numbers]; // [0, 1, 2]

const base = { a: 1, b: 2 };
const updated = { ...base, c: 3 }; // { a: 1, b: 2, c: 3 }
const overriden = { ...base, b: 5 }; // { a: 1, b: 5 }
```

### `rest` operator
```js
const [x, ...rest] = [1, 2, 3, 4]; // x=1, rest=[2,3,4]
```

## closures
when an inner function is returned from an outer function they maintain its lexical scope therefore the count variable is stored.
so, a function bundled together with its lexical scope forms a closure.
```js
function counter() {
  let count = 0;
  return function () {
    count++;
    return count;
  };
}

let add = counter();
console.log(add()); // 1
console.log(add()); // 2
console.log(add()); // 3
```

### memory leak due to closure
```js
function createListener() {
  let bigData = new Array(1000000).fill("leak"); // large memory
  document.body.addEventListener("click", () => {
    console.log(bigData[0]); // closure holds bigData
  });
}
createListener(); // bigData is never freed!
```

## event loop
```js
console.log("start");
setTimeout(() => console.log("timeout"), 0);
Promise.resolve().then(() => console.log("promise"));
console.log("end");
```

Execution Flow:
"start" → printed immediately.
setTimeout(...) → goes to Web API → after 0ms, callback moves to Callback Queue.
Promise.then(...) → goes to Microtask Queue.
"end" → printed immediately.
Call Stack empty → Event Loop checks Microtask Queue → runs "promise".
Then takes "timeout" from Callback Queue.

In Short -

JS runs all synchronous code first.
Then Event Loop gives priority to Microtasks (Promises).
Finally, it runs Macrotasks (setTimeout, events).

## callback hell
Callback hell happens when you have many nested callbacks in asynchronous code, making it look messy, hard to read, and harder to maintain.
```js
getData1(result1 => {
  getData2(result1, result2 => {
    getData3(result2, result3 => {
      getData4(result3, result4 => {
        console.log("Final result:", result4);
      });
    });
  });
});
```

## promise
same code using promise
```js
getData1()
  .then(result1 => getData2(result1))
  .then(result2 => getData3(result2))
  .then(result3 => {
    console.log("Final result:", result3);
  })
  .catch(error => {
    console.error(error);
  });
```

## async await
```js
async function fetchData() {
  try {
    const result1 = await getData1();
    const result2 = await getData2(result1);
    const result3 = await getData3(result2);
    console.log("Final result:", result3);
  } catch (error) {
    console.error(error);
  }
}

fetchData();
```

# ReactJS

## virtual DOM
DOM → real browser structure, direct updates, can be slow.
Virtual DOM → JS copy, diffs changes, updates only what’s needed, faster for complex UIs.

The **Virtual DOM** in frameworks like React works like this:

1. JS creates a **Virtual DOM** (a lightweight copy of the real DOM).
2. When state or props change, a **new Virtual DOM tree** is created.
3. React **diffs** the new tree with the previous Virtual DOM tree to see what actually changed.
4. Only the **minimal set of real DOM updates** are applied.

This is called **reconciliation** and makes updates much faster than re-rendering the entire real DOM.

## useEffect
A component lifecycle consists of:
mount -> update -> unmount (loop) (unmount happens when the component is removed from the DOM)
and useEffect is used to run the code in these 3 states of a component lifecycle.
```js
import { useState, useEffect } from "react";

function Example() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    // ✅ Mount: runs once when component mounts
    console.log("Component Mounted");

    // ✅ Return function → Unmount: runs when component unmounts
    return () => {
      console.log("Component Unmounted");
    };
  }, []); // empty dependency → mount only

  useEffect(() => {
    // ✅ Update: runs every time `count` changes + the initial render
    if (count > 0) {
      console.log("Component Updated, count:", count);
    }
  }, [count]); // dependency array → watch `count`

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
}

export default Example;
```

## useRef
useRef creates a mutable reference that persists across renders.
Changing a ref does not trigger a re-render.
Often used for:
- Accessing DOM elements directly.
- Storing any mutable value that survives renders.

```js
import { useRef } from "react";

function Example() {
  const inputRef = useRef(null);

  const changeWidth = () => {
    inputRef.current.style.width = "300px"; // directly update DOM
  };

  return (
    <div>
      <input ref={inputRef} placeholder="Type here" />
      <button onClick={changeWidth}>Increase Width</button>
    </div>
  );
}

export default Example;
```
useRef → for direct DOM manipulation or storing values without triggering render.
useState → for reactive state that needs to trigger UI updates.

Here’s a concise guide to **Conditional Rendering & Lists in React**:

---

## 🔹 1. Rendering Lists using `map()`

* Use `Array.map()` to render multiple components dynamically.
* Always provide a **unique `key` prop** for each item to help React track elements.

```jsx
function FruitsList() {
  const fruits = ["Apple", "Banana", "Cherry"];

  return (
    <ul>
      {fruits.map((fruit, index) => (
        <li key={index}>{fruit}</li> // key helps React identify items
      ))}
    </ul>
  );
}
```

**Best practice for `key`:**

* Use a **unique and stable identifier** (like `id`) instead of `index` if possible.

```jsx
{users.map(user => <li key={user.id}>{user.name}</li>)}
```

---

## 🔹 2. Conditional Rendering

* Render elements **conditionally** based on state or props.

### Using Ternary Operator

```jsx
const isLoggedIn = true;
return <div>{isLoggedIn ? "Welcome User" : "Please Login"}</div>;
```

### Using `&&` Operator

* Useful when you only want to render something **if condition is true**.

```jsx
const showMessage = true;
return <div>{showMessage && <p>Hello!</p>}</div>;
```

### Using `||` Operator

* Provide a **fallback** if value is falsy.

```jsx
const name = "";
return <div>{name || "Guest"}</div>; // Guest
```
## useMemo
useMemo memoizes the result of a function so that it only recomputes when dependencies change.
Helps optimize performance for expensive calculations that don’t need to run on every render.

```js
import { useMemo, useState } from "react";

function ExpensiveComponent({ num }) {
  // expensive calculation
  const factorial = useMemo(() => {
    console.log("Calculating factorial...");
    let result = 1;
    for (let i = 1; i <= num; i++) result *= i;
    return result;
  }, [num]); // only recompute when num changes

  return <p>Factorial of {num} is {factorial}</p>;
}

function App() {
  const [count, setCount] = useState(5);
  return (
    <div>
      <ExpensiveComponent num={count} />
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
}
```