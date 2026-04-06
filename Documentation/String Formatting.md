## String Formatting

Variables and expressions can be embedded directly into strings using `{}`:

``` VEX
let name: String = "World";
println("Hello {name}!");   // Hello World!
```

Any valid VEX expression can be placed inside `{}`, including function calls, method chains, and arithmetic:

``` VEX
println("Sum: {add(a: 3, b: 4)}");       // Sum: 7
println("Square: {x.square()}");          // Square: 25
println("Result: {a + b}");              // Result of expression
```

The expression inside `{}` follows normal ownership rules.