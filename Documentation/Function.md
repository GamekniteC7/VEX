## Functions

Functions are declared with the `fn` keyword:

``` VEX
fn function_name(mutability parameter_name: type) >> return_type {

}
```

- The `mutability` modifier can be empty (immutable), `mut` (mutable value), or `tmut` (mutable type).
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

fn process(tmut item: i32) >> String {
    // ...
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

### Generic Functions

Functions can be generic over types using the `<T>` syntax. The type must be specified explicitly at the call site:

``` VEX
fn first<T>(list: vector<T>) >> Option<T> {
    // ...
}

// calling a generic function
let x: i32 = first<i32>(list: numbers);
```
### Calling Functions as Methods (Uniform Function Call Syntax)

Any function can be called as a method using dot notation on its first argument. For full details on this behavior, see [Methods](./Methods.md).

``` VEX
fn square(mut x: i32) >> i32 {
    x * x;
}

let mut x: i32 = 5;

// Call as a function:
x = square(x: x);

// Call as a method:
x.square(); // x is now 25, works like a method
```
