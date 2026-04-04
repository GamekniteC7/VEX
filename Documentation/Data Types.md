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