## Methods (Uniform Function Call Syntax)

In VEX, there is no special method declaration keyword. Instead, **any function** can be called as a method using **dot notation**. When a function is called as a method, the value before the dot is automatically passed as the first parameter to the function.

When called as functions, they behave like normal VEX functions. When called as methods, they behave like VEX methods.

- Methods are called using **dot notation** with parentheses: the value before the dot is automatically passed as the first parameter.
- Methods **implicitly reassign** the result back to the caller variable. The caller must therefore be at least `mut`.
- The return type of a method does not need to match the original type — after a method call, the variable's type changes to the return type of the function. If a method changes the variable's type, the variable **must** be declared as `tmut` (type-mutable). Calling a type-changing method on a purely `mut` variable is a compile-time error.
- Named arguments are required for any additional parameters.
- `pub fn` makes a function available across files, which can then be called as a method.

**Example:**

``` VEX
fn square(mut x: i32) >> i32 {
    x * x;
}

let mut x: i32 = 5;
x.square()      // equivalent to: 
x = square(x: x)
                // x is now 25

// You can also call it as a normal function:
x = square(x: x);
```

**Type change after method call:**

``` VEX
let tmut x: i32 = 5;
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

A function's applicable type when called as a method is determined by its first parameter's type. A function declared with `(x: i32)` will only work as a method on `i32`.

### Generic Methods

Functions can also be generic. When called as a method, the type is inferred automatically from the variable the method is called on — no explicit type annotation needed at the call site:

``` VEX
fn first<T>(tmut x: T, list: vector<T>) >> Option<T> {
    // ...
}

let tmut x: i32;
x.first(list: numbers);  // T is inferred as i32 from x
```

### Method Naming

Function names must be **unique within a file** but can share names across files. When calling a method from a specific file, use:

``` VEX
x.file_name::function_name()
```

If a namespace alias is defined for the file, you can use the alias instead:

``` VEX
x.alias::function_name()
```

If a function name exists in multiple attached files and no file is specified, VEX looks for the function in the **current file**. If it isn't found there, a compile-time error is raised requiring explicit file qualification.

> Function overloading (same name, different types) is **not supported** within a file.
