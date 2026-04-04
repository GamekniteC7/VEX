## Code Examples

### Hello World

``` VEX
// main.vxl
main(){
    hello_world();
}

fn hello_world() >> () {
    println("Hello, World!");
}
```

**Output:** `Hello, World!`

---

### Addition

``` VEX
// main.vxl
main(){
    let added: i32 = add(a: 5, b: 7);
}

fn add(mut a: i32, b: i32) >> i32 {
    a + b;
}
```

**Output:** `12`

---

### FizzBuzz

``` VEX
// main.vxl
main(){
    for i in 1..100 {
        println(FizzBuzz(x: &i));
    }
}

fn FizzBuzz(mut x: i32) >> String {
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
```

**Output:** FizzBuzz sequence from 1 to 100.

---

### Methods

``` VEX
// main.vxl
main(){
    let mut x: i32 = 5;
    println(x.square());
}

md square(mut x: i32) >> i32 {
    x * x;
}
```

**Output:** `25`