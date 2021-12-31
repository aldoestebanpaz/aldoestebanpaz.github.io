---
title: "Typescript usage notes"
tags: usage typescript ts
---

# Typescript usage notes

## Installing, initializing and configure

### 1 - Install and initialize

Having node.js and npm installed, run the `npm install` command in a project directory:

```sh
mkdir project_dir
cd project_dir
# Create a package.json file
# and automatically answer "yes" to any prompts that npm might print on the command line.
npm init -y
# Install the 'typescript' package
# and add it in the 'devDependencies' section.
npm install typescript --save-dev
# Initialize the TypeScript project creating a tsconfig.json file.
# This file will allow you to configure further and customize how TypeScript and
# the tsc transpiler interact with the project.
npx tsc --init
# Using a linter when coding will help you to quickly find inconsistencies,
# syntax errors, and omissions in your code.
# The de facto linter for TypeScript is ESLint.
npm install eslint --save-dev
# Initialize the ESLint project creating a .eslintrc.{js,yml,json} file.
# This file will allow you to configure further and customize how ESLint
# interacts with the project.
npx eslint --init
```

`--save-dev` adds the 'typescript' package in the 'devDependencies' section of the
package.json file (if the file does not exists already, it will be created with that section only).

### 2 - Configure package.json scripts

Set the following scripts in package.json:

```json
"scripts": {
  "build": "tsc",
  "watch": "tsc --watch",
  "clean": "rm -rf ./dist",
  "execute": "npm run build && node ./dist/index.js"
}
```

### 3 - Configure typescript

Open tsconfig.json and set the following values:

```
"outDir": "dist",
"sourceMap": true
```

The 'outDir' value specifies an output folder for all generated files (.js,
.d.ts, .js.map, etc.).

Activating 'sourceMap' enables the generation of sourcemap files (.js.map or or
.jsx.map files located next to the corresponding .js output file). A sourcemap
is a file that maps a generated js file to the original TypeScript file. These
files allow debuggers and other tools to display the original TypeScript source
code when actually running the generated js file.

