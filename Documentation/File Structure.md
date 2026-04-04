## File Structure

Every VEX project has one `main.vxl` file. This file contains the entry point of the program — the `main()` function. All other `.vxl` files must be attached to `main.vxl` directly or indirectly to exist in the program.

``` VEX
main(){
    // entry point
}
```

> `main()` is special. It is not declared with `fn` or any visibility modifier. Any function named `main` elsewhere — such as `pub fn main()` in another file — is treated as a regular function, not the entry point. Within `main.vxl` itself, the name `main` is reserved for the entry point only.

### Attaching Files

Files are attached using the `attach` keyword. Without attachment, a file is completely invisible to the rest of the program.

``` VEX
// main.vxl
attach math
attach utils

main() {
    ...
}
```

Attachments can be organized into an intermediate file to keep `main.vxl` clean:

``` VEX
// modules.vxl
attach math
attach utils
attach graphics
```

``` VEX
// main.vxl
attach modules

main() {
    ...
}
```

- `attach` only makes `pub` declarations from the attached file accessible.
- Attached declarations are always called with explicit file qualification: `file_name::function_name`.
- The order of `attach` statements does not matter.
- Circular attachments are a **compile-time error**.