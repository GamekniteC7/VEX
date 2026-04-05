## Error Handling

VEX has three distinct concepts for handling the absence of values and errors, each serving a different purpose.

### `>> ()` — Void

A function declared with `>> ()` **never** returns a value. It is purely for side effects:

``` VEX
fn hello() >> () {
    println("Hello!");
}
```

### `Option<T>` — Value or None

A function returning `Option<T>` should return a value of type `T` but might return `()`. This is distinct from `>> ()` — the function *intends* to return a value but cannot always do so:

``` VEX
fn find(list: vector<i32>, target: i32) >> Option<i32> {
    // returns the index if found, () if not
}
```

Since `Option<T>` can return `()`, the compiler enforces that you check for `()` before using the value, consistent with VEX's none-checking rules.

### `Result<T, E>` — Value or Error

A function returning `Result<T, E>` either returns a value of type `T` wrapped in `Ok`, or an error of type `E` wrapped in `Err`:

``` VEX
fn divide(a: f32, b: f32) >> Result<f32, String> {
    if b == 0.0 {
        Err("Division by zero");
    } else {
        Ok(a);
    }
}
```

### The `?` Operator

The `?` operator works on both `Option<T>` and `Result<T, E>`. It propagates the failure case early up the call stack, identical to Rust:

``` VEX
// With Result
fn do_something() >> Result<f32, String> {
    let result: f32 = divide(a: 5.0, b: 0.0)?;  // returns Err early if failed
    Ok(result);
}

// With Option
fn do_something() >> Option<i32> {
    let result: i32 = find(list: myList, target: 5)?;  // returns () early if not found
    Option(result);
}
```

### `.extract_value()`

`.extract_value()` extracts the value from an `Option<T>` or `Result<T, E>`. If the value is `()` or `Err`, it triggers a `panic!`:

``` VEX
// panics if ()
let result: i32 = find(list: myList, target: 5).extract_value();

// panics if Err
let result: f32 = divide(a: 5.0, b: 0.0).extract_value();
```

### `panic!`

`panic!` triggers an unrecoverable error and crashes the program. The `!` is part of the syntax — it is not a macro, but emphasizes the severity:

``` VEX
panic!("something went terribly wrong");
```

### Summary

|Return type|Meaning|Failure case|
|---|---|---|
|`>> ()`|Never returns a value|—|
|`>> Option<T>`|Should return `T`, might return `()`|`()`|
|`>> Result<T, E>`|Returns `T` or an error `E`|`Err(E)`|