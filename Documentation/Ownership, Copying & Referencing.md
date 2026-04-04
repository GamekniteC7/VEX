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