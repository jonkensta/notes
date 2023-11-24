# More on Functions

## Call Signatures

If we want to describe something callable with properties, we can write a call signature in an object type:

```typescript
type DescribableFunction = {
  description: string;
  (someArg: number): boolean;
};

function doSomething(fn: DescribableFunction) {
  console.log(fn.description + " returned " + fn(6));
}
 
function myFunc(someArg: number) {
  return someArg > 3;
}
myFunc.description = "default description";
 
doSomething(myFunc);
```

Note that the syntax is slightly different compared to a function type expression - use `:` between the parameter list and the return type rather than `=>`.

## Construct Signatures

You can write a construct signature by adding the new keyword in front of a call signature:

```typescript
type SomeConstructor = {
  new (s: string): SomeObject;
};
function fn(ctor: SomeConstructor) {
  return new ctor("hello");
}
```

## Generic Functions

### Push Type Parameters Down

Consider the following pair of functions:

```typescript
function firstElement1<Type>(arr: Type[]) {
  return arr[0];
}
 
function firstElement2<Type extends any[]>(arr: Type) {
  return arr[0];
}
 
// a: number (good)
const a = firstElement1([1, 2, 3]);
// b: any (bad)
const b = firstElement2([1, 2, 3]);
```

These might seem identical at first glance, but` firstElement1` is a much better way to write this function.
Its inferred return type is `Type`, but `firstElement2`’s inferred return type is `any` because TypeScript has to resolve the `arr[0]` expression using the constraint type, rather than "waiting" to resolve the element during a call.

### Use Fewer Type Parameters

Consider the following pair of functions:

```typescript
function filter1<Type>(arr: Type[], func: (arg: Type) => boolean): Type[] {
  return arr.filter(func);
}
 
function filter2<Type, Func extends (arg: Type) => boolean>(
  arr: Type[],
  func: Func
): Type[] {
  return arr.filter(func);
}
```

We’ve created a type parameter `Func` that doesn’t relate two values.
`Func` doesn’t do anything but make the function harder to read and reason about!
In general, it's best to use as few type parameters as possible when using generic functions.

### Type Parameters Should Appear Twice

Consider the following pair of functions:

```typescript
function greet1<Str extends string>(s: Str) {
  console.log("Hello, " + s);
}

function greet2(s: string) {
  console.log("Hello, " + s);
}
```

The second function `greet2()` is the better implementation because it avoids using a generic function altogether and does the same thing.
This is because `greet1()` does not use `Str` to relate the types of two values.
When this is the case, this means that you might be using an unnecessary generic function type.

## Optional Parameters

### Optional Parameters in Callbacks

Here is a snippet of code that illustrates one common gotcha when specifying optional parameters in callbacks:

```typescript
function myForEach(arr: any[], callback: (arg: any, index?: number) => void) {
  for (let i = 0; i < arr.length; i++) {
    callback(arr[i], i);
  }
}
```

What people usually intend when writing index? as an optional parameter is that they want both of these calls to be legal:

```typescript
myForEach([1, 2, 3], (a) => console.log(a));
myForEach([1, 2, 3], (a, i) => console.log(a, i));
```

However, in reality, this is modeling the case where the given callback might get invoked with one argument within the `myForEach()` function.

```typescript
function myForEach(arr: any[], callback: (arg: any, index?: number) => void) {
  for (let i = 0; i < arr.length; i++) {
    // I don't feel like providing the index today
    callback(arr[i]);
  }
}
```

The Typescript compiler will in turn emit errors in cases where they are not really possible:

```typescript
myForEach([1, 2, 3], (a, i) => {
  console.log(i.toFixed()); // error: 'i' is possibly 'undefined'.
});
```

The better implementation is to use the fact that Javascript ignores extra function parameters when they are passed:

```typescript
function myForEach(arr: any[], callback: (arg: any, index: number) => void) {
  for (let i = 0; i < arr.length; i++) {
    callback(arr[i], i);
  }
}
```

And this will work for both of the intended use cases above.

## Function Overloads

In TypeScript, we can specify a function that can be called in different ways by writing overload signatures.
To do this, write some number of function signatures (usually two or more), followed by the body of the function:

```
function makeDate(timestamp: number): Date;
function makeDate(m: number, d: number, y: number): Date;
function makeDate(mOrTimestamp: number, d?: number, y?: number): Date {
  if (d !== undefined && y !== undefined) {
    return new Date(y, mOrTimestamp, d);
  } else {
    return new Date(mOrTimestamp);
  }
}
const d1 = makeDate(12345678);
const d2 = makeDate(5, 5, 5);
const d3 = makeDate(1, 3);
```

