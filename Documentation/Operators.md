## Operators

### Logical Operators: `&&` and `||`

VEX's `&&` and `||` operators have a convenience feature: if a comparison operator (`==`, `!=`, `<`, `>`, `<=`, `>=`) appears on one side of a `&&` or `||` but not the other, the compiler **distributes** the comparison to the incomplete side automatically.

``` VEX
(x % 3) && (x % 5) == 0
// becomes:
(x % 3 == 0) && (x % 5 == 0)
```

``` VEX
(x % 3) || (x % 5) != 0
// becomes:
(x % 3 != 0) || (x % 5 != 0)
```

> If both sides already have a comparison operator, no distribution occurs.

### Mathematical Operator Shorthand

Writing a mathematical expression as a standalone statement implicitly assigns the result back to the **first variable** in the expression:

``` VEX
x * x;
// becomes:
x = x * x;
```

With two different variables, the first one is changed:

``` VEX
x * y + z;
// becomes:
x = (x * y + z);
```

> If a literal comes first (e.g. `5 * x`), this is a **compile-time error**. You must write `x = 5 * x` explicitly.