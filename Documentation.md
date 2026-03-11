# VEX Language Documentation

> **VEX** — _Vague Errors Excluded_  
> A general-purpose programming language designed to eliminate ambiguity and misunderstanding, without sacrificing expressiveness.

---

## File Structure

Every VEX project has one `main.vxl` file. This file contains the entry point of the program — the `main()` function. All other `.vxl` files must be attached to `main.vxl` directly or indirectly to exist in the program.

```
main(){
    // entry point
}
```

> `main()` is special. It is not declared with `fn` or any visibility modifier. Any function named `main` elsewhere — such as `pub fn main()` in another file — is treated as a regular function, not the entry point. Within `main.vxl` itself, the name `main` is reserved for the entry point only.

### Attaching Files

Files are attached using the `attach` keyword. Without attachment, a file is completely invisible to the rest of the program.

```
// main.vxl
attach math
attach utils

main() {
    ...
}
```

Attachments can be organized into an intermediate file to keep `main.vxl` clean:

```
// modules.vxl
attach math
attach utils
attach graphics
```

```
// main.vxl
attach modules

main() {
    ...
}
```

- `attach` only makes `pub` declarations from the attached file accessible.
- Attached declarations are always called with explicit file qualification: `file_name::function_name`.
- The order of `attach` statements does not matter.
- Circular attachments are a **compile-time error**.

---

## Functions

Functions are declared with the `fn` keyword:

```
fn function_name(parameter_name: type) >> return_type {

}
```

- The **return value** is always the last expression inside the function (implicit return).
- `return` is only used for **early exits**.
- **Named arguments are required** when calling a function. Omitting the name is a compile-time error.
- The return type `()` represents void (no return value).

**Example:**

```
fn add(a: int, b: int) >> int {
    a + b;
}

// calling the function
add(a: 5, b: 7);
```

### Visibility

- No keyword → the function is scoped to where it is defined and all sub-scopes.
- `pub fn` → the function is available across files (when attached to a module).

### Nested Functions

Functions can be nested inside other functions to arbitrary depth. A nested function has access to its own scope and all parent scopes, but not to sibling or outer-sibling scopes.

```
fn outer() >> () {
    fn middle() >> () {
        fn inner() >> () {
            // valid
        }
    }
}
```

### First-Class Functions

Functions can be passed as arguments. The full signature of the function is written inline as the parameter type:

```
fn apply(fn: function_name(x: int) >> int, x: int) >> int {
    fn(x: x);
}
```

Functions from other files are referenced as `file_name::function_name`:

```
fn apply(fn: file_name::function_name(x: int) >> int, x: int) >> int {
    fn(x: x);
}
```

> There are no anonymous functions (lambdas) in VEX.

---

## Methods

Methods are declared with the `md` keyword:

```
md method_name(parameter_name: type) >> return_type {

}
```

- Methods follow the same return rules as functions (implicit return, `return` for early exits).
- Methods are called using **dot notation** with parentheses: the value before the dot is automatically passed as the first parameter.
- Methods **implicitly reassign** the result back to the caller variable. The caller must therefore be `mut`.
- The return type of a method must not necessarily match the original type — after a method call, the variable's type changes to the return type of the method.
- Named arguments are required for any additional parameters.
- `pub md` makes a method available across files.

**Example:**

```
md square(x: int) >> int {
    x * x;
}

let mut int x = 5;
x.square()      // equivalent to: x = square(x: x)
                // x is now 25
```

**Type change after method call:**

```
let mut int x = 5;
x.toString()    // x is now a String
```

**With additional parameters:**

```
x.pow(b: &y)    // x is auto-referenced, y must be explicitly referenced
```

**Using a method as an expression:**

```
println(x.square())
// equivalent to:
x.square();
println(x);
```

**Passing a reference after a method call:**

```
add(a: &x.square(), b: 5)
// equivalent to:
x.square();
add(a: &x, b: 5);
```

### Method Chaining

Methods can be chained — each method is applied to the result of the previous one:

```
x.square().double().abs()
// equivalent to:
x.square();
x.double();
x.abs();
```

### Method Types

A method's applicable type is determined by its first parameter's type. A method declared with `(x: int)` will only work on integers.

### Method Naming

Method names must be **unique within a file** but can share names across files. When calling a method from a specific file, use:

```
x.file_name::method_name()
```

If a method name exists in multiple attached files and no file is specified, VEX looks for the method in the **current file**. If it isn't found there, a compile-time error is raised requiring explicit file qualification.

> Method overloading (same name, different types) is **not supported** within a file.

---

## Variables

Variables are declared with the `let` keyword:

```
let type name = value;
```

### Mutability

- `let type name = value;` → **immutable** (constant). Reassigning transfers ownership but does not allow mutation.
- `let mut type name = value;` → **mutable**.

```
let int x = 5;         // immutable
let mut int y = 5;     // mutable
```

Attempting to mutate an immutable variable is a compile-time error. Reassigning an immutable variable transfers ownership — the old binding becomes invalid after the move, as in Rust.

### Type Inference

Inside function or method parameters, `name: type` doubles as a variable declaration. Outside of parameters, type inference is supported — the compiler can infer the type from the assigned value:

```
added = add(a: 5, b: 7);  // type inferred from return type of add()
```

---

## Types

### Integers

|Type|Description|
|---|---|
|`i8`, `i16`, `i32`, `i64`|Signed integers of explicit size|
|`int`|Signed integer — compiler picks the smallest fitting type|
|`u8`, `u16`, `u32`, `u64`|Unsigned integers of explicit size|
|`uint`|Unsigned integer — compiler picks the smallest fitting type|

### Floats

|Type|Description|
|---|---|
|`f8`, `f16`, `f32`, `f64`|Floats of explicit size|
|`float`|Float — compiler picks the smallest fitting type|

### Other Types

|Type|Description|
|---|---|
|`bool`|Boolean (`true` / `false`)|
|`String`|UTF-8 string|
|`()`|None — the absence of a value|

### None / `()`

`()` is VEX's none type, representing the absence of a value. It is used in two contexts:

**As a return type** — a function that returns nothing:

```
fn hello() >> () {
    println("Hello!");
}
```

**As a value** — a variable that has no value yet:

```
let int x = ();   // x is an int holding no value
let int y;        // equivalent — y is also an int holding no value
```

#### Checking for None

Use `==` to check if a variable is `()`:

```
if (x == ()) {
    // handle the none case
} else {
    // x is guaranteed to have a value here
}
```

#### Compiler Enforcement

The compiler tracks `()` values and raises a **compile-time error** if a `()` value is used in an operation without first being checked:

```
let int x;
let int y = x + 5;   // compile-time error — x might be ()

if (x == ()) {
    // handle none
} else {
    let int y = x + 5;   // valid — x is guaranteed not to be ()
}
```

Passing a `()` value to a function is allowed. The compile-time error is only triggered at the point where the value is actually used inside the function:

```
fn add(a: int, b: int) >> int {
    a + b;   // compile-time error if a or b might be ()
}

let int x;
add(a: x, b: 5);   // fine — passing () is allowed
```

### Arrays

Arrays have a **fixed length** defined at declaration:

```
array<type, length>
```

### Vectors

Vectors are like arrays with a **dynamic length**:

```
vector<type>
```

---

## Structs

Structs are custom data types that group named fields together under one type.

### Declaration

```
struct Rectangle {
    mut width: float,
    height: float,
}
```

- Fields are **immutable by default**.
- Use `mut` before a field to make it mutable across all instances.
- Use `pub struct` to make the struct available across files.

### Default Values

Default field values are defined separately using `.default()`:

```
Rectangle.default() {
    width: 1.0,
    height: 1.0,
}
```

### Instantiation

```
// explicit values
let Rectangle r = Rectangle(width: 5.0, height: 3.0);

// all defaults
let Rectangle r2 = Rectangle();

// partial defaults — height: with nothing after colon means use default
let Rectangle r3 = Rectangle(width: 5.0, height:);

// override a mut field to immutable at instantiation
let Rectangle r4 = Rectangle(imut width: 5.0, height: 3.0);
```

> Leaving a field empty (e.g. `height:`) when no default is defined is a **compile-time error**.

### Field Access

Fields are accessed using the `^` operator:

```
r^width    // 5.0
r^height   // 3.0
```

Nested struct field access chains naturally:

```
line^start^width
```

### Mutability

- Field mutability is defined in the struct declaration with `mut`.
- At instantiation, a `mut` field can be forced immutable using `imut`.
- Each struct's own field mutability rules apply independently — nesting does not affect inner struct mutability.

### Methods on Struct Fields

Methods can be called directly on struct fields. The field must be `mut` since the result is reassigned back to it:

```
r^width.square()
// equivalent to:
r^width = square(x: r^width);
```

---

## Enums

Enums define a type with a fixed set of possible named variants. Variants can optionally carry named data fields.

### Declaration

```
enum Shape {
    Circle(radius: float);
    Rectangle(width: float, height: float);
    Empty();
}
```

- Variants are separated by **semicolons**.
- Variants with no data use empty parentheses `()`.
- Use `pub enum` to make the enum available across files.
- Enums can be nested — a variant can carry another enum as its data.

### Instantiation

```
let s = Circle(radius: 5.0);
let e = Empty();
```

### Field Access

Enum variant data is accessed using the `->` operator:

```
s->radius   // 5.0
```

### Methods

Methods work on enum values just like any other type, determined by the first parameter's type.

### Match Statement

A match statement handles each possible variant. **All variants must be covered** — a non-exhaustive match is a compile-time error. A `_` fallback case is supported:

```
match s {
    Circle(radius: r) => { ... }
    Rectangle(width: w, height: h) => { ... }
    Empty() => { ... }
    _ => { ... }   // optional fallback
}
```

Data carried by a variant is unpacked and bound to a new name in the match arm. For example `Circle(radius: r)` binds the `radius` field to `r` for use inside that arm.

### Match as Expression

A match can also act as a special function that takes an enum and returns a value based on the variant. It is called like a regular function with named arguments:

```
match area(s: Shape) >> float {
    Circle(radius: r) => { r * r * 3.14; }
    Rectangle(width: w, height: h) => { w * h; }
    Empty() => { return; }
}

// calling it
area(s: myShape);
```

- The return value is the last expression in each arm (implicit return), consistent with functions and methods.
- `return;` with no value means early exit with no return — used for variants that don't produce a meaningful value.
- All variants must still be covered.

---

## Ownership, Copying & References

VEX has a precise system for controlling how values are passed and shared.

### Ownership Transfer

By default, assigning a variable to another transfers ownership. The original binding becomes invalid:

```
let int x = 5;
let y = x;      // ownership transferred
// x is now invalid
```

### Copying

To copy a value without transferring ownership, use `(copy)`:

```
let int x = 5;
let mut y = (copy) x;   // y gets a copy of x, x stays valid
```

### References in Function Parameters

`&` is **only valid in function and method parameters** — not in variable declarations. Passing `&x` passes a live reference to `x` for the duration of the function call. After the call, the reference is gone. The original `x` stays valid.

```
fn double(x: &int) >> int {
    x * 2;
}

double(x: &x);   // x stays valid after the call
```

> References are only live for the duration of the function call. A reference **cannot be returned** from a function — `&type` is not a valid return type and causes a compile-time error.

### Copying in Function Parameters

To pass a copy as a parameter instead of a reference:

```
fn double(x: (copy) int) >> int {
    x * 2;
}
```

### Full Comparison

|Syntax|Context|Effect|
|---|---|---|
|`let y = x`|variable declaration|ownership transferred, x invalid|
|`let y = (copy) x`|variable declaration|y gets copy, x stays valid|
|`fn(param: x)`|function parameter|ownership transferred, x invalid|
|`fn(param: &x)`|function parameter|live reference for duration of call, x stays valid|
|`fn(param: (copy) x)`|function parameter|copy passed, x stays valid|

### Compiler Optimization

When `(copy) x` is passed as a parameter and the original `x` is never mutated during the call, the compiler may internally optimize this into a reference — since the result would be identical. The programmer never needs to manage this manually.

### References and Mutation

Holding a reference to a variable while mutating or retyping that variable is a **compile-time error**:

```
let mut int x = 5;
fn modify(a: &int, b: &int) >> int { ... }
modify(a: &x, b: &x);   // error if x is mutated inside modify
```

---

## Operators

### Logical Operators: `&&` and `||`

VEX's `&&` and `||` operators have a convenience feature: if a comparison operator (`==`, `!=`, `<`, `>`, `<=`, `>=`) appears on one side of a `&&` or `||` but not the other, the compiler **distributes** the comparison to the incomplete side automatically.

```
(x % 3) && (x % 5) == 0
// becomes:
(x % 3 == 0) && (x % 5 == 0)
```

```
(x % 3) || (x % 5) != 0
// becomes:
(x % 3 != 0) || (x % 5 != 0)
```

> If both sides already have a comparison operator, no distribution occurs.

---

## Control Flow

### If Statement

```
if (condition) {

}
else if (condition) {

}
else {

}
```

### If-Except Statement

The `if-except` statement executes a block if a condition is true, **except** when a second condition is also true. The `if` part is evaluated first; if it passes, the `except` part is then checked. If the `except` condition is true, the block is **not** executed.

```
if (condition) except (exception_condition) {

}
```

### For Loop

```
for (i = 0; i <= 99; i++) {

}
```

### While Loop

```
while (condition) {

}
```

### Loop (Infinite)

`loop` is equivalent to `while (true)`:

```
loop {

}
```

---

## Code Examples

### Hello World

```
// main.vxl
main(){
    hello_world();
}

fn hello_world() >> () {
    println("Hello, World!");
}
```

**Output:** `Hello, World!`

---

### Addition

```
// main.vxl
main(){
    added = add(a: 5, b: 7);
}

fn add(a: int, b: int) >> int {
    a + b;
}
```

**Output:** `12`

---

### FizzBuzz

```
// main.vxl
main(){
    for (i = 0; i <= 99; i++) {
        println(FizzBuzz(x: &i));
    }
}

fn FizzBuzz(x: int) >> String {
    if ((x % 3) && (x % 5) == 0) {
        return "FizzBuzz!";
    }
    else if (x % 3 == 0) {
        return "Fizz!";
    }
    else if (x % 5 == 0) {
        return "Buzz!";
    }
    else {
        return (x.toString());
    }
}
```

**Output:** FizzBuzz sequence from 0 to 99.

---

### Methods

```
// main.vxl
main(){
    let mut int x = 5;
    println(x.square());
}

md square(x: int) >> int {
    x * x;
}
```

**Output:** `25`
