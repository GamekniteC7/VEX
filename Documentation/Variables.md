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