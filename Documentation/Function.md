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