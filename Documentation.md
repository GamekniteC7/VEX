# VEX Language Documentation

> **VEX** — _Vague Errors Excluded_  
> A general-purpose programming language designed to eliminate ambiguity and misunderstanding, without sacrificing expressiveness.

---

## File Structure

Every VEX project has one `main.vxl` file. This file contains the entry point of the program — the `main()` function. All other `.vxl` files must be attached to `main.vxl` directly or indirectly to exist in the program.

``` VEX
main(){
    // entry point
}
```

> `main()` is special. It is not declared with `fn` or any visibility modifier. Any function named `main` elsewhere — such as `pub fn main()` in another file — is treated as a regular function, not the entry point. Within `main.vxl` itself, the name `main` is reserved for the entry point only.

### Attaching Files

Files are attached using the `attach` keyword. Without attachment, a file is completely invisible to the rest of the program.

``` VEX
// main.vxl
attach math
attach utils

main() {
    ...
}
```

Attachments can be organized into an intermediate file to keep `main.vxl` clean:

``` VEX
// modules.vxl
attach math
attach utils
attach graphics
```

``` VEX
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

### Namespaces

To avoid typing full file names every time you call a function from an attached file, you can define a `NAMESPACE` block to create short aliases.

``` VEX
NAMESPACE {
    file_name1: alias1,
    file_name2: alias2
}
```

**Example:**

``` VEX
// main.vxl
attach window

NAMESPACE {
    window: w
}

