# TypeScript Cheatsheet — Quick Learning & Revision

Goal: concise explanations, runnable snippets, best practices, and real-world use cases.

---

## Table of contents
- Why TypeScript
- Setup & tooling
- Basic types
- Type annotations & inference
- Interfaces vs type aliases
- Functions & overloads
- Union, intersection, literal types, discriminated unions
- Generics
- Utility types
- Advanced types (mapped, conditional, infer, keyof)
- Classes & decorators (brief)
- Modules & declaration files
- tsconfig & compiler options
- Type guards & narrowing
- DOM & frontend tips
- Best practices & migration tips
- Real-world examples / snippets

---

## Why TypeScript
- Static typing, tooling (editor autocompletion), earlier error detection, better refactoring.
- Incremental adoption: start with .ts files or add // @ts-check to JS.

---

## Setup & tooling
- Install: npm i -D typescript
- Init: npx tsc --init
- Useful tools: ts-node, ts-jest, eslint (with @typescript-eslint), vite/webpack.

Minimal tsconfig:
```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "ESNext",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "outDir": "dist"
  }
}
```

---

## Basic types
```ts
let n: number = 1;
let s: string = "hi";
let b: boolean = true;
let u: undefined = undefined;
let nl: null = null;
let anyVal: any = 123; // avoid when possible
let unknownVal: unknown = "x"; // safer than any
let sym: symbol = Symbol("id");
let big: bigint = 123n;
```

Type inference:
```ts
const x = 1; // inferred as number
```

---

## Type annotations & inference
- Prefer type inference; annotate function params/returns.
```ts
function add(a: number, b: number): number {
  return a + b;
}
```

---

## Arrays, tuples, readonly
```ts
const nums: number[] = [1,2,3];
const strs: Array<string> = ["a"];
const tup: [number, string] = [1, 'a'];
const ro: readonly number[] = [1,2];
```

---

## Interfaces vs type aliases
- interface: extendable, merges declarations.
- type: unions, primitives, mapped types.

Examples:
```ts
interface User { id: number; name: string; }
type Point = { x: number; y: number; }
type ID = number | string;
```

Extend/implement:
```ts
interface A { a: string; }
interface B extends A { b: number; }
class C implements B { a = 'x'; b = 1; }
```

---

## Union, intersection, literal types, discriminated unions
```ts
type Res = { ok: true; data: string } | { ok: false; error: string };
function handle(r: Res) {
  if (r.ok) { console.log(r.data); } else { console.error(r.error); }
}
```
Intersection:
```ts
type A = { a: number }; type B = { b: string };
type AB = A & B; // has both
```

Literal & narrowing:
```ts
type Dir = 'left' | 'right';
function move(d: Dir) { /* d is 'left' | 'right' */ }
```

---

## Generics
- Reusable, type-safe functions/types.
```ts
function identity<T>(v: T): T { return v; }
const id = identity<number>(1);

type ApiResponse<T> = { data: T; status: number };
async function fetchJson<T>(url: string): Promise<T> {
  const r = await fetch(url);
  return r.json() as Promise<T>;
}
```

Generic constraints:
```ts
function pluck<T, K extends keyof T>(obj: T, key: K) { return obj[key]; }
```

---

## Utility types (common)
- Partial<T>, Required<T>, Readonly<T>, Pick<T,K>, Omit<T,K>, Record<K,T>, Exclude/Extract, NonNullable<T>, ReturnType<F>, Parameters<F>

Examples:
```ts
type UserUpdate = Partial<User>;
type IDMap = Record<string, number>;
type OnlyName = Pick<User, 'name'>;
```

---

## Advanced types
- keyof: get property names
- mapped types: { [K in Keys]: ... }
- conditional types: T extends U ? X : Y
- infer: extract types in conditional types

Examples:
```ts
type Keys = keyof User; // 'id'|'name'
type ReadonlyUser = { readonly [K in keyof User]: User[K] };
type UnwrapPromise<T> = T extends Promise<infer U> ? U : T;
```

---

## Type guards & narrowing
- typeof, instanceof, in, user-defined type guards (x is T)
```ts
function isString(x: unknown): x is string { return typeof x === 'string'; }
if (isString(val)) { /* val typed as string */ }
```

---

## Classes & OOP
```ts
class Person {
  constructor(public name: string, private age: number) {}
  greet() { return `Hi ${this.name}`; }
}
```
- Access modifiers: public, private, protected, readonly
- use interfaces to type shapes; prefer composition.

---

## Declaration files & ambient types
- Use .d.ts to declare types for JS libs or global vars.
```ts
// global.d.ts
declare module 'some-lib';
```
- For DOM libs, install @types packages.

---

## Modules & imports
- Use ES module syntax and keep types separate.
```ts
// export
export interface Todo { id: string; title: string; done: boolean; }
// import
import type { Todo } from './types';
```
- import type only to avoid runtime import.

---

## tsconfig highlights
- strict: enables strictNullChecks, noImplicitAny, etc.
- noImplicitAny: prevent implicit any
- strictNullChecks: make null/undefined explicit
- allowJs / checkJs: gradual migration
- paths & baseUrl: configure module resolution

---

## DOM & frontend tips
- DOM types: HTMLElement, HTMLInputElement
```ts
const el = document.querySelector('#q') as HTMLInputElement | null;
if (el) el.value = 'x';
```
- Prefer type-safe event handlers:
```ts
const btn = document.querySelector('button')!;
btn.addEventListener('click', (e: MouseEvent) => { /* ... */ });
```
- React: type props and state
```ts
type Props = { title: string };
function Comp({ title }: Props) { return <div>{title}</div>; }
```

---

## Best practices
- Enable "strict" and fix errors incrementally.
- Prefer unknown over any.
- Use type inference where possible; explicitly annotate public API.
- Prefer composition over inheritance.
- Keep types small and focused (single responsibility).
- Use utility types to avoid duplication.
- Avoid overusing type assertions (as); prefer refinements/guards.

---

## Migration tips from JS
- Start with "allowJs": true and "checkJs": true or add JSDoc.
- Rename files .ts/.tsx gradually.
- Add types to public interfaces first.
- Use @ts-ignore sparingly and remove when possible.

---

## Real-world examples & snippets

1) API client with typed responses
```ts
type Post = { id: number; title: string; body: string };
async function getPosts(): Promise<Post[]> {
  const r = await fetch('/posts');
  return r.json() as Promise<Post[]>;
}
```

2) Typed Redux/Context selector
```ts
type State = { todos: Todo[] };
const selectTodos = (s: State) => s.todos;
```

3) Generic data loader hook (React)
```ts
function useData<T>(url: string) {
  const [data, setData] = useState<T | null>(null);
  useEffect(() => { fetch(url).then(r=>r.json()).then(d=>setData(d)); }, [url]);
  return data;
}
```

4) Safe parse JSON
```ts
function parseJSON<T>(s: string): T | null {
  try { return JSON.parse(s) as T; } catch { return null; }
}
```

---

## Common pitfalls
- Forgetting to handle undefined/null when strictNullChecks is on.
- Using any which defeats type safety.
- Misusing type assertions to silence errors.
- Overly complex types that are hard to maintain.

---

## Further reading / next steps
- In-depth: conditional & mapped types, type-level programming
- Real projects: type third-party libs, write .d.ts, advanced generics
- Tools: TypeScript Language Service, typescript-eslint rules

---

Keep this file as a concise TypeScript quick reference; expand examples as you use patterns in projects.
