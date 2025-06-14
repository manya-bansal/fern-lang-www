## Making a Function Call in Gern

The easiest program to write in Gern is to just make a simple function call:


```C++
#include "helpers.h"

int main() {

    // Our first, simple program.
    auto a = mk_array("a");
    auto b = mk_array("b");

    gern::annot::add_1 add_1;

    Composable program = {
        add_1(a, b),
    };

    compile_program(program);
}
```
