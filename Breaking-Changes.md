These changes list where implementation differs between versions as the spec and compiler are simplified and inconsistencies are corrected.

> For breaking changes to the compiler/services API, please check the [[API Breaking Changes]] page.

# TypeScript 1.5

For full list of breaking changes see the [breaking change issues](https://github.com/Microsoft/TypeScript/issues?q=is%3Aissue+milestone%3A%22TypeScript+1.5%22+label%3A%22breaking+change%22).

#### Referencing `arguments` in arrow functions is not allowed
This is an alignment with the ES6 semantics of arrow functions. Previously arguments within an arrow function would bind to the arrow function arguments. As per [ES6 spec draft](http://wiki.ecmascript.org/doku.php?id=harmony:specification_drafts) 9.2.12, arrow functions do not have an arguments objects. 
In TypeScript 1.5, the use of arguments object in arrow functions will be flagged as an error to ensure your code ports to ES6 with no change in semantics.

**Example:**
```ts
function f() {
    return () => arguments; // Error: The 'arguments' object cannot be referenced in an arrow function. 
}
```

**Recommendations:**
```ts
// 1. Use named rest args 
function f() {
    return (...args) => { args; }
}

// 2. Use function expressions instead
function f() {
    return function(){ arguments; }
}
```

#### Enum reference in-lining changes
For regular enums, pre 1.5, the compiler *only* inline constant members, and a member was only constant if its initializer was a literal. That resulted in inconsistent behavior depending on whether the enum value is initalized with a literal or an expression. Starting with Typescript 1.5 all non-const enum members are not inlined.

**Example:**
```ts
var x = E.a;  // previously inlined as "var x = 1; /*E.a*/"

enum E {
   a = 1
}
```

**Recommendation:**
Add the `const` modifier to the enum declaration to ensure it is consistently inlined at all consumption sites.

For more details see issue [#2183](https://github.com/Microsoft/TypeScript/issues/2183).


#### Contextual type flows through Super and parenthesized expressions
Prior to this release, contextual types did not flow through parenthesized expressions. This has forced explicit type casts, especially in cases where parentheses are *required* to make an expression parse.

In the examples below, `m` will have a contextual type, where previously it did not.
```ts
var x: SomeType = (n) => ((m) => q)); 
var y: SomeType = t ? (m => m.length) : undefined; 

class C extends CBase<string> {
    constructor() {
        super({
            method(m) { return m.length; }
        });
    }
}
```

See issues [#1425](https://github.com/Microsoft/TypeScript/issues/1425) and [#920](https://github.com/Microsoft/TypeScript/issues/920) for more details.

# TypeScript 1.4

For full list of breaking changes see the [breaking change issues](https://github.com/Microsoft/TypeScript/issues?q=is%3Aissue+milestone%3A%22TypeScript+1.4%22+label%3A%22breaking+change%22).

See [issue #868](https://github.com/Microsoft/TypeScript/issues/868) for more details about breaking changes related to Union Types

#### Multiple Best Common Type Candidates
Given multiple viable candidates from a Best Common Type computation we now choose an item (depending on the compiler's implementation) rather than the first item.

```ts
var a: { x: number; y?: number };
var b: { x: number; z?: number };

// was { x: number; z?: number; }[]
// now { x: number; y?: number; }[]
var bs = [b, a]; 
```

This can happen in a variety of circumstances. A shared set of required properties and a disjoint set of other properties (optional or otherwise), empty types, compatible signature types (including generic and non-generic signatures when type parameters are stamped out with ```any```).

**Recommendation**
Provide a type annotation if you need a specific type to be chosen
```ts
var bs: { x: number; y?: number; z?: number }[] = [b, a];
```

#### Generic Type Inference
Using different types for multiple arguments of type T is now an error, even with constraints involved:

```ts
declare function foo<T>(x: T, y:T): T;
var r = foo(1, ""); // r used to be {}, now this is an error
```
With constraints:

```ts
interface Animal { x }
interface Giraffe extends Animal { y }
interface Elephant extends Animal { z }
function f<T extends Animal>(x: T, y: T): T { return undefined; }
var g: Giraffe;
var e: Elephant;
f(g, e);
```

See https://github.com/Microsoft/TypeScript/pull/824#discussion_r18665727 for explanation.

**Recommendations**
Specify an explicit type parameter if the mismatch was intentional:
```ts
var r = foo<{}>(1, ""); // Emulates 1.0 behavior
var r = foo<string|number>(1, ""); // Most useful
var r = foo<any>(1, ""); // Easiest
f<Animal>(g, e);
```
*or* rewrite the function definition to specify that mismatches are OK:
```ts
declare function foo<T,U>(x: T, y:U): T|U;
function f<T extends Animal, U extends Animal>(x: T, y: U): T|U { return undefined; }
```

#### Generic Rest Parameters 
You cannot use heterogeneous argument types anymore:

```ts
function makeArray<T>(...items: T[]): T[] { return items; }
var r = makeArray(1, ""); // used to return {}[], now an error
```
Likewise for `new Array(...)`

**Recommendations**
Declare a back-compat signature if the 1.0 behavior was desired:
```ts
function makeArray<T>(...items: T[]): T[];
function makeArray(...items: {}[]): {}[];
function makeArray<T>(...items: T[]): T[] { return items; }
```

#### Overload Resolution with Type Argument Inference

```ts
var f10: <T>(x: T, b: () => (a: T) => void, y: T) => T;
var r9 = f10('', () => (a => a.foo), 1); // r9 was any, now this is an error
```

**Recommendations**
Manually specify a type parameter
```ts
var r9 = f10<any>('', () => (a => a.foo), 1);
```

#### Strict Mode Parsing for Class Declarations and Class Expressions
ECMAScript 2015 Language Specification (ECMA-262 6<sup>th</sup> Edition) specifies that *ClassDeclaration* and *ClassExpression* are strict mode productions. 
Thus, additional restrictions will be applied when parsing a class declaration or class expression. 

Examples:

```ts
class implements {}  // Invalid: implements is a reserved word in strict mode
class C {
    foo(arguments: any) {   // Invalid: "arguments" is not allow as a function argument
        var eval = 10;      // Invalid: "eval" is not allowed as the left-hand-side expression
        arguments = [];     // Invalid: arguments object is immutable
	}
}
```
For complete list of strict mode restrictions, please see Annex C - The Strict Mode of ECMAScript of ECMA-262 6<sup>th</sup> Edition.


# TypeScript 1.1

For full list of breaking changes see the [breaking change issues](https://github.com/Microsoft/TypeScript/issues?q=is%3Aissue+milestone%3A%22TypeScript+1.1%22+label%3A%22breaking+change%22+).

## Working with null and undefined in ways that are observably incorrect is now an error

Examples: 

```TypeScript
var ResultIsNumber17 = +(null + undefined);
// Operator '+' cannot be applied to types 'undefined' and 'undefined'.

var ResultIsNumber18 = +(null + null);
// Operator '+' cannot be applied to types 'null' and 'null'.

var ResultIsNumber19 = +(undefined + undefined);
// Operator '+' cannot be applied to types 'undefined' and 'undefined'.
```

Similarly, using null and undefined directly as objects that have methods now is an error

Examples:

```TypeScript
null.toBAZ();

undefined.toBAZ();
```
