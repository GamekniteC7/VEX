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

Arrays can be sliced using range syntax:

``` VEX
array[1..3]   // elements from index 1 to 3 inclusive
```

### Vectors

Vectors are like arrays with a **dynamic length**:

``` VEX
vector<type>
```

### Range

A range represents an inclusive sequence of numeric values. Ranges can contain integers or floats, specified by `range<type>`:

``` VEX
let r: range<i32> = 0..100;    // inclusive range from 0 to 100
let r2: range<f32> = 0.0..1.0; // float range
```

The compiler parses ranges left to right — it reads the first number, checks for `..`, then reads the second number. Both sides of `..` must be the same type, enforced by the type checker.

Ranges can be used in `for` loops, as slice indices, and stored as variables.

#### Range Methods

All methods that modify the range require the range to be `mut`:

| Method | Description | Requires `mut` |
|--------|-------------|----------------|
| `.shift_pos(by: T)` | Shifts the entire range in the positive direction | Yes |
| `.shift_neg(by: T)` | Shifts the entire range in the negative direction | Yes |
| `.add(how_much: T)` | Extends the end of the range | Yes |
| `.subt(how_much: T)` | Shrinks the end of the range | Yes |
| `.contains(n: T)` | Returns `true` if the range contains `n` | No |
| `.get(position: uint)` | Returns the value at the given index (0-based) — panics if out of bounds | No |
| `.try_get(position: uint)` | Returns `Option<T>` — `()` if out of bounds | No |

**Example:**

``` VEX
let mut r: range<i32> = 0..10;

r.shift_pos(by: 5);     // r is now 5..15
r.add(how_much: 5);     // r is now 5..20
r.contains(n: 12);      // true
r.get(position: 0);     // 5
r.try_get(position: 99); // ()
```

Checking containment:

``` VEX
if 0..100.contains(x: myValue) {
    // myValue is in range
}
```