main() {
    // Instead of window::function_name();
    w::function_name();
}
```

- Technically, `NAMESPACE` blocks can be placed anywhere in the file, but it is customary to place them at the top of the file, typically right after your `attach` statements.
- You can define multiple `NAMESPACE` blocks in a single file, though it is usually cleaner to group all aliases into one block.

---

## Functions

Functions are declared with the `fn` keyword:

``` VEX
fn function_name(mutability parameter_name: type) >> return_type {

}
```

- The **return value** is always the last expression inside the function (implicit return).
- `return` is only used for **early exits**.
- **Named arguments are required** when calling a function. Omitting the name is a compile-time error.
- The return type `()` represents void (no return value).
- There are no anonymous functions (lambdas) or first-class functions in VEX.

**Example:**

``` VEX
fn add(mut a: i32, b: i32) >> i32 {
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

``` VEX
fn outer() >> () {
    fn middle() >> () {
        fn inner() >> () {
            // valid
        }
    }
}
```

---

## Methods

Methods are declared with the `md` keyword:

``` VEX
md method_name(parameter_name: type) >> return_type {

}
```

- Methods follow the same return rules as functions (implicit return, `return` for early exits).
- Methods are called using **dot notation** with parentheses: the value before the dot is automatically passed as the first parameter.
- Methods **implicitly reassign** the result back to the caller variable. The caller must therefore be `mut`.
- The return type of a method does not need to match the original type — after a method call, the variable's type changes to the return type of the method.
- Named arguments are required for any additional parameters.
- `pub md` makes a method available across files.

**Example:**

``` VEX
md square(mut x: i32) >> i32 {
    x * x;
}

let mut x: i32 = 5;
x.square()      // equivalent to: x = square(x: x)
                // x is now 25
```

**Type change after method call:**

``` VEX
let mut x: i32 = 5;
x.toString()    // x is now a String
```

**With additional parameters:**

``` VEX
x.pow(b: &y)    // x is auto-referenced, y must be explicitly referenced
```

**Using a method as an expression:**

``` VEX
println(x.square())
// equivalent to:
x.square();
println(x);
```

**Passing a reference after a method call:**

``` VEX
add(a: &x.square(), b: 5)
// equivalent to:
x.square();
add(a: &x, b: 5);
```

### Method Chaining

Methods can be chained — each method is applied to the result of the previous one:

``` VEX
x.square().double().abs()
// equivalent to:
x.square();
x.double();
x.abs();
```

### Method Types

A method's applicable type is determined by its first parameter's type. A method declared with `(x: i32)` will only work on `i32`.

### Method Naming

Method names must be **unique within a file** but can share names across files. When calling a method from a specific file, use:

``` VEX
x.file_name::method_name()
```

If a namespace alias is defined for the file, you can use the alias instead:

``` VEX
x.alias::method_name()
```

If a method name exists in multiple attached files and no file is specified, VEX looks for the method in the **current file**. If it isn't found there, a compile-time error is raised requiring explicit file qualification.

> Method overloading (same name, different types) is **not supported** within a file.

---

## Variables

Variables are declared with the `let` keyword:

``` VEX
let name: type = value;
```

### Mutability

- `let name: type = value;` → **immutable** (constant). Reassigning transfers ownership but does not allow mutation.
- `let mut name: type = value;` → **mutable**.

``` VEX
let x: int = 5;         // immutable
let mut y: i32 = 5;     // mutable
```

Attempting to mutate an immutable variable is a compile-time error. Reassigning an immutable variable transfers ownership — the old binding becomes invalid after the move.

### Type Inference

Inside function or method parameters, `name: type` doubles as a variable declaration. Outside of parameters, type inference is supported — the compiler can infer the type from the assigned value:

``` VEX
let added = add(a: 5, b: 7);  // type inferred from return type of add()
```

---

## Data Types

### Integers

|Type|Description|
|---|---|
|`i8`, `i16`, `i32`, `i64`, `i128`|Signed integers of explicit size|
|`int`|Signed integer — compiler picks the smallest fitting type at declaration (immutable variables only)|
|`u8`, `u16`, `u32`, `u64`, `u128`|Unsigned integers of explicit size|
|`uint`|Unsigned integer — compiler picks the smallest fitting type at declaration (immutable variables only)|

### Floats

|Type|Description|
|---|---|
|`f8`, `f16`, `f32`, `f64`, `f128`|Floats of explicit size|
|`float`|Float — compiler picks the smallest fitting type at declaration (immutable variables only)|

### Other Types

|Type|Description|
|---|---|
|`bool`|Boolean (`true` / `false`)|
|`String`|UTF-8 string|
|`()`|None — the absence of a value|

### None / `()`

`()` is VEX's none type, representing the absence of a value. It is used in two contexts:

**As a return type** — a function that returns nothing:

``` VEX
fn hello() >> () {
    println("Hello!");
}
```

**As a value** — a variable that has no value yet:

``` VEX
let x: int = ();   // x is an int holding no value
let y: int;        // equivalent — y is also an int holding no value
```

#### Checking for None

Use `==` to check if a variable is `()`:

``` VEX
if x == () {
    // handle the none case
} else {
    // x is guaranteed to have a value here
}
```

#### Compiler Enforcement

The compiler tracks `()` values and raises a **compile-time error** if a `()` value is used in an operation without first being checked:

``` VEX
let x: int;
let y: int = x + 5;   // compile-time error — x might be ()

if x == () {
    // handle none
} else {
    let y: int = x + 5;   // valid — x is guaranteed not to be ()
}
```

Passing a `()` value to a function is allowed. The compile-time error is only triggered at the point where the value is actually used inside the function:

``` VEX
fn add(mut a: i32, b: i32) >> i32 {
    a + b;   // compile-time error if a or b might be ()
}

let x: int;
add(a: x, b: 5);   // fine — passing () is allowed
```

### Arrays

Arrays have a **fixed length** defined at declaration:

``` VEX
array<type, length>
```

### Vectors

Vectors are like arrays with a **dynamic length**:

``` VEX
vector<type>
```

---

## Structs

Structs are custom data types that group named fields together under one type.

### Declaration

``` VEX
struct Rectangle {
    mut width: f32,
    height: f32,
}
```

- Fields are **immutable by default**.
- Use `mut` before a field to make it mutable across all instances.
- Use `pub struct` to make the struct available across files.

### Default Values

Default field values are defined separately using `.default()`:

``` VEX
Rectangle.default() {
    width: 1.0,
    height: 1.0,
}
```

### Instantiation

``` VEX
// explicit values
let r: Rectangle = Rectangle(width: 5.0, height: 3.0);

// all defaults
let r2: Rectangle = Rectangle();

// partial defaults — height: with nothing after colon means use default
let r3: Rectangle = Rectangle(width: 5.0, height:);

// override a mut field to immutable at instantiation
let r4: Rectangle = Rectangle(imut width: 5.0, height: 3.0);
```

> Leaving a field empty (e.g. `height:`) when no default is defined is a **compile-time error**.

### Field Access

Fields are accessed using the `'` operator:

``` VEX
r'width    // 5.0
r'height   // 3.0
```

Nested struct field access chains naturally:

``` VEX
line'start'width
```

### Mutability

- Field mutability is defined in the struct declaration with `mut`.
- At instantiation, a `mut` field can be forced immutable using `imut`.
- Each struct's own field mutability rules apply independently — nesting does not affect inner struct mutability.

### Methods on Struct Fields

Methods can be called directly on struct fields. The field must be `mut` since the result is reassigned back to it:

``` VEX
r'width.square()
// equivalent to:
r'width = square(x: r'width);
```

---

## Enums

Enums define a type with a fixed set of possible named variants. Variants can optionally carry named data fields.

### Declaration

``` VEX
enum Shape {
    Circle(radius: f32);
    Rectangle(width: f32, height: f32);
    Empty();
}
```

- Variants are separated by **semicolons**.
- Variants with no data use empty parentheses `()`.
- Use `pub enum` to make the enum available across files.
- Enums can be nested — a variant can carry another enum as its data.

### Instantiation

``` VEX
let s: Shape = Circle(radius: 5.0);
let e: Shape = Empty();
```

### Field Access

Enum variant data is accessed using the `->` operator:

``` VEX
s->radius   // 5.0
```

### Methods

Methods work on enum values just like any other type, determined by the first parameter's type.

### Match Statement

A match statement handles each possible variant. **All variants must be covered** — a non-exhaustive match is a compile-time error. A `_` fallback case is supported:

``` VEX
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

``` VEX
match area(s: Shape) >> f32 {
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

``` VEX
let x: i32 = 5;
let y: i32 = x;      // ownership transferred
// x is now invalid
```

### Copying

To copy a value without transferring ownership, use `(copy)`:

``` VEX
let x: i32 = 5;
let y: i32 = (copy) x;   // y gets a copy of x, x stays valid
// both x and y are 5
```

### Immutable References

`&x` creates a live reference to `x`. The reference tracks changes to `x` — if `x` changes, the reference reflects the new value. The original `x` stays valid and its ownership is not transferred.

``` VEX
let mut x: i32 = 5;
let y: i32 = &x;    // y is a live reference to x
x + 2;              // x is now 7
// y is also 7
```

Unlike `(copy)`, a reference is not a new independent variable — it is a live link to the original.

### Mutable References

`mut&x` creates a mutable live reference. Changing the mutable reference changes the original variable and all other references to it. The original variable must be `mut`, and the holder of a `mut&` reference must also be `mut`.

``` VEX
let mut x: i32 = 5;
let mut y: i32 = mut&x;   // y is a mutable reference to x

y + 2;    // x is now 7, all references to x are now 7
x + 3;    // x is now 10, y is also 10
```

There can be as many mutable or immutable references to a variable as you want. Changing the original or any mutable reference updates all of them.

### Dereferencing

To remove a reference from a variable, use `deref()` or `cderef()`:

**`deref()`** — eliminates the reference and drops the variable entirely:

``` VEX
let x: i32 = 5;
let y: i32 = &x;

y.deref();       // y is eliminated and dropped
println(y);      // compile-time error: y doesn't exist
```

**`cderef()`** — eliminates the reference and copies the current value of the original into the holder. Requires the holder to be `mut`. The variable takes the mutability it had before becoming a reference:

``` VEX
let mut x: i32 = 5;
let mut y: i32 = &x;

x + 2;           // x is now 7
y.cderef();      // y is now an independent variable with value 7
println(y);      // valid — y is now 7
```

### References and Ownership

All references to a variable must be eliminated with `deref()` or `cderef()` before the variable's ownership can be transferred:

``` VEX
let x: i32 = 5;
let y: i32 = &x;

let z: i32 = x;   // compile-time error — y still holds a reference to x
y.deref();
let z: i32 = x;   // valid — no more references to x
```

### References in Function Parameters

`&x` and `mut&x` can also be passed as function parameters. A parameter reference is only live for the duration of the function call:

``` VEX
fn double(mut x: &i32) >> i32 {
    x * 2;
}

double(x: &x);   // x stays valid after the call
```

> A reference **cannot be returned** from a function — `&type` is not a valid return type and causes a compile-time error.

### Full Comparison

|Syntax|Tracks changes|Can change original|Original must be `mut`|Holder must be `mut`|
|---|---|---|---|---|
|`(copy) x`|No|No|No|No|
|`&x`|Yes|No|No|No|
|`mut&x`|Yes|Yes|Yes|Yes|

|Syntax|Context|Effect|
|---|---|---|
|`let y = x`|variable declaration|ownership transferred, x invalid|
|`let y = (copy) x`|variable declaration|y gets copy, x stays valid|
|`let y = &x`|variable declaration|live reference, x stays valid|
|`let mut y = mut&x`|variable declaration|mutable live reference, x stays valid|
|`fn(param: x)`|function parameter|ownership transferred, x invalid|
|`fn(param: &x)`|function parameter|live reference for duration of call, x stays valid|
|`fn(param: mut&x)`|function parameter|mutable reference for duration of call|
|`fn(param: (copy) x)`|function parameter|copy passed, x stays valid|

### Compiler Optimization

When `(copy) x` is passed as a parameter and the original `x` is never mutated during the call, the compiler may internally optimize this into a reference — since the result would be identical. The compiler will produce a warning suggesting the optimization.

---

## Operators

### Logical Operators: `&&` and `||`

VEX's `&&` and `||` operators have a convenience feature: if a comparison operator (`==`, `!=`, `<`, `>`, `<=`, `>=`) appears on one side of a `&&` or `||` but not the other, the compiler **distributes** the comparison to the incomplete side automatically.

``` VEX
(x % 3) && (x % 5) == 0
// becomes:
(x % 3 == 0) && (x % 5 == 0)
```

``` VEX
(x % 3) || (x % 5) != 0
// becomes:
(x % 3 != 0) || (x % 5 != 0)
```

> If both sides already have a comparison operator, no distribution occurs.

### Mathematical Operator Shorthand

Writing a mathematical expression as a standalone statement implicitly assigns the result back to the **first variable** in the expression:

``` VEX
x * x;
// becomes:
x = x * x;
```

With two different variables, the first one is changed:

``` VEX
x * y + z;
// becomes:
x = (x * y + z);
```

> If a literal comes first (e.g. `5 * x`), this is a **compile-time error**. You must write `x = 5 * x` explicitly.

---

## Control Flow

### If Statement

``` VEX
if condition {

}
else if condition {

}
else {

}
```

### If-Except Statement

The `if-except` statement executes a block if a condition is true, **except** when a second condition is also true. The `if` part is evaluated first; if it passes, the `except` part is then checked. If the `except` condition is true, the block is **not** executed. The optional `else except` block executes only when the `except` condition is true. You can still use a normal `else`.

``` VEX
if condition except exception_condition {

}
else except{   // executes only when except is true

}
else {         // executes when all of the above are false (normal else)

}
```

### For Loop

``` VEX
for i in 0..99 {

}
```

### While Loop

``` VEX
while condition {

}
```

### Loop (Infinite)

`loop` is equivalent to `while true`:

``` VEX
loop {

}
```

---

## Code Examples

### Hello World

``` VEX
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

``` VEX
// main.vxl
main(){
    let added: i32 = add(a: 5, b: 7);
}

fn add(mut a: i32, b: i32) >> i32 {
    a + b;
}
```

**Output:** `12`

---

### FizzBuzz

``` VEX
// main.vxl
main(){
    for i in 1..100 {
        println(FizzBuzz(x: &i));
    }
}

fn FizzBuzz(mut x: i32) >> String {
    if (x % 3) && (x % 5) == 0 {
        return "FizzBuzz!";
    }
    else if x % 3 == 0 {
        return "Fizz!";
    }
    else if x % 5 == 0 {
        return "Buzz!";
    }
    else {
        return x.toString();
    }
}
```

**Output:** FizzBuzz sequence from 1 to 100.

---

### Methods

``` VEX
// main.vxl
main(){
    let mut x: i32 = 5;
    println(x.square());
}

md square(mut x: i32) >> i32 {
    x * x;
}
```

**Output:** `25`