In this example, we wrote two overloads: one accepting one argument, and another accepting three arguments.
These first two signatures are called the overload signatures.

Then, we wrote a function implementation with a compatible signature.
Functions have an implementation signature, but this signature can’t be called directly.
Even though we wrote a function with two optional parameters after the required one, it can’t be called with two parameters!

### Writing Good Overloads

When considering overloads, we might be tempted to write code like this:

```typescript
function len(s: string): number;
function len(arr: any[]): number;
function len(x: any) {
  return x.length;
}
```

While this works pretty well, it will not work with the following code:

```typescript
len(Math.random() > 0.5 ? "hello" : [0]);
```

For the above use this, the following simpler code using type unions works:

```typescript
function len(x: any[] | string) {
  return x.length;
}
```

For this reason, we should generally prefer type unions over function overloads.

## Declaring `this` in a Function

Typescript will infer what the `this` in a function should refer to:

```typescript
const user = {
  id: 123,
 
  admin: false,
  becomeAdmin: function () {
    this.admin = true;
  },
};
```

This can be enough in many cases, but there are also a lot of cases where you need to control over what object `this` represents.
The JavaScript specification states that you cannot have a parameter called `this`, and so TypeScript uses that syntax space to let you declare the type for this in the function body.

```typescript
interface DB {
  filterUsers(filter: (this: User) => boolean): User[];
}
 
const db = getDB();
const admins = db.filterUsers(function (this: User) {
  return this.admin;
});
```

This pattern is common with callback-style APIs, where another object typically controls when your function is called.
Note that you need to use `function` and not arrow functions to get this behavior:

```typescript
interface DB {
  filterUsers(filter: (this: User) => boolean): User[];
}
 
const db = getDB();
const admins = db.filterUsers(() => this.admin); // The containing arrow function captures the global value of 'this'.
```

## Other Types to Know About

### `void`

`void` represents the return value of functions which don’t return a value.

```typescript
// The inferred return type is void
function noop() {
  return;
}
```

### `object`

The special type object refers to any value that isn’t a primitive (`string`, `number`, `bigint`, `boolean`, `symbol`, `null`, or `undefined`).
This is different from the empty object type `{ }`, and also different from the global type `Object`.
Note that function types are considered to be `object`s in TypeScript.

### `unknown`

The `unknown` type represents any value.
This is similar to the `any` type, but is safer because it’s not legal to do anything with an `unknown` value:

```typescript
function f1(a: any) {
  a.b(); // OK
}
function f2(a: unknown) {
  a.b(); // error: 'a' is of type 'unknown'.
}
```

This is useful when describing function types because you can describe functions that accept any value without having `any` values in your function body.

Conversely, you can describe a function that returns a value of `unknown` type:

```typescript
function safeParse(s: string): unknown {
  return JSON.parse(s);
}
 
// Need to be careful with 'obj'!
const obj = safeParse(someRandomString);
```

### `never`

Some functions never return a value:

```typescript
function fail(msg: string): never {
  throw new Error(msg);
}
```

The `never` type represents values which are never observed.
In a return type, this means that the function throws an exception or terminates execution of the program.

`never` also appears when TypeScript determines there’s nothing left in a union:

```typescript
function fn(x: string | number) {
  if (typeof x === "string") {
    // do something
  } else if (typeof x === "number") {
    // do something else
  } else {
    x; // has type 'never'!
  }
}
```

### `Function`

The global type `Function` describes properties like `bind`, `call`, `apply`, and others present on all function values in JavaScript.
It also has the special property that values of type `Function` can always be called; these calls return `any`:

```typescript
function doSomething(f: Function) {
  return f(1, 2, 3);
}
```

This is an untyped function call and is generally best avoided because of the unsafe `any` return type.

If you need to accept an arbitrary function but don’t intend to call it, the type `() => void` is generally safer.

## Rest Parameters

Note that in general, TypeScript does not assume that arrays are immutable.
This can lead to some surprising behavior when using rest parameters:

```typescript
const args = [8, 5];
const angle = Math.atan2(...args); // emits an error
```

One workaround for this is to use `const`:

```typescript
// Inferred as 2-length tuple
const args = [8, 5] as const;
// OK
const angle = Math.atan2(...args);
```
