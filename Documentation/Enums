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