Reference
- [tsc CLI Options](https://www.typescriptlang.org/docs/handbook/compiler-options.html)
- [TSConfig Reference](https://www.typescriptlang.org/tsconfig)

### 4 - Execute

Now you can create .ts, .tsx, and .d.ts files - even organized inside
directories, and compile them using `npm run build` or `npm run watch` (the last
will be constantly scanning any changes you make and generate files again
automatically).

Execute `npm run execute` directly if you want to build and run the generated
files.

## What tsc compiles by default

If you run `npx tsc` inside the project directory, it will read the
tsconfig.json file that instructs what to compile, what to produce, and
where to store generated files.

tsconfig.json could have an optional property called 'include' that specifies an
array of filenames or patterns to include in the program. These filenames are
resolved relative to the directory containing the tsconfig.json file and, if a
glob pattern doesn't include a file extension, then only files with supported
extensions are included (e.g. .ts, .tsx, and .d.ts by default, with .js and
.jsx if 'allowJs' is set to true).

Another optional property, 'files', allows to specify an specific and fixed
allowlist of files to include in the program. An error occurs if any of the
files can't be found. This is useful when you only have a small number of files
and don't need to use a glob to reference many files.

'include' and 'files' are two alternatives to specify the files to compile. By
default 'files' is disabled and 'include' is set to the ** glob pattern. If you
set 'files', then 'include' will be disabled otherwise.

That being said, if you run `npx tsc` then it will process all the .ts, .tsx,
and .d.ts files inside the project directory by default.

## Useful commands for troubleshooting

- `npx tsc --listFilesOnly` - Print names of files that are part of the compilation instead of building.
- `npx tsc --showConfig` - Print the final configuration instead of building.

## Stricteness

TypeScript has several type-checking strictness flags that can be turned on or
off, and all of our examples will be written with all of them enabled unless
otherwise stated.

The 'strict' flag in the CLI, or '"strict": true' in tsconfig.json toggles them
all on simultaneously, but you can opt out of them individually. The two biggest
ones you should know about are:

- 'noImplicitAny': In some places, TypeScript doesn't try to infer types for us
  and instead falls back to the most lenient type: 'any'. Turning on this flag
  will issue an error on any variables whose type is implicitly inferred as 'any'.
- 'strictNullChecks': By default, values like 'null' and 'undefined' are
  assignable to any other type. This can make writing some code easier, but
  forgetting to handle null and undefined is the cause of countless bugs in the
  world - some consider it a [billion dollar mistake](). This flag makes
  handling 'null' and 'undefined' more explicit, and spares you from worrying
  about whether you forgot to handle 'null' and 'undefined'.

### Set 'strict' to true on tsconfig.json

The strict flag enables a wide range of type checking behavior that results in stronger guarantees of program correctness. Turning this on is equivalent to enabling all of the _strict mode family_ options, which are outlined in the TSConfig reference page.

In some cases where no type annotations are present, TypeScript will fall back
to the type 'any' for a variable when it cannot infer the type. Turning on
'noImplicitAny' however TypeScript will issue an error whenever it would have
inferred 'any'. 'noImplicitAny' will default to true if 'strict' is enabled.

By default, Typescript also ignores 'null' and 'undefined'. The lack of checking for
these values tends to be a major source of bugs, that is why 'strictNullChecks'
will default to true if 'strict' is enabled.

With 'strictNullChecks' enabled, 'null' and 'undefined' have their own distinct
types and you'll get a type error if you try to use them where a concrete value
is expected. This means that you will need to use narrowing to test for those
values before using methods or properties on some variable or function parameter.

## Downleveling

'downleveling' is the process of generating code from a newer or 'higher'
version of ECMAScript down to an older or 'lower' one.

Template strings, for example, are a feature from ECMAScript 2015 (a.k.a.
ECMAScript 6, ES2015, ES6, etc.). With 'downleveling' Typescript has the ability
to rewrite the code to older ones such as ECMAScript 3 or ECMAScript 5
(a.k.a. ES3 and ES5), in this case using plain strings with concatenations (+).

By default TypeScript targets ES3, an extremely old version of ECMAScript.

You can change which JS features are downleveled and which are left intact
setting the [target](https://www.typescriptlang.org/tsconfig#target) option.

Reference:
https://www.typescriptlang.org/tsconfig#target

## Module resolution

Module resolution is the process the compiler uses to figure out what an
'import' refers to.

For an import statement like, for example, `import { a } from MODULE_PATH;` the
compiler needs to know exactly what represents 'a', and will need to check its
definition MODULE_PATH.

'moduleA' could be defined in one of your own .ts/.tsx files, or in a .d.ts that
your code depends on. The compiler will try to locate 'moduleA' using one of the
following methods:

* Locate a file that represents the imported module using the 'Classic' strategy.
* Locate a file that represents the imported module using the 'Node' strategy.
* If that didn't work and if the module name is non-relative (a module name that
  does not starts with /, ./ or ../, e.g. "moduleA", "jquery" or
  "@angular/core"), then the compiler will attempt to locate an 'ambient module
  declaration'.

Module imports are resolved differently based on whether the module reference is
relative or non-relative too.

Finally, if the compiler could not resolve the module, it will log an error. In
this case, the error would be something like `error TS2307: Cannot find module 'moduleA'`.

Reference:
https://www.typescriptlang.org/docs/handbook/modules.html
https://www.typescriptlang.org/docs/handbook/module-resolution.html

### Relative (non-absolute) module import

A relative import is resolved relative to the importing file and cannot resolve
to an 'ambient module declaration'.

A relative import is one that starts with /, ./ or ../. Some examples include:

```ts
import Entry from "./components/Entry";
import { DefaultHeaders } from "../constants/http";
import "/mod";
```

You should use relative imports for your own modules that are guaranteed to
maintain their relative location at runtime.

### Non-relative (absolute) module imports

Any other import is considered non-relative.

A non-relative import can be resolved:
- relative to baseUrl,
- through path mapping,
- or to 'ambient module declarations'.

Some examples include:

```ts
import * as $ from "jquery";
import { Component } from "@angular/core";
```

Use non-relative paths when importing any of your external dependencies.

## Type annotations, the TypeScript's type system

When you declare a variable using 'const', 'var', or 'let', you can optionally
add a type annotation to explicitly specify the type of the variable.

Wherever possible, TypeScript tries to automatically infer the types in your
code. That means that you don't always have to write explicit type annotations.

You can also add parameter type annotations and return type annotations to
function declarations. Much like variable type annotations, you usually don't
need a return type annotation because TypeScript will infer the function's
return type based on its return statements. Some codebases will explicitly
specify a return type for documentation purposes, to prevent accidental changes,
or just for personal preference.

Anonymous functions are a little bit different from function declarations. When
a function appears in a place where TypeScript can determine how it's going to
be called, the parameters of that function are automatically given types. This
process is called 'contextual typing' because the context that the function
occurred within informs what type it should have.

```ts
// Type annotations for variables
// 'string' represents string values.
const myVariable1: string = "foo";
// 'number' is for numbers like 42. JavaScript does not have a special runtime
// value for integers, so there's no equivalent to int or float - everything is
// simply number.
const myVariable2: number = 123;
// 'boolean' is for the two values: true and false.
const myVariable3: boolean = false;

// 'bigint'. From ES2020 onwards, there is a primitive in JavaScript used for
// very large integers.
// Creating a bigint via the BigInt function
let oneHundred: bigint = BigInt(100);
// Creating a BigInt via the literal syntax
oneHundred = 100n;

// 'symbol'. This is a JavasCript primitive used to create a globally unique reference via
// the function Symbol().
const name1 = Symbol("name");
const name2 = Symbol("name");
// ERROR: This condition will always return 'false' since the types 'typeof name1'
// and 'typeof name2' have no overlap.
// if (name1 === name2) { }



// No type annotation needed -- inferred as type 'string'
const myVariable4 = "foo";



// Parameter type annotation
// You can add type annotations after each parameter of a function declaration
// to declare what types of parameters the function accepts.
function foo(bar: string) {
  console.log('Ey ' + bar.toUpperCase() + '!');
}

foo('Aldo');

// ERROR: Argument of type 'number' is not assignable to parameter of type 'string'.
// foo(42);

// ERROR:
// Typescript checks that you passed the right number of arguments.
// Even if there is no type annotations on function parameters.
// foo('Aldo', 42)



// Return type annotations
// You can also add return type annotations after the parameter list.
function bar(): number {
  return 26;
}

console.log(bar());



// Contextual typing
// Even though the parameter 'item' didn't have a type annotation, TypeScript
// used the types of the forEach function, along with the inferred type of the
// array, to determine the type 'item' will have.
const myArray = ["Alice", "Bob", "Eve"];
// ERROR: Property 'touppercase' does not exist on type 'string'. Did you mean 'toUpperCase'?
// myArray.forEach(function (item) { console.log(item.touppercase()) });
// myArray.forEach((item) => console.log(item.touppercase()));



// Object types
// This refers to any JavaScript value with properties.
// You can use , or ; to separate the properties, and the last separator is optional.
// The type part of each property is also optional. If you don't specify a type,
// it will be assumed to be 'any'.
const point: { x: number; y } = { x: 3, y: 7 };
console.log(point.x);
// console.log(point.y);  // ERROR if 'strict' enabled and 'noImplicitAny' not configured.

// Object types with optional properties
// Object types can also specify that some or all of their properties are
// optional. To do this, add a ? after the property name.
// When you read from an optional property, you'll have to check for 'undefined'
// before using it.
const printName = (obj: { first: string; last?: string }) =>
  console.log(`${obj.first} ${obj.last !== undefined ? obj.last : '-'}`);
printName({ first: "Anahi" }); // prints: Anahi -
printName({ first: "Alice", last: "Alisson" }); // prints: Alice Alisson



// Union types
// A union type is a type formed from two or more other types, representing
// values that may be any one of those types. TypeScript refer to each of these
// types as the union's members.
let id: number | string;
id = 123;
console.log(id);
id = '0000-0001';
console.log(id);

// At least you use the 'narrowing' technique, TypeScript will only allow you to
// do things with the union if that thing is valid for every member of the union.
// ERROR: Property 'toUpperCase' does not exist on type 'string | number'.
// const printId = (id: number | string) => console.log(id.toUpperCase());

// Union types and Narrowing
// Narrowing occurs when TypeScript can deduce a more specific type for a value
// based on the structure of the code.
// If every member in a union has a property in common, you can use that
// property without narrowing.
function printConstants(value: number | string | string[]) {
  if (Array.isArray(value)) {
    // 'value' is of type 'string[]'
    console.log(value.map(x => x.toUpperCase()).join(", "));
  } else if (typeof value === "string") {
    // In this branch, 'value' is of type 'string'
    console.log(value.toUpperCase());
  } else {
    // Here, 'value' is of type 'number'
    console.log(value);
  }
}
printConstants(12);
printConstants(['foo', 'bar', 'baz']);
printConstants('boobar');



// Union types and literal types
interface Options {
  width: number;
}
let align: Options | "left" | "right" | "center" | "auto";
align = 'left';
console.log(align);
align = { width: 100 };
console.log(align.width);
// ERROR: Type '"aaaa"' is not assignable to type '"left" | bla bla bla
// align = 'aaaa';



// Inference with literal types
const logRequest = ( url: string, method: "GET" | "POST" | "PUT" | "PATCH" ) =>
  console.log(`url: ${url}, method: ${method}`);

// I know that req.method has the value "GET"
const req1 = { url: 'http://localhost', method: 'GET' };
logRequest(req1.url, req1.method as 'GET');

// prevents the possible assignment of e.g. "GUESS" to that field after.
const req2 = { url: 'http://localhost', method: 'GET' as 'GET' };
logRequest(req2.url, req2.method);

// 'as const' suffix acts like 'const' but for the type system, ensuring that
// all properties are assigned the literal type instead of a more general
// version like 'string' or 'number'.
const req3 = { url: 'http://localhost', method: 'GET' } as const;
logRequest(req3.url, req3.method);
```

### Primitives and the 'object' type

The following types are considered primitives:

- string
- number
- bigint
- boolean
- symbol
- null
- undefined

The special type 'object' refers to any value that isn't a primitive. This is
different from the 'empty object' type (the two curly brackets '{ }'), and also
different from the global type 'Object'.

> It's very likely you will never use 'Object'. 'object' is not 'Object'. Always
> use 'object'.

### Functions are of type 'object'

In JavaScript, function values are objects - They have properties, have
'Object.prototype' in their prototype chain, are 'instanceof Object', you can
call 'Object.keys' on them, and so on. For this reason, function types are
considered to be objects in TypeScript.

### The 'boolean' type

The type 'boolean' itself is actually just an alias for the union 'true | false'.

### The 'any' type

When you don't specify a type, and TypeScript can't infer it from context, the
compiler will typically default to 'any'.

When a value is of type 'any', you can access any properties of it (which will
in turn be of type any), call it like a function, assign it to (or from) a value
of any type, or pretty much anything else that's syntactically legal:

```ts
let obj: any = { x: 0 };
// None of the following lines of code will throw compiler errors.
// Using 'any' disables all further type checking, and it is assumed
// you know the environment better than TypeScript.
obj.foo();
obj();
obj.bar = 100;
obj = "hello";
const n: number = obj;
```

You usually want to avoid this, and will want to mark any implicit 'any' as an
error. That is why 'noImplicitAny' will default to true if 'strict' is enabled.

### The 'unknown' type

The 'unknown' type represents any value. This is similar to the 'any' type, but
is safer because it's not legal to do anything with an 'unknown' value.

```ts
function f1(a: any) {
  a.b(); // OK
}
function f2(a: unknown) {
  a.b();  // ERROR: Object is of type 'unknown'.
}



// You can describe a function that returns a value of unknown type.
function safeParse(s: string): unknown {
  return JSON.parse(s);
}
const obj = safeParse(someRandomString);  // Need to be careful with 'obj'!
```

### The 'void' and 'undefined' types

In JavaScript, a function that doesn't return any value will implicitly return
the value 'undefined'.

TypeScript defines the 'void' type that represents the return value of functions
which don't return a value.

> Contextual typing with a return type of 'void' does not force functions to not
> return something. Another way to say this is a contextual function type with a
> 'void' return type (type vf = () => void), when implemented, can return any
> other value, but it will be ignored.

> 'void' and 'undefined' are not the same thing in TypeScript.

For TypeScript 'void' will be the inferred type any time a function doesn't have
any return statements, or doesn't return any explicit value from those return
statements.

```ts
// 'void' is the inferred return type here
function noop1() { }
function noop2() { return; }



// Contextual typing with a return type of 'void' does not force functions to
// not return something.
type voidFunc = () => void;
const f1: voidFunc = () => { return true; };
const f2: voidFunc = () => true;
const f3: voidFunc = function () { return true; };
// The following variables will retain the type of 'void'
const v1 = f1();
const v2 = f2();
const v3 = f3();

// This behavior exists so that the following code is valid.
// Array.prototype.push returns a number
// and the Array.prototype.forEach method expects a function with a return type of 'void'.
const src = [1, 2, 3];
const dst = [0];
src.forEach((x) => dst.push(x));



// When a literal function definition has a 'void' return type, that function
// must not return anything.

// ERROR: Type 'boolean' is not assignable to type 'void'.
// function f2(): void {
//   return true;
// }

// ERROR: Type 'boolean' is not assignable to type 'void'.
// const f3 = function (): void {
//   return true;
// };
```

### The 'null' and 'undefined' types

By default, Typescript ignores 'null' and 'undefined'. The lack of checking for
these values tends to be a major source of bugs, that is why 'strictNullChecks'
will default to true if 'strict' is enabled.

With 'strictNullChecks' enabled, 'null' and 'undefined' have their own distinct
types and you'll get a type error if you try to use them where a concrete value
is expected. This means that you will need to use narrowing to test for those
values before using methods or properties on some variable or function parameter.

```ts
function doSomething(x: string | null) {
  if (x !== null) {
    console.log("Hello, " + x.toUpperCase());
  }
}
doSomething(null);
doSomething('foo');

// Non-null Assertion Operator (postfix !)
// TypeScript also has a special syntax for removing 'null' and 'undefined' from
// a type without doing any explicit checking. Writing ! after any expression is
// effectively a type assertion that the value isn't 'null' or 'undefined'.
function liveDangerously(x?: number | null) {
  // No error
  console.log(x!.toFixed());
}
// Works
liveDangerously(123);
// TypeError: Cannot read properties of undefined (reading 'toFixed')
liveDangerously();
// TypeError: Cannot read properties of null (reading 'toFixed')
liveDangerously(null);

// WARNING: just like other type assertions, this doesn't change the runtime behavior
// of your code, so it's important to only use ! when you know that the value
// can't be null or undefined.
```

### The 'never' type

The 'never' type represents values which are never observed.

```ts
// In a return type, this means that the function throws an exception or terminates execution of the program.
function fail(msg: string): never {
  throw new Error(msg);
}



// When TypeScript determines there's nothing left in a union it says 'never'.
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

### The 'Function' type

Values of type 'Function' can always be called; these calls return 'any'. This
is an 'untyped function' call and is generally best avoided because of the
unsafe 'any' return type.

```ts
function doSomething(f: Function) {
  return f(1, 2, 3);
}
```

> If you need to accept an arbitrary function but don't intend to call it, the
> type '() => void' is generally safer.

### Type assertions

Type assertions are a way to tell the compiler
"trust me, I know what I'm doing." A type assertion is like a type cast in other
languages, but it performs no special checking or restructuring of data. It has
no runtime impact and is used purely by the compiler. TypeScript assumes that
you, the programmer, have performed any special checks that you need.

TypeScript only allows type assertions which convert to a more specific or less
specific version of a type. This rule prevents impossible coercions like
converting 'number' to 'string'. Sometimes this rule can be too conservative and
will disallow more complex coercions that might be valid. If this happens, you
can use two assertions, first to 'any' (or 'unknown'),
then to the desired type: `(expr as any) as T`.

WARNING: There is no runtime checking associated with a type assertion. There
won't be an exception or null generated if the type assertion is wrong.

```ts
const someValue: any = 'random value';

// as-syntax
const lng1: number = (someValue as string).length;
console.log(lng1);

// angle-bracket syntax
// Does not works on .tsx files. Cannot use JSX unless the '--jsx' flag is provided.
const lng2: number = (<string>someValue).length;
console.log(lng2);
```

### Arrays

The array generic syntax like number[] or string[] is really just a shorthand
for Array<number> or Array<string>.

WARNING: Do not confuse the syntax for arrays like 'number[];' with the syntax for tuples
like '[number]' that is a different thing.

```ts
function doSomething(value: Array<string>) { }
let myArray: string[] = ["foo", "bar"];
doSomething(myArray);
myArray = new Array("hello", "world")
doSomething(myArray);
```

### Tuples

A tuple type is another sort of Array type that knows exactly how many elements
it contains, and exactly which types it contains at specific positions.

```ts
// It has no representation at runtime, but is significant to TypeScript.
// To the type system, StringNumberPair describes arrays whose
// 0 index contains a string
// and whose 1 index contains a number.
type StringNumberPair = [string, number];

function printValues(pair: [string, number]) {
  // const a = pair[0];  // const a: string
  // const b = pair[1];  // const b: number
  // You can also destructure tuples using JavaScript's array destructuring.
  const [a, b] = pair;

  // ERROR: Tuple type '[string, number]' of length '2' has no element at index '2'.
  // const c = pair[2];

  console.log(`${b * 2} -- ${a.toUpperCase()}`);
}
printValues(["foo", 12]);



// Optional tuple elements
// Tuples can have optional properties by writing out a question mark
// (? after an element's type). These optional elements can only come at the
// end, and also affect the type of length
type Either2dOr3d = [number, number, number?];
function setCoordinate(coord: Either2dOr3d) {
  // const x: number
  // const y: number
  // const z: number | undefined
  const [x, y, z] = coord;
  console.log(`Length: ${coord.length}`);  // "Length: 2" or "Length: 3"
}



// Tuples and rest elements
type StringNumberBooleans = [string, number, ...boolean[]];
// type StringBooleansNumber = [string, ...boolean[], number];
// type BooleansStringNumber = [...boolean[], string, number];
const a: StringNumberBooleans = ["hello", 1];
const b: StringNumberBooleans = ["beautiful", 2, true];
const c: StringNumberBooleans = ["world", 3, true, false, true, false, true];
```

### Type aliases and interfaces

To use a custom object definition, a custom type, more than once and refer to it
by a single name, you could use 'type aliases' or 'interfaces'.

A type alias (a.k.a. 'named type') is exactly that - a name for any type. You cannot use type aliases
to create different/distinct versions of the same type.

Interfaces are similar to type aliases, but extendable.

If you would like a heuristic, use 'interface' until you need to use features from
'type'.

```ts
type Point = {
  x: number;
  y: number;
};
// ERROR: Duplicate identifier 'Point'.
// type Point = { x: number; y: number; z: number };
const pt: Point = { x: 1, y: 100 };
console.log(`point: (${pt.x},${pt.y})`);

type Point3d = Point & {
  deep: number
}
const pt3d: Point3d = { x: 1, y: 100, deep: 0.2 };
console.log(`point: (${pt3d.x},${pt3d.y},${pt3d.deep})`);


type ID = number | string;
let myId: ID = '0000-0000';
console.log(`id: ${myId}`);
myId = 123;
console.log(`id: ${myId}`);

// Aliases are just that - aliases
// You cannot use type aliases to create different/distinct versions of the same type.
type sanitizedString = string;
const input1: sanitizedString = 'foo';
const input2: string = input1;
console.log(input2);



interface Coordinate {
  x: number;
  y: number;
}
interface Coordinate {
  deep?: number;
}
const printCoord = (coord: Coordinate) =>
  console.log(
    `x:${coord.x} - y:${coord.y} ${coord.deep ? ' - deep: ' + coord.deep : '' }`
  );
printCoord({ x: 2, y: 3 });

interface Win extends Coordinate {
  title: string
}
const printWindowInfo = (win: Win) =>
  console.log(`Win: ${win.title} (x:${win.x} - y:${win.y} - deep:${win.deep})`);
printWindowInfo({ title: 'WinSample', x: 2, y: 3, deep: 0.2 });

// Interfaces works like aliases too
interface Interface1 { title: string };
interface Interface2 { title: string };
const interface1var: Interface1 = { title: 'foo' };
const interface2var: Interface2 = interface1var;
console.log(interface2var);
```

### Object types

```ts
// Anonymous object types
const person1: { name: string; age: number } = { name: 'foobar', age: 23 };

// Named object types via interfaces
interface Person2 {
  name: string;
  age: number;
}
const person2: Person2 = { name: 'foobar', age: 23 };

// Named object types via type alias
type Person3 = {
  name: string;
  age: number;
};
const person3: Person3 = { name: 'foobar', age: 23 };



// Interfaces can extend from multiple types
interface Colorful {
  color: string;
}
interface Circle {
  radius: number;
}
interface ColorfulCircle extends Colorful, Circle {}
const cc: ColorfulCircle = {
  color: "red",
  radius: 42,
};



// Generic object types
interface Box<Type> {
  contents: Type;
}
const boxA: Box<string> = { contents: "hello" };



// Property modifiers - Optional properties
interface Shape { }
interface PaintOptions {
  shape: Shape;
  xPos?: number;
  yPos?: number;
}
function paintShape1(opts: PaintOptions) {
  // When you read an optional properties under strictNullChecks,
  // TypeScript will tell you they're potentially undefined.
  const xPos1 = opts.xPos;
  // You can just manage 'undefined' specially usin narrowing.
  const xPos2 = opts.xPos === undefined ? 0 : opts.xPos;
}
const shape = {};
paintShape1({ shape });
paintShape1({ shape, xPos: 100 });
paintShape1({ shape, yPos: 100 });
paintShape1({ shape, xPos: 100, yPos: 100 });

// Here you see a destructuring pattern for paintShape's parameter,
// and provided default values for xPos and yPos.
function paintShape2({ shape, xPos = 0, yPos = 0 }: PaintOptions) {
  console.log("x coordinate at", xPos);
  console.log("y coordinate at", yPos);
}

// There is currently no way to place type annotations within destructuring patterns.
// This is because the following syntax already means something different in JavaScript.
// function draw({ shape: Shape, xPos: number = 100 }) { }



// Property modifiers - Readonly properties
interface Home {
  readonly resident: { name: string; age: number };
}
function visitForBirthday(home: Home) {
  // You can read and update properties from 'home.resident'.
  console.log(`Happy birthday ${home.resident.name}!`);
  home.resident.age++;
}
// function evict(home: Home) {
  // ERROR: Cannot assign to 'resident' because it is a read-only property.
  // home.resident = { name: "Victor the Evictor", age: 42, };
// }
```

### The 'readonly' modifier

Object properties, arrays, index signatures, and tuples can be marked as
'readonly' on TypeScript.

While it won't change any behavior at runtime, anything marked as 'readonly'
can't be written to during type-checking.

WARNING: Using the 'readonly' modifier doesn't necessarily imply that a value is
totally immutable - It just means the property itself can't be re-written to,
but its internal contents could change.

You can make index signatures readonly too in order to prevent assignment to
their indices.

The ReadonlyArray<Type> (and the a shorthand syntax 'readonly Type[]') is also a
special type that describes arrays that shouldn't be changed.

```ts
// Readonly properties
interface Home {
  readonly resident: { name: string; age: number };
}
const home: Home = { resident: { name: 'foobar', age: 20 } };
// You can read and update properties from 'home.resident'.
home.resident.age++;
// ERROR: Cannot assign to 'resident' because it is a read-only property.
// home.resident = { name: 'foobaz', age: 20 };



// Type aliasing and readonly
// It's useful to signal intent during development time for TypeScript on how an object should be used.
interface WritablePerson { name: string; age: number; }
interface ReadonlyPerson { readonly name: string; readonly age: number; }
const writablePerson: WritablePerson = { name: "foo", age: 42 };
const readonlyPerson: ReadonlyPerson = writablePerson;
console.log(readonlyPerson.age); // prints: 42
writablePerson.age++;
console.log(readonlyPerson.age); // prints: 43

// NOTE: Using [mapping modifiers](https://www.typescriptlang.org/docs/handbook/2/mapped-types.html#mapping-modifiers), you can remove 'readonly' attributes.



// Readonly array
const fixedArray: readonly string[] = [ 'foo', 'bar', 'baz' ];
const copy = fixedArray.slice();
// ERROR: Index signature in type 'readonly string[]' only permits reading.
// fixedArray[1] = "Aldo";
// ERROR: Property 'push' does not exist on type 'readonly string[]'.
// fixedArray.push("hello!");



// Readonly index signatures
interface ReadonlyStringArray {
  readonly [index: number]: string;
}
const myArray: ReadonlyStringArray = [ 'foo', 'bar', 'baz' ];
// ERROR: Index signature in type 'ReadonlyStringArray' only permits reading.
// myArray[2] = "Mallory";
// ERROR: Property 'push' does not exist on type 'ReadonlyStringArray'.
// myArray.push("hello!");

let x: readonly string[] = [];
let y: string[] = [];
x = y;
// ERROR: The type 'readonly string[]' is 'readonly' and cannot be assigned to the mutable type 'string[]'.
// y = x;



// Readonly tuples
let readonlyTuple: readonly [string, number];
// ERROR: Variable 'readonlyTuple' is used before being assigned.
readonlyTuple[0] = "hello!";

// 'const' assertions will be inferred with readonly tuple types.
const printPoint = ([x, y]: readonly [number, number]) => console.log(`(${x},${y})`);
const point1 = [3, 4] as const;
const point2 = [3, 4];
printPoint(point1);
// ERROR: Argument of type 'number[]' is not assignable to parameter of type 'readonly [number, number]'.
// printPoint(point2);
printPoint([1, 2]);
```

## Type manipulation

TypeScript's type system is very powerful because it allows expressing types in
terms of other types.

The simplest form of this idea is generics, you actually have a wide variety of
type operators available to use. It's also possible to express types in terms
of values that you already have.

By combining various type operators, you can express complex operations and
values in a succinct, maintainable way.

The following sections cover ways to express a new type in terms of an existing
type or value.

### The 'keyof' type operator

The 'keyof' operator takes an object type and produces a string or numeric
literal union of its keys.

```ts
type Point = { x: number; y: number };
type P = keyof Point;  // same type as "x" | "y"



// If the type has a string or number 'index signature', 'keyof' will return
// those types instead.
type Arrayish = { [n: number]: unknown };
type A = keyof Arrayish;  // type A = number

// The following is because JavaScript object keys are always coerced to a
// string, so obj[0] is always the same as obj["0"].
type Mapish = { [k: string]: boolean };
type M = keyof Mapish;  // type M = string | number
```

### The 'typeof' type operator

JavaScript already has a 'typeof' operator you can use in an expression context.

TypeScript adds a 'typeof' operator you can use in a type context to refer to
the type of a variable or property.

'typeof' isn't very useful for basic types, but combined with other type
operators, you can use this to conveniently express many patterns.

> Remember that values and types aren't the same thing. To refer to the type
> that the value x has, use 'typeof'.

TypeScript intentionally limits the sorts of expressions you can use 'typeof'
on. Specifically, it's only legal to use 'typeof' on identifiers (i.e. variable
names) or their properties.

```ts
// This isn't very useful.
let s = "hello";
let n: typeof s;  // let n: string



// Meant to use = ReturnType<typeof msgbox>
// It's only legal to use 'typeof' on identifiers (i.e. variable names) or their
// properties.
let shouldContinue: typeof msgbox("Are you sure you want to continue?");



// Using the predefined type ReturnType<T>, it takes a function type and
//produces its return type.
type Predicate = (x: unknown) => boolean;
type K = ReturnType<Predicate>;  // type K = boolean



function f() {
  return { x: 10, y: 3 };
}
type P = ReturnType<typeof f>;  // type P = { x: number; y: number; }
// ERROR: 'f' refers to a value, but is being used as a type here. Did you mean
// 'typeof f'?
// If you try to use ReturnType on a function name, you will see an instructive error.
// type P = ReturnType<f>;
```

### Indexed access types

You can use an 'indexed access type' to look up a specific property on another type.

```ts
type Person = { age: number; name: string; alive: boolean };
type Age = Person["age"];  // type Age = number



// The indexing type is itself a type, so you can use unions, keyof, or other
// types entirely.
type I1 = Person["age" | "name"];  // type I1 = string | number
type I2 = Person[keyof Person];  // type I2 = string | number | boolean
type AgeOrName = "age" | "name";
type I3 = Person[AgeOrName];  // type I3 = string | boolean
// ERROR: Property 'alve' does not exist on type 'Person'
// type I1 = Person["alve"];



// You can combine 'number' with 'typeof' to conveniently capture the element
// type of an array literal.
const MyArray = [
  { name: "Alice", age: 15 },
  { name: "Bob", age: 23 },
  { name: "Eve", age: 38 },
];
type Person1 = typeof MyArray[number];  // type Person = { name: string; age: number; }
type Age1 = Person1["age"];  // type Age1 = number
type Age2 = typeof MyArray[number]["age"];  // type Age2 = number



// You can only use types when indexing.
type Person2 = { name: string; age: number; }
type key = "age";
type Age = Person2[key];
// You can't use a const to make a variable reference.
// const keyConst = "age";
// ERROR: Type 'key' cannot be used as an index type. 'key' refers to a value,
// but is being used as a type here. Did you mean 'typeof key'?
// type Age = Person[keyConst];
```

### Conditional types

Conditional types help describe the relation between the types of inputs and
outputs.

```ts
interface IdLabel {
  id: number;
}
interface NameLabel {
  name: string;
}
type NameOrId<T extends number | string> = T extends number ? IdLabel : NameLabel;
function createLabel<T extends number | string>(idOrName: T): NameOrId<T> {
  throw "unimplemented";
}
let a = createLabel("typescript");  // let a: NameLabel
let b = createLabel(2.8);  // let b: IdLabel
let c = createLabel(Math.random() ? "hello" : 42);  // let c: NameLabel | IdLabel



// Conditional Type Constraints

// In the following example T has a constraint where it should have a 'message'
// property. Without a contraint it will fail necause TypesCript cannot determine
// if any object has that property.
type MessageOf1<T extends { message: unknown }> = T["message"];
interface Email {
  message: string;
}
type EmailMessageContents1 = MessageOf1<Email>;  // type EmailMessageContents = string

// In the following MessageOf2 can take any type, and default to something like
// 'never' if a 'message' property isn't available
type MessageOf2<T> = T extends { message: unknown } ? T["message"] : never;
interface Email {
  message: string;
}
interface Dog {
  bark(): void;
}
type EmailMessageContents2 = MessageOf2<Email>;  // type EmailMessageContents = string
type DogMessageContents = MessageOf2<Dog>;  // type DogMessageContents = never

// The following flattens array types to their element types, but leaves them alone otherwise
// (alternative 1) Element type in Flatten extracted manually with an indexed access type
type Flatten<T> = T extends any[] ? T[number] : T;
// (alternative 2) You could have inferred the element type in Flatten instead of fetching it out manually with an indexed access type
// type Flatten<T> = T extends Array<infer ItemT> ? ItemT : T;
type Str1 = Flatten<string[]>;  // type Str = string
type Num1 = Flatten<number>;  // type Num = number



// Inferring Within Conditional Types

// extract the return type out from function types
type GetReturnType<Type> = Type extends (...args: never[]) => infer Return ? Return : never;
type Num2 = GetReturnType<() => number>;  // type Num = number
type Str2 = GetReturnType<(x: string) => string>;  // type Str = string
type Bools = GetReturnType<(a: boolean, b: boolean) => boolean[]>;  // type Bools = boolean[]
// NOTE: It is not possible to perform overload resolution based on a list of argument types.
declare function stringOrNum(x: string): number;
declare function stringOrNum(x: number): string;
declare function stringOrNum(x: string | number): string | number;
type T1 = GetReturnType<typeof stringOrNum>;  // type T1 = string | number



// Distributive Conditional Types
type ToArray<Type> = Type extends any ? Type[] : never;
type StrArrOrNumArr1 = ToArray<string | number>;  // type StrArrOrNumArr = string[] | number[]

// Typically, distributivity is the desired behavior. To avoid that behavior,
// you can surround each side of the extends keyword with square brackets.
type ToArrayNonDist<Type> = [Type] extends [any] ? Type[] : never;
type StrArrOrNumArr2 = ToArrayNonDist<string | number>;  // type StrArrOrNumArr = (string | number)[]
```

### Mapped types

A mapped type is a generic type which uses a union of PropertyKeys (frequently
created via a 'keyof') to iterate through keys to create a type.

There are two modifiers which can be applied during mapping: 'readonly' and '?'
which affect mutability and optionality respectively. You can remove or add
these modifiers by prefixing with '-' or '+'. If you don't add a prefix, then
'+' is assumed.

In TypeScript 4.1 and onwards, you can re-map keys in mapped types with an 'as'
clause in a mapped type. You can also leverage features like 'template literal
types' to create new property names from prior ones.

Mapped types work well with other features for 'type manipulation', see the last
example in the following code.

```ts
type OptionsFlags<Type> = {
  [Property in keyof Type]: boolean;
};
type FeatureFlags = {
  darkMode: () => void;
  newUserProfile: () => void;
};
type FeatureOptions = OptionsFlags<FeatureFlags>;
// equals to:
// type FeatureOptions = {
//     darkMode: boolean;
//     newUserProfile: boolean;
// }



// Mapping modifiers

// The following creates a new type without 'readonly' attributes from a type's properties
type CreateMutable<Type> = {
  -readonly [Property in keyof Type]: Type[Property];
};
type LockedAccount = {
  readonly id: string;
  readonly name: string;
};
type UnlockedAccount = CreateMutable<LockedAccount>;
// equals to:
// type UnlockedAccount = {
//     id: string;
//     name: string;
// }

// The following creates a new type without optional attributes from a type's properties
type Concrete<Type> = {
  [Property in keyof Type]-?: Type[Property];
};
type MaybeUser = {
  id: string;
  name?: string;
  age?: number;
};
type User = Concrete<MaybeUser>;
// equals to:
// type User = {
//     id: string;
//     name: string;
//     age: number;
// }



// Key remapping

// The following creates a type with getters
type Getters<Type> = {
    [Property in keyof Type as `get${Capitalize<string & Property>}`]: () => Type[Property]
};
interface Person {
    name: string;
    age: number;
    location: string;
}
type LazyPerson = Getters<Person>;
// equals to:
// type LazyPerson = {
//     getName: () => string;
//     getAge: () => number;
//     getLocation: () => string;
// }

// The following filter out keys ('kind' property in this example) by producing
// 'never' via a conditional type
type RemoveKindField<Type> = {
    [Property in keyof Type as Exclude<Property, "kind">]: Type[Property]
};
interface Circle {
    kind: "circle";
    radius: number;
}
type KindlessCircle = RemoveKindField<Circle>;
// equals to:
// type KindlessCircle = {
//     radius: number;
// }

// Mapping over arbitrary unions
type EventConfig<Events extends { kind: string }> = {
    [E in Events as E["kind"]]: (event: E) => void;
}
type SquareEvent = { kind: "square", x: number, y: number };
type CircleEvent = { kind: "circle", radius: number };
type Config = EventConfig<SquareEvent | CircleEvent>
// equals to:
// type Config = {
//     square: (event: SquareEvent) => void;
//     circle: (event: CircleEvent) => void;
// }

// Mapped type + conditional type
// This returns either a 'true' or 'false' depending on whether an object has the
// property 'pii' set to the literal 'true'.
type ExtractPII<Type> = {
  [Property in keyof Type]: Type[Property] extends { pii: true } ? true : false;
};
type DBFields = {
  id: { format: "incrementing" };
  name: { type: string; pii: true };
};
type ObjectsNeedingGDPRDeletion = ExtractPII<DBFields>;
// equals to:
// type ObjectsNeedingGDPRDeletion = {
//     id: false;
//     name: true;
// }
```

### Template literal types

Template literal types build on string literal types, and have the ability to
expand into many strings via unions. They have the same syntax as template
literal strings in JavaScript, but are used in type positions.

```ts
// Template literal types with concrete types
type World = "world";
type Greeting = `hello ${World}`;  // type Greeting = "hello world"



// Template literal types with onion in the interpolated position
type EmailLocaleIDs = "welcome_email" | "email_heading";
type FooterLocaleIDs = "footer_title" | "footer_sendoff";
type AllLocaleIDs = `${EmailLocaleIDs | FooterLocaleIDs}_id`;
// equals to:
// type AllLocaleIDs = "welcome_email_id" | "email_heading_id" | "footer_title_id" | "footer_sendoff_id"

// For each interpolated position in the template literal, the unions are cross multiplied.
type AllLocaleIDs = `${EmailLocaleIDs | FooterLocaleIDs}_id`;
type Lang = "en" | "ja" | "pt";
type LocaleMessageIDs = `${Lang}_${AllLocaleIDs}`;
// equals to:
// type LocaleMessageIDs = "en_welcome_email_id" | "en_email_heading_id" | "en_footer_title_id" | "en_footer_sendoff_id" | "ja_welcome_email_id" | "ja_email_heading_id" | "ja_footer_title_id" | "ja_footer_sendoff_id" | "pt_welcome_email_id" | "pt_email_heading_id" | "pt_footer_title_id" | "pt_footer_sendoff_id"



// String Unions in Types
type Person = {
  firstName: string;
  lastName: string;
  age: number;
};
type PropEventSource1<Type> = {
    on(eventName: `${string & keyof Type}Changed`, callback: (newValue: any) => void): void;
};
declare function makeWatchedObject1<Type>(obj: Type): Type & PropEventSource1<Type>;
const person1 = makeWatchedObject1({
  firstName: "Saoirse",
  lastName: "Ronan",
  age: 26
});
person1.on("firstNameChanged", () => {});
// ERROR: Argument of type '"firstName"' is not assignable to parameter of type '"firstNameChanged" | "lastNameChanged" | "ageChanged"'.
// person1.on("firstName", () => {});



// Inference with Template Literals
type PropEventSource2<Type> = {
    on<Key extends string & keyof Type>
      (eventName: `${Key}Changed`, callback: (newValue: Type[Key]) => void): void;
};
declare function makeWatchedObject2<Type>(obj: Type): Type & PropEventSource2<Type>;
const person2 = makeWatchedObject2({
  firstName: "Saoirse",
  lastName: "Ronan",
  age: 26
});
person2.on("firstNameChanged", newName => {     // newName: string
  console.log(`new name is ${newName.toUpperCase()}`);
});
person2.on("ageChanged", newAge => {            // newAge: number
  if (newAge < 0) {
      console.warn("warning! negative age");
  }
});



// Intrinsic String manipulation types
// To help with string manipulation, TypeScript includes a set of types which
// can be used in string manipulation. These types come built-in to the compiler
// for performance and can't be found in the .d.ts files included with
// TypeScript.

// Uppercase<StringType>
type Greeting = "Hello, world"
type ShoutyGreeting = Uppercase<Greeting>
// type ShoutyGreeting = "HELLO, WORLD"
type ASCIICacheKey<Str extends string> = `ID-${Uppercase<Str>}`
type MainID = ASCIICacheKey<"my_app">
// type MainID = "ID-MY_APP"

// Lowercase<StringType>
type Greeting = "Hello, world"
type QuietGreeting = Lowercase<Greeting>
// type QuietGreeting = "hello, world"
type ASCIICacheKey<Str extends string> = `id-${Lowercase<Str>}`
type MainID = ASCIICacheKey<"MY_APP">
// type MainID = "id-my_app"

// Capitalize<StringType>
// Converts the first character in the string to an uppercase equivalent.
type LowercaseGreeting = "hello, world";
type Greeting = Capitalize<LowercaseGreeting>;
// type Greeting = "Hello, world"

// Uncapitalize<StringType>
// Converts the first character in the string to a lowercase equivalent.
type UppercaseGreeting = "HELLO WORLD";
type UncomfortableGreeting = Uncapitalize<UppercaseGreeting>;
// type UncomfortableGreeting = "hELLO WORLD"
```

## Truthy and falsy values

In JavaScript, constructs like 'if' first 'coerce' their conditions to booleans to
make sense of them, and then choose their branches depending on whether the
result is true or false.

The following values corce to false:

```
false
0
-0
0n    (the bigint version of zero)
NaN
null
undefined
""    (the empty string)
```

All other values, including any object, an empty array ([]), or the string
"false", create an object with an initial value of true.

## Type guards and narrowing

Within 'if' checks TypeScript sees your 'typeof' expressions and understands
those as special forms of code called a 'type guards'. TypeScript follows
possible paths of execution that our programs can take to analyze the most
specific possible type of a value at a given position. It looks at these special
checks (called type guards) and assignments, and the process of refining types
to more specific types than declared is called narrowing.

### The 'typeof' type guard

In TypeScript, checking against the value returned by 'typeof' is a common 'type guard'.

The following are the possible values 'typeof' returns:

- "string"
- "number"
- "bigint"
- "boolean"
- "symbol"
- "undefined"
- "object"
- "function"

WARNING: in JavaScript, 'typeof null' is actually 'object'! This is one of those
unfortunate accidents of history.

```ts
// Type inference and narrowing
// TypeScript looks at the right side of the assignment and narrows the left side appropriately.
let x = Math.random() < 0.5 ? 1 : "foo";  // inferred as let 'x: string | number'
x = 2;
x = 'bar';
// ERROR: Type 'boolean' is not assignable to type 'string | number'.
x = true;

// At least you use the 'narrowing' technique, TypeScript will only allow you to
// do things with the union if that thing is valid for every member of the union.
// ERROR: Property 'toUpperCase' does not exist on type 'string | number'.
// const printId = (id: number | string) => console.log(id.toUpperCase());

// Union types and Narrowing
// Narrowing occurs when TypeScript can deduce a more specific type for a value
// based on the structure of the code.
// If every member in a union has a property in common, you can use that
// property without narrowing.
function printConstants(value: number | string | string[]) {
  if (Array.isArray(value)) {
    // 'value' is of type 'string[]'
    console.log(value.map(x => x.toUpperCase()).join(", "));
  } else if (typeof value === "string") {
    // In this branch, 'value' is of type 'string'
    console.log(value.toUpperCase());
  } else {
    // Here, 'value' is of type 'number'
    console.log(value);
  }
}
printConstants(12);
printConstants(['foo', 'bar', 'baz']);
printConstants('boobar');



const printAll1 = (value: string | string[] | null) => {
  if (typeof value === "string") {
    console.log(value);
  }
  // 'typeof null' is actually 'object'
  // ERROR; Object is possibly 'null'.
  // else if (typeof value === "object") {
  //   for (const s of value) {
  //     console.log(s);
  //   }
  // }
  else if (value && typeof value === "object") {
    for (const s of value) {
      console.log(s);
    }
  }
}

const printAll2 = (value: string | string[] | null) => {
  // narrowing to non-null values
  if (value !== null) {
    if (typeof value === "string") {
      console.log(strs);
    }
    else if (typeof value === "object") {
      for (const s of strs) {
        console.log(s);
      }
    }
  }
}
```

### The 'in' type guard for objects

JavaScript has the 'in' operator for determining if an object has a property
with a name. TypeScript takes this into account as a way to narrow down
potential types.

```ts
type Fish = { swim: () => void };
type Bird = { fly: () => void };

function move(animal: Fish | Bird) {
  if ("swim" in animal) {
    return animal.swim();
  }
  return animal.fly();
}

// both prints
move({ swim: () => console.log('swimming!') });
move({ fly: () => console.log('flying!') });
```

### The 'instanceof' type guard for objects

JavaScript has the 'instanceof' operator for checking whether or not a value is
an instance of another value. `x instanceof Foo` checks whether the prototype
chain of x contains Foo.prototype.

This is useful for most values that can be constructed with the 'new' operator.

```ts
function logValue(x: Date | string) {
  if (x instanceof Date) {
    console.log(x.toUTCString());
  } else {
    console.log(x.toUpperCase());
  }
}
logValue(new Date);  // prints: Tue, 07 Dec 2021 01:05:49 GMT
logValue('uppercase');  // prints: UPPERCASE
```

### Type predicates as a type guard for narrowing to specific object types

A 'type predicate' is a special return type that signals to the Typescript
compiler what type a particular value is. A type predicate is always attached to
a function that takes a single argument and returns a boolean.

```ts
interface Cat {
  numberOfLives: number;
}
interface Dog {
  isAGoodBoy: boolean;
}

function isCat(animal: Cat | Dog): animal is Cat {
  return (animal as Cat).numberOfLives !== undefined;
}

const animal: Cat | Dog = { numberOfLives: 10 };

if (isCat(animal)) {
  // animal successfully cast as a Cat
  console.log(animal.numberOfLives);
} else {
  // animal successfully cast as a Dog
  console.log(animal.isAGoodBoy);

  // ERROR: Property 'numberOfLives' does not exist on type 'never'.
  // TypeScript will use a never type to represent a state which shouldn't exist.
  // console.log(animal.numberOfLives);
}
```

Since this function returns a boolean and includes the type predicate
'animal is Cat', the Typescript compiler will correctly cast the 'animal' as
'Cat' if 'isCat' evaluates as 'true'. It will also cast 'animal' as 'Dog' if
'isCat' evaluates as false.

## Enums

Enums are one of the few features TypeScript has which is not a type-level
extension of JavaScript.

Enums allow a developer to define a set of named constants. Using enums can make
it easier to document intent, or create a set of distinct cases. TypeScript
provides both numeric and string-based enums.

```ts
// Numeric enums
// Up would have the value 0, Down would have 1, etc
enum Direction1 {
  Up,
  Down,
  Left,
  Right,
}
// Up has the value 1, Down has 2, Left has 3, and Right has 4
enum Direction2 {
  Up = 1,
  Down,
  Left,
  Right,
}
const value1: Direction1 = Direction1.Up;
const value2 = Direction1['Up'];
console.log(value1);  // prints: 0
console.log(value2);  // prints: 0

// You can use literal enum values as literal types.
let upDirection: Direction1.Up = Direction1.Up;
// ERROR: Type 'Direction1.Left' is not assignable to type 'Direction1.Up'
// upDirection = Direction1.Left;

// Numeric enums are compiled into an object that stores both
// forward (name -> value)
// and reverse (value -> name) mappings.
const rDirection = Direction1.Right;
let keyOfValue = Direction1[rDirection];
console.log(`${keyOfValue} : ${rDirection}`);  // prints: Right : 3
keyOfValue = Direction1[0];
console.log(`${keyOfValue} : 0`);  // prints: Up : 0



// Enum.Key    -- returns the value
// Enum["Key"] -- returns the value
// type EnumKeys = keyof typeof Enum; -- defines an union with all enum keys as strings
enum LogLevel {
  ERROR,
  WARN,
  INFO,
  DEBUG,
}
/**
 * This is equivalent to:
 * type LogLevelStrings = 'ERROR' | 'WARN' | 'INFO' | 'DEBUG';
 */
type LogLevelStrings = keyof typeof LogLevel;
let lvl: LogLevelStrings = "WARN";
console.log(`${lvl} : ${LogLevel[lvl]}`);  // prints: WARN : 1
console.log(LogLevel[lvl] < LogLevel.INFO ? 'true' : 'false');  // prints: true


// String enums
// String enums are a similar concept, but have some subtle runtime differences
enum Direction3 {
  Up = "UP",
  Down = "DOWN",
  Left = "LEFT",
  Right = "RIGHT"
}
// String enum members do not get a reverse mapping generated at all.
// ERROR: Property 'RIGHT' does not exist on type 'typeof Direction3'. Did you mean 'Right'?
// console.log(Direction3["RIGHT"]);



// Const enums
// Const enums can only use constant enum expressions
// and unlike regular enums they are completely removed during compilation.
const enum Direction4 {
  Up,
  Down,
  Left,
  Right,
}
```

While string enums don't have auto-incrementing behavior, string enums have the
benefit that they serialize well. In other words, if you were debugging and had
to read the runtime value of a numeric enum, the value is often opaque - it
doesn't convey any useful meaning on its own (though reverse mapping can often
help), string enums allow you to give a meaningful and readable value when your
code runs, independent of the name of the enum member itself.

## Generics

In TypeScript, generics are used when you want to describe a correspondence between two values.

```ts
// Generic interfaces
// You might read this as “A Box of Type is something whose contents have type Type”.
interface Box1<Type> {
  contents: Type;
}
interface Apple { }

// When you refer to Box, you have to give a type argument in place of Type.
let stringBox: Box1<string>;
// Box is reusable in that Type can be substituted with anything.
let appleBox: Box1<Aapple>;



// Generic type aliases
// It is worth noting that type aliases can also be generic.
type Box2<Type> = {
  contents: Type;
};

// Since type aliases, unlike interfaces, can describe more than just object types,
// you can also use them to write other kinds of generic helper types.
type OrNull<Type> = Type | null;
type OneOrMany<Type> = Type | Type[];
type OneOrManyOrNull<Type> = OrNull<OneOrMany<Type>>;

type OneOrManyOrNullStrings = OneOrManyOrNull<string>;
```

## Functions

Functions, like other types, are also values, and just like other values,
TypeScript has many ways to describe how functions can be called.

> Remember, when using generics with functions, type parameters are for relating
> the types of multiple values. If a type parameter is only used once in the
> function signature, it's not relating anything.

> Rule: Type Parameters should appear twice. If a type parameter only appears in
> one location, strongly reconsider if you actually need it.

```ts
// Function type with alias
// These types are syntactically similar to arrow functions.
// Note that the PARAMETER NAME IS REQUIRED.

// (a: string) => void means "a function with one parameter, named a, of type
// string, that doesn't have a return value"
type Foo = (a: string) => void;
function fooCaller(fn: Foo) { }

// (string) => void means "a function with a parameter named string of type any"
type Bar = (string) => void;
function barCaller(fn: Bar) { }



// Function with Optional parameters
// Although the parameter is specified as type 'number', the x parameter will
// actually have the type 'number | undefined' because unspecified parameters in
// JavaScript get the value 'undefined'.
function f1(x?: number) { }
f1();
f1(10);
// When a parameter is optional, callers can always pass 'undefined', as this
// simply simulates a missing argument.
f1(undefined);



// Function with default parameters
// Here x will have type 'number' because any 'undefined' argument will be replaced with 10.
function f2(x = 10) { }
f2();  // prints: 10
f2(42);  // prints: 42
f2(undefined);  // prints: 10



// Function type expression in function declaration
function greeter(fn: (a: string) => void) {
  fn("Hello, World");
}
function printToConsole(s: string) {
  console.log(s);
}
greeter(printToConsole);



// DON'T DO: Callbacks with optional parameters
// When writing a function type for a callback, never write an optional parameter
// unless you intend to call the function without passing that argument.
// function myForEach(arr: any[], callback: (arg: any, index?: number) => void) {
//   for (let i = 0; i < arr.length; i++) {
//     callback(arr[i], i);
//   }
// }
// myForEach([1, 2, 3], (a, i) => {
//   console.log(i.toFixed());  // Object is possibly 'undefined'.
// });



// Parameter Destructuring
// You can use parameter destructuring to conveniently unpack objects provided
// as an argument into one or more local variables in the function body.
function sum1({ a, b, c }: { a: number; b: number; c: number }) {
  console.log(a + b + c);
}

// Parameter Destructuring with a named type
type ABC = { a: number; b: number; c: number };
function sum2({ a, b, c }: ABC) {
  console.log(a + b + c);
}



// 'void' is the inferred return type here
function noop() {
  return;
}

// 'never' return type here
// This means that the function throws an exception or terminates execution of the program.
function fail(msg: string): never {
  throw new Error(msg);
}



// Generic functions
// It's common to write a function where the types of the input relate to the
// type of the output, or where the types of two inputs are related in some way.
// Note that you didn't have to specify Type in this sample. The type was
// inferred - chosen automatically - by TypeScript.
function firstElement<T>(arr: T[]): Type | undefined {
  return arr[0];
}
// s is of type 'string'
const s = firstElement(["a", "b", "c"]);
// n is of type 'number'
const n = firstElement([1, 2, 3]);
// u is of type undefined
const u = firstElement([]);

// TypeScript could infer both the type of the Input type parameter (from the
// given string array), as well as the Output type parameter based on the return
// value of the function expression (number).
function map<Input, Output>(arr: Input[], func: (arg: Input) => Output): Output[] {
  return arr.map(func);
}
// Parameter 'n' is of type 'string'
// 'parsed' is of type 'number[]'
const parsed = map(["1", "2", "3"], (n) => parseInt(n));



// Generic types
// The type of generic functions is just like those of non-generic functions,
// with the type parameters listed first, similarly to function declarations.
function identity<Type>(arg: Type): Type {
  return arg;
}
let myIdentity1: <Input>(arg: Input) => Input = identity;

// You can also write the generic type as a call signature of an object literal type.
let myIdentity2: { <Type>(arg: Type): Type } = identity;

// You can also replace the object literal with an interface.
interface GenericIdentityFn1 {
  <Type>(arg: Type): Type;
}
let myIdentity3: GenericIdentityFn1 = identity;

// You may want to move the generic parameter to be a parameter of the whole
// interface (Generic interface). This lets you see what type(s) we're generic over
// (e.g. Dictionary<string> rather than just Dictionary). This makes the type
// parameter visible to all the other members of the interface.
interface GenericIdentityFn2<Type> {
  (arg: Type): Type;
}
let myIdentit4: GenericIdentityFn2<number> = identity;



// Specifying Type Arguments
// TypeScript can usually infer the intended type arguments in a generic call, but not always.
// You could manually specify Type if needed.
function combine<Type>(arr1: Type[], arr2: Type[]): Type[] {
  return arr1.concat(arr2);
}
const arr = combine<string | number>([1, 2, 3], ["hello"]);
```

### Rest parameters and the spread syntax

You can define functions that take an unbounded number of arguments using
'rest parameters'. A 'rest parameter' appears after all other parameters, and
uses the '...' syntax.

In TypeScript, the type annotation on these parameters is implicitly 'any[]'
instead of 'any', and any type annotation given must be of the form 'Array<T>'
or 'T[]', or a tuple type.

```ts
function multiply(n: number, ...m: number[]) {
  return m.map((x) => n * x);
}
const a = multiply(10, 1, 2, 3, 4);  // 'a' gets value [10, 20, 30, 40]
```

In TypeScript you could use the 'spread syntax' with tuple types or when an array
is passed to a 'rest parameter'.

```ts
// Array passed to a 'rest parameter'
// e.g. The push method of arrays takes any number of arguments
const arr1 = [1, 2, 3];
const arr2 = [4, 5, 6];
arr1.push(...arr2);



// ERROR: A spread argument must either have a tuple type or be passed to a rest parameter.
// Inferred type is number[] -- "an array with zero or more numbers",
// not specifically two numbers

// const args = [8, 5];
// const angle = Math.atan2(...args);
```

### Function overloads

Some JavaScript functions can be called in a variety of argument counts and
types.

In TypeScript, you can specify a function that can be called in different ways
by writing overload signatures.

> You should always have two or more signatures above the implementation of the
> function.

> Rule: Always, when possible, prefer parameters with 'union types' instead of
> 'function overloads'.

In the following example, there is a function that produces a Date that
takes either a timestamp (one argument) or a month + day + year specification
(three arguments).

```ts
// Overload signatures
function makeDate(timestamp: number): Date;
function makeDate(m: number, d: number, y: number): Date;
// Implementation signature
// This signature can't be "seen" from the outside and even can't be called
// directly. Even though you wrote a function with two optional parameters after
// the required one, it can't be called with two parameters.
function makeDate(mOrTimestamp: number, d?: number, y?: number): Date {
  if (d !== undefined && y !== undefined) {
    return new Date(y, mOrTimestamp, d);
  } else {
    return new Date(mOrTimestamp);
  }
}
const d1 = makeDate(12345678);
const d2 = makeDate(5, 5, 5);
// ERROR: No overload expects 2 arguments, but overloads do exist that expect either 1 or 3 arguments.
// const d3 = makeDate(1, 3);
```

### Generic functions with constraints (a.k.a. Generic constraints)

NOTE **function promises to return the same kind of object as was passed in**,
not just some object matching the constraint.

> Rule: When possible, use the type parameter itself rather than constraining it.

```ts
// Generic functions with constraints
// Because you constrained Type to { length: number }, you were allowed to
// access the .length property of the a and b parameters. Without the type
// constraint, you wouldn't be able to access those properties because the
// values might have been some other type without a length property.
// Note return type inference also works on generic functions. The types of
// longerArray and longerString were inferred based on the arguments.
function longest1<Type extends { length: number }>(a: Type, b: Type) {
  if (a.length >= b.length) {
    return a;
  } else {
    return b;
  }
}
// longerArray is of type 'number[]'
const longerArray1 = longest1([1, 2], [1, 2, 3]);
// longerString is of type 'alice' | 'bob'
const longerString1 = longest1("alice", "bob");
// You could pass in values whose type has all the required properties.
const longerObj1 = longest1({ length: 5, value: 'me' }, { length: 1, value: 'notMe' });
// ERROR: Numbers don't have a 'length' property
// const notOK = longest(10, 100);



// You can create an interface that describes the constraint too.
interface Lengthwise {
  length: number;
}
function longest2<Type extends Lengthwise>(a: Type, b: Type) {
  if (a.length >= b.length) {
    return a;
  } else {
    return b;
  }
}
const longerArray2 = longest2([1, 2], [1, 2, 3]);
const longerString2 = longest2("alice", "bob");



// You can declare a type parameter that is constrained by another type parameter.
function getProperty<Type, Key extends keyof Type>(obj: Type, key: Key) {
  return obj[key];
}
const x = { a: 1, b: 2, c: 3, d: 4 };
getProperty(x, "a");
// ERROR: Argument of type '"m"' is not assignable to parameter of type '"a" | "b" | "c" | "d"'.
// getProperty(x, "m");



// Example of wrong function with constrains

// Function promises to return the same kind of object as was passed in,
// not just some object matching the constraint.
// function minimumLength<Type extends { length: number }>(
//   obj: Type,
//   minimum: number
// ): Type {
//   if (obj.length >= minimum) {
//     return obj;
//   } else {
//     // ERROR:
//     // Type '{ length: number; }' is not assignable to type 'Type'.
//     // '{ length: number; }' is assignable to the constraint of type 'Type', but
//     // 'Type' could be instantiated with a different subtype of constraint
//     // '{ length: number; }'.
//     return { length: minimum };
//   }
// }

// If this code above were legal, you could write code that definitely wouldn't work.

// const arr = minimumLength([1, 2, 3], 6);  // 'arr' gets value { length: 6 }
// arr.slice(0);  // and crashes here because arrays have a 'slice' method, but not the returned object.
```

### Call signatures and construct signatures

In JavaScript, functions can have properties in addition to being callable. If
you want to describe something callable with properties, you have to write a
'call signature' in an object type.

JavaScript functions can also be invoked with the 'new' operator. TypeScript
refers to these as 'constructors' because they usually create a new object.


```ts
// Call signatures
// Syntax is slightly different compared to a function type expression
// use : between the parameter list and the return type rather than =>.
type DescribableFunction = {
  description: string;
  (someArg: number): boolean;
};
function doSomething(fn: DescribableFunction) {
  console.log(fn.description + " returned " + fn(6));
}

// Construct signatures
type SomeObject = { };
type SomeConstructor = {
  new (s: string): SomeObject;
};
function fn(ctor: SomeConstructor) {
  return new ctor("hello");
}

// Call signatures + Construct Signatures
// You can combine call and construct signatures in the same type arbitrarily.
// e.g. Some objects, like JavaScript's Date object, can be called with or without new.
interface CallOrConstruct {
  new (s: string): Date;
  (n?: number): number;
}
```

# Classes

TypeScript offers full support for the class keyword introduced in ES2015 and
also adds type annotations and other syntax to allow you to express relationships
between classes and other types.

For fields in classes, as with other locations, the type annotation is optional,
but will be an implicit any if not specified. Also, just like with const, let,
and var, the initializer of a class property will be used to infer its type.

```ts
// Fields with initializers
class Point1 {
  x: number = 0;
  y = 0;
}
const pt = new Point1();
pt.x = 1;
// ERROR: Type 'string' is not assignable to type 'number'.
// pt.y = "0";
console.log(pt);  // prints: Point1 { x: 1, y: 0 }



// Fields initialized in the constructor
// Use --strictPropertyInitialization if you want to force this type of initialization.
class GoodGreeter {
  name: string;

  constructor() {
    this.name = "hello";
  }
}



// Fields not initialized forced with the 'definite assignment assertion'
class OKGreeter {
  // Not initialized, but no error
  name!: string;
}



// Fields may be prefixed with the readonly modifier.
// This prevents assignments to the field outside of the constructor.
class Greeter {
  readonly name: string = "world";

  constructor(name?: string) {
    if (name !== undefined) {
      this.name = name;
    }
  }

  err() {
    // ERROR: Cannot assign to 'name' because it is a read-only property.
    // this.name = "not ok";
  }
}
const g = new Greeter();
// ERROR: Cannot assign to 'name' because it is a read-only property.
// g.name = "also not ok";



// Constructor with initializers
class Point2 {
  x: number;
  y: number;

  constructor(x = 0, y = 0) {
    this.x = x;
    this.y = y;
  }
}



// Constructor with overloads
// There are just a few differences between class constructor signatures and
// function signatures:
// - Constructors can’t have type parameters (Generics).
// - Constructors can’t have return type annotations.
class Point3 {
  constructor(x: number, y: string);
  constructor(s: string);
  constructor(xs: any, y?: any) { }
}



// Base and derived classes (a.k.a. Superclass and subclass)
class Base {
  k = 4;
}
class Derived extends Base {
  constructor() {
    // ERROR: 'super' must be called before accessing 'this' in the constructor of a derived class.
    // This prints a wrong value in ES5; throws exception in ES6
    // console.log(this.k);
    super();
    console.log(this.k);
  }
}



// Class with constructor and a sample method
class Point4 {
  x: number;
  y: number;

  constructor(x = 10, y = 10) {
    this.x = x;
    this.y = y;
  }

  scale(n: number): void {
    // ERROR: Cannot find name 'x'. Did you mean the instance member 'this.x'?
    // Note that inside a method body, it is still mandatory to access fields
    // and other methods via 'this.'. An unqualified name in a method body will
    // always refer to something in the enclosing scope.
    // x = 5;
    this.x *= n;
    this.y *= n;
  }
}



// Class with index signatures
class MyClass {
  [s: string]: boolean | ((s: string) => boolean);

  check(s: string) {
    return this[s] as boolean;
  }
}



// Class with accessors (getters and setters)
// TypeScript has some special inference rules for accessors:
// - If get exists but no set, the property is automatically readonly.
// - If the type of the setter parameter is not specified, it is inferred from the return type of the getter.
// - Getters and setters must have the same 'member visibility'.
class C {
  _length = 0;

  get length() {
    return this._length;
  }

  set length(value) {
    this._length = value;
  }
}

// Since TypeScript 4.3, it is possible to have accessors with different types for getting and setting.
class Thing {
  _size = 0;

  get size(): number {
    return this._size;
  }

  set size(value: string | number | boolean) {
    let num = Number(value);

    // Don't allow NaN, Infinity, etc
    if (!Number.isFinite(num)) {
      this._size = 0;
      return;
    }

    this._size = num;
  }
}



// Class implementing an interface
// NOTE 'implements' clause is only a check that the class can be treated as the
// interface type. It doesn't change the type of the class or its methods at all.
// Classes may also implement multiple interfaces, e.g. 'class C implements A, B {'
interface Pingable {
  opt?: string;

  ping(): void;
  check(name: string): boolean;
}
class Sonar implements Pingable {
  ping() {
    console.log("ping!");
  }

  // Notice no error here
  // NOTE 'implements' clauses don't change how the class body is checked or its type inferred.
  check(s: any) {
    return s.toLowercse() === "ok";
  }
}

const c = new Sonar();
// ERROR: Property 'opt' does not exist on type 'Sonar'.
// NOTE that implementing an interface with an optional property doesn't create that property.
// c.opt = 10;



// Class extending from a base class.
// The order of class initialization, as defined by JavaScript, is:
// - The base class fields are initialized
// - The base class constructor runs
// - The derived class fields are initialized
// - The derived class constructor run
class Animal {
  move() {
    console.log("Moving along!");
  }
}
class Dog extends Animal {
  woof(times: number) {
    for (let i = 0; i < times; i++) {
      console.log("woof!");
    }
  }
}
const d = new Dog();
// Base class method
d.move();
// Derived class method
d.woof(3);



// Overriding methods
class Base {
  greet() {
    console.log("Hello, world!");
  }
}
class Derived extends Base {

  // ERROR: Property 'greet' in type 'Derived' is not assignable to the same
  // property in base type 'Base'. Type '(name: string) => void' is not
  // assignable to type '() => void'.
  // greet(name: string) {
  //   console.log(`Hello, ${name.toUpperCase()}`);
  // }
  greet(name?: string) {
    if (name === undefined) {
      super.greet();
    } else {
      console.log(`Hello, ${name.toUpperCase()}`);
    }
  }
}
const d = new Derived();
d.greet();
d.greet("reader");
const e: Base = d;
// No problem
e.greet();




```
