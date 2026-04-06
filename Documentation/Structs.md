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

### Generic Structs

Structs can be generic over types:

``` VEX
struct Pair<T> {
    first: T,
    second: T,
}

let p: Pair<i32> = Pair(first: 1, second: 2);
```

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