## Print Functions

VEX has three built-in print functions:

| Function | Description |
|----------|-------------|
| `print(value)` | Prints to stdout without a trailing newline |
| `println(value)` | Prints to stdout with a trailing newline |
| `printf(value)` | Prints to stdout with support for escape sequences |

### Escape Sequences in `printf()`

| Sequence | Description |
|----------|-------------|
| `\t` | Tab |
| `\n` | Newline |
| `\r` | Carriage return |
| `\\` | Literal backslash |
| `\"` | Literal double quote |

**Example:**

``` VEX
printf("Name:\t{name}\nAge:\t{age}");
```

Output:

``` Console
Name:   name
Age:    age
```