# JavaScript Cheatsheet — Quick Learning & Revision

Goal: concise explanations, runnable code snippets, best practices, and real-world use cases.

---

## Table of contents
- Basics & setup
- Syntax & types
- Operators & control flow
- Functions, closures & scope
- Objects, prototypes & classes
- Modules
- Async: promises, async/await, event loop
- DOM & browser APIs
- Patterns & best practices
- Performance & debugging
- Security
- Tooling & testing
- Real-world examples / use cases

---

## Basics & setup
- Run JS: node file.js (backend), <script> in browser (frontend).
- Use strict mode: "use strict"; helps catch errors.
- Package manager: npm / yarn / pnpm.

Example: package.json minimal
```json
{
  "name": "app",
  "version": "1.0.0",
  "type": "module",
  "scripts": { "start": "node index.js", "test": "jest" }
}
```

---

## Syntax & primitive types
- Primitives: number, string, boolean, null, undefined, symbol, bigint
- Objects: objects, arrays, functions, dates, regex

Examples:
```js
const n = 42;          // number
const s = "hello";     // string
const b = true;        // boolean
const z = null;        // explicit no-value
let u;                 // undefined
const id = Symbol("id");
const big = 10n ** 20n;
```

Type conversion:
```js
String(123); Number("123"); Boolean(""); + "123";
```

Best practice: prefer const, then let, avoid var.

---

## Operators & control flow
- Arithmetic, comparison (===), logical, nullish (??), optional chaining (?.), ternary
- Use strict equality (===) unless intentionally coercing.

Control flow examples:
```js
if (a === 1) { ... }
for (const x of arr) { ... }
switch (val) { case 1: ...; break; }
```

---

## Functions, scope, closures
- Declarations, expressions, arrow functions.
- Arrow functions don't bind their own this.
- Closures: functions that capture outer variables.

Examples:
```js
function add(a, b = 1) { return a + b; }
const sum = (...nums) => nums.reduce((s,n) => s + n, 0);

function counter() {
  let count = 0;
  return () => ++count; // closure: private state
}
const c = counter();
c(); // 1
```

Best practice: keep small pure functions; avoid large side-effectful functions.

---

## Hoisting, temporal dead zone, this
- var hoisted and initialized to undefined. let/const are hoisted but in TDZ until declaration.
- this depends on call site: method -> object, simple function -> undefined (strict) or global (non-strict), arrow -> lexical this.

Example:
```js
// TDZ
// console.log(x); // ReferenceError
let x = 1;

// this vs arrow
const obj = { val: 1, getVal() { return this.val; }, arrow: () => this.val };
```

---

## Objects, prototypes & classes
- Objects: literal, dynamic keys, getters/setters.
- Prototypes: inheritance model before classes.
- Classes: syntactic sugar over prototypes.

Examples:
```js
const o = { a:1, ['b' + 1]: 2, get total(){ return this.a + this.b; } };

// prototype
function Person(name){ this.name = name; }
Person.prototype.greet = function(){ return `Hi ${this.name}`; };

// class
class PersonC {
  constructor(name){ this.name = name; }
  greet(){ return `Hi ${this.name}`; }
}
```

Best practice: prefer classes for OOP readability; prefer composition over deep inheritance.

---

## Arrays — common methods
- map, filter, reduce, find, some, every, flatMap
- avoid mutating methods in functional code (splice, sort). Prefer immutable patterns.

Examples:
```js
const arr = [1,2,3];
const doubled = arr.map(n => n * 2);
const evens = arr.filter(n => n % 2 === 0);
const sum = arr.reduce((s,n)=>s+n,0);
```

---

## Modules (ESM & CommonJS)
- ESM: import/export (recommended for modern projects)
  - export const x = 1; export default fn;
  - import {x} from './mod.js'; import def from './mod.js';
- CommonJS (Node): module.exports / require()

Best practice: use ESM ("type": "module") in new projects.

---

## Promises & async/await
- Promises: encapsulate async work.
- Use async/await for readable code; handle errors with try/catch.
- Concurrency: Promise.all, allSettled, race.

Examples:
```js
// fetch JSON
async function getData(url){
  const r = await fetch(url);
  if (!r.ok) throw new Error('Network error');
  return await r.json();
}

// concurrency
const [a,b] = await Promise.all([getData('/a'), getData('/b')]);
```

Best practice: prefer async/await; use timeout/retry patterns for network calls.

---

## Event loop — microtasks & macrotasks (brief)
- Call stack -> Web APIs/timers -> task queue (macrotask)
- microtasks (Promises .then, queueMicrotask) run before next macrotask.
- Understanding ordering helps avoid UI jank and unexpected ordering.

Example:
```js
console.log('start');
Promise.resolve().then(()=>console.log('micro'));
setTimeout(()=>console.log('macro'),0);
console.log('end');
// order: start, end, micro, macro
```

---

## Error handling
- throw new Error('msg'); try/catch/finally
- Add context to errors; avoid swallowing errors silently.

---

## DOM & Browser APIs
- Query: document.querySelector, querySelectorAll
- Create/insert: document.createElement, append, insertBefore
- Events: addEventListener, removeEventListener, event delegation
- Forms: FormData, input validation
- Storage: localStorage, sessionStorage, IndexedDB (for large/structured data)
- Fetch API for network requests; use AbortController to cancel.

Examples:
```js
// event delegation
document.querySelector('#list').addEventListener('click', e => {
  const item = e.target.closest('.item');
  if (!item) return;
  // handle item click
});

// fetch with timeout
const controller = new AbortController();
setTimeout(()=>controller.abort(), 5000);
fetch('/api', { signal: controller.signal });
```

Best practice: avoid heavy DOM ops in loops, batch updates, use requestAnimationFrame for animations.

---

## Useful patterns & snippets

Debounce:
```js
function debounce(fn, wait=200){
  let t;
  return (...args) => {
    clearTimeout(t);
    t = setTimeout(()=>fn(...args), wait);
  };
}
```

Throttle:
```js
function throttle(fn, limit=200){
  let last = 0;
  return (...args) => {
    const now = Date.now();
    if (now - last >= limit) { last = now; fn(...args); }
  };
}
```

Memoize:
```js
function memoize(fn){
  const cache = new Map();
  return (...args) => {
    const key = JSON.stringify(args);
    if (cache.has(key)) return cache.get(key);
    const res = fn(...args);
    cache.set(key, res);
    return res;
  };
}
```

Deep clone:
- Prefer structuredClone(obj) in modern environments; fallback to libraries (lodash.cloneDeep) when necessary.
```js
const copy = structuredClone(obj);
```

Event delegation:
```js
document.body.addEventListener('click', e => {
  if (e.target.matches('.btn')) { /* handle */ }
});
```

Abortable fetch:
```js
const ctrl = new AbortController();
fetch(url, { signal: ctrl.signal }).catch(err => { if (err.name === 'AbortError') /* cancelled */ });
ctrl.abort();
```

---

## Real-world examples & use cases

1) API fetch + render list (with loading & error)
```js
async function renderList(container){
  const el = document.querySelector(container);
  try {
    el.textContent = 'Loading...';
    const data = await getData('/posts');
    el.innerHTML = data.map(d => `<li>${d.title}</li>`).join('');
  } catch(e) {
    el.textContent = 'Failed to load';
    console.error(e);
  }
}
```

Use case: typical single-page app data fetching.

2) Search input with debounce (improves UX & reduces requests)
```js
const search = document.querySelector('#q');
search.addEventListener('input', debounce(async (e) => {
  const q = e.target.value;
  if (!q) return;
  const res = await fetch(`/search?q=${encodeURIComponent(q)}`);
  // handle results
}, 300));
```

3) Infinite scroll with throttling
- Use IntersectionObserver for modern approach (better than scroll events).

4) Optimistic UI (immediate UI update then roll back on failure)
```js
async function toggleLike(itemId){
  const prev = item.liked;
  item.liked = !prev; // optimistic update
  try {
    await api.patch(`/items/${itemId}/like`, { liked: item.liked });
  } catch(e){
    item.liked = prev; // rollback
  }
}
```

5) Token refresh with axios interceptor (real-world auth flows)
- Keep token refresh logic central to avoid race conditions.

---

## Performance & memory
- Avoid unnecessary re-renders and DOM thrashing.
- Use web workers for heavy CPU tasks.
- Profile with browser DevTools: CPU, memory, network.
- Avoid creating large intermediate arrays; stream/iterate when possible.

---

## Security (frontend)
- Prevent XSS: escape user content, use safe innerText/textContent, not innerHTML.
- Use Content Security Policy (CSP).
- Sanitize inputs and server-side validation.
- Protect tokens: keep sensitive tokens out of localStorage when possible; consider httpOnly cookies.

---

## Testing, tooling & ecosystem
- Testing: Jest, Mocha, Cypress (E2E)
- Bundlers: webpack, rollup, parcel, vite
- Linting & formatting: ESLint, Prettier
- Types: consider TypeScript for large projects for better maintainability.
- CI: run tests, lint, build on push.

---

## Debugging tips
- console.log, console.table, console.error, debugger
- Source maps for transpiled code
- Breakpoints and step-through in DevTools
- Use smaller reproducible cases for bugs

---

## Quick reference cheats (one-liners)
- Clone array: const copy = [...arr];
- Merge objects: const merged = {...a, ...b};
- Optional chaining: const x = obj?.a?.b;
- Nullish coalescing: const v = val ?? defaultVal;
- Safe parse: const data = JSON.parse(str || '{}');

---

## Further reading / next steps
- Event loop & concurrency details
- Memory management & garbage collection
- Deep dive into prototype chain
- Advanced patterns: reactive programming (RxJS), state management (Redux, Zustand)

---

Keep this file as your single-page quick reference. Expand real-world snippets as you encounter patterns in projects.
