## Methods

Methods are declared with the `md` keyword:

``` VEX
md method_name(parameter_name: type) >> return_type {

}
```

- Methods follow the same return rules as functions (implicit return, `return` for early exits).
- Methods are called using **dot notation** with parentheses: the value before the dot is automatically passed as the first parameter.
- Methods **implicitly reassign** the result back to the caller variable. The caller must therefore be `mut`.
- The return type of a method does not need to match the original type — after a method call, the variable's type changes to the return type of the method.
- Named arguments are required for any additional parameters.
- `pub md` makes a method available across files.

**Example:**

``` VEX
md square(mut x: i32) >> i32 {
    x * x;
}

let mut x: i32 = 5;
x.square()      // equivalent to: x = square(x: x)
                // x is now 25
```

**Type change after method call:**

``` VEX
let mut x: i32 = 5;
x.toString()    // x is now a String
```

**With additional parameters:**

``` VEX
x.pow(b: &y)    // x is auto-referenced, y must be explicitly referenced
```

**Using a method as an expression:**

``` VEX
println(x.square())
// equivalent to:
x.square();
println(x);
```

**Passing a reference after a method call:**

``` VEX
add(a: &x.square(), b: 5)
// equivalent to:
x.square();
add(a: &x, b: 5);
```

### Method Chaining

Methods can be chained — each method is applied to the result of the previous one:

``` VEX
x.square().double().abs()
// equivalent to:
x.square();
x.double();
x.abs();
```

### Method Types

A method's applicable type is determined by its first parameter's type. A method declared with `(x: i32)` will only work on `i32`.

### Generic Methods

Methods can also be generic. The type is inferred automatically from the variable the method is called on — no explicit type annotation needed at the call site:

``` VEX
md first<T>(mut x: T, list: vector<T>) >> Option<T> {
    // ...
}

let mut x: i32;
x.first(list: numbers);  // T is inferred as i32 from x
```

### Method Naming

Method names must be **unique within a file** but can share names across files. When calling a method from a specific file, use:

``` VEX
x.file_name::method_name()
```

If a namespace alias is defined for the file, you can use the alias instead:

``` VEX
x.alias::method_name()
```

If a method name exists in multiple attached files and no file is specified, VEX looks for the method in the **current file**. If it isn't found there, a compile-time error is raised requiring explicit file qualification.

> Method overloading (same name, different types) is **not supported** within a file.