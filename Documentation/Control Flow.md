## Control Flow

### If Statement

``` VEX
if condition {

}
else if condition {

}
else {

}
```

### If-Except Statement

The `if-except` statement executes a block if a condition is true, **except** when a second condition is also true. The `if` part is evaluated first; if it passes, the `except` part is then checked. If the `except` condition is true, the block is **not** executed. The optional `else except` block executes only when the `except` condition is true. You can still use a normal `else`.

``` VEX
if condition except exception_condition {

}
else except{   // executes only when except is true

}
else {         // executes when all of the above are false (normal else)

}
```

### For Loop

``` VEX
for i in 0..99 {

}
```

### While Loop

``` VEX
while condition {

}
```

### Loop (Infinite)

`loop` is equivalent to `while true`:

``` VEX
loop {

}
```