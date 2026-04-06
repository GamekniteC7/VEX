## Casting

Every type has built-in casting methods.

### Unsafe Casting

`.to_type()` casts a value to the target type. If the cast fails, it panics with the message:

```
file_name:line_number => Casting {type_before} to {type_after} failed
```

``` VEX
let mut x: i64 = 1000;
x.to_i32()     // x is now i32
x.to_f64()     // x is now f64
x.to_string()  // x is now String
```

### Safe Casting

`.try_to_type()` casts a value to the target type and returns `Result<T, String>` instead of panicking. The error string is `"Casting {type_before} to {type_after} failed"`:

``` VEX
let mut x: i64 = 1000;
let result: Result<i32, String> = x.try_to_i32();
```