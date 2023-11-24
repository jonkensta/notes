# Narrowing

## Using `this`-based type guards

You can use `this is Type` in the return position for methods in classes and interfaces:

```typescript
class FileSystemObject {
  isFile(): this is FileRep {
    return this instanceof FileRep;
  }
  isDirectory(): this is Directory {
    return this instanceof Directory;
  }
  isNetworked(): this is Networked & this {
    return this.networked;
  }
  constructor(public path: string, private networked: boolean) {}
}
 
class FileRep extends FileSystemObject {
  constructor(path: string, public content: string) {
    super(path, false);
  }
}
 
class Directory extends FileSystemObject {
  children: FileSystemObject[];
}
 
interface Networked {
  host: string;
}
 
const fso: FileSystemObject = new FileRep("foo/bar.txt", "foo");
 
if (fso.isFile()) {
  fso.content;
  
const fso: FileRep
} else if (fso.isDirectory()) {
  fso.children;
  
const fso: Directory
} else if (fso.isNetworked()) {
  fso.host;
  
const fso: Networked & FileSystemObject
}
```

## Using `assert` statements

Types can be narrowed using the `assert` function:

```typescript
assert(someValue === 42);
```

## Discriminated Unions

When modeling different variants of a Typescript interface, we might be tempted to do something as follows:

```typescript
interface Shape {
  kind: "circle" | "square";
  radius?: number;
  sideLength?: number;
}
```

Where both `"circle"` and `"square"` are modeled using the same interface.

However, here we run into trouble because we are intending `radius` to be defined for all `circle` variants and also `sideLength` to be defined for all `square` variants.
Unfortunately, this is not represented here because Typescript will assume `radius` and `sideLength` is optional for all variables of type `Shape`.

A better option is to do the following:

```typescript
interface Circle {
  kind: "circle";
  radius: number;
}
 
interface Square {
  kind: "square";
  sideLength: number;
}
 
type Shape = Circle | Square;
```

Now Typescript will correctly infer `radius` is defined for all `Shape` variables when `kind === "square"`.
