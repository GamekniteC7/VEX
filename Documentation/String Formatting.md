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

Any valid VEX expression can be placed inside {}, including function calls, method chains, and arithmetic. You can even embed entire programs — by opening a new main(){} scope inside the brackets, you create a fully isolated execution environment whose output is inserted into the string:

``` VEX
println("FizzBuzz Output: {
    main() {
        for i in 1..100 {
            println(FizzBuzz(x: &i));
        }
    }

    fn FizzBuzz(tmut x: i32) >> String {
        if (x % 3) && (x % 5) == 0 {
            return "FizzBuzz!";
        }
        else if x % 3 == 0 {
            return "Fizz!";
        }
        else if x % 5 == 0 {
            return "Buzz!";
        }
        else {
            return x.toString();
        }
    }
}");
```

The main(){} inside the brackets acts as the entry point of the embedded program. The scope is completely isolated from the surrounding code — variables and functions defined inside do not leak out, and outer variables are not accessible inside.
