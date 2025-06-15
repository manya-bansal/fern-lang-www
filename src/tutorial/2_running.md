# Running our trivial program

[Part one](./1_trivial.md) demonstrated how to write a trivial, single
function program. To run it, we had to figure out where the generated
file lives, include it, compile and then execute it. While this works, it's
still annoying to keep including a file into the project just to
execute it. To avoid this, Fern also provides an interface for
jitting and loading the function pointer, so the code can be compiled
and executed more ergonomically.


In this case, we can consider the program as being split into two parts:
program definition and program execution.

```C++
#include "helpers.h"
#include "library/array/impl/cpu-array.h"

int main() {

    // ***** PROGRAM DEFINITION *****
    // Our first, simple program.
    auto a_def = mk_array("input");
    auto b_def = mk_array("output");

    gern::annot::add_1 add_1;

    Composable program = {
        add_1(a_def, b_def),
    };

    // ***** PROGRAM EVALUATION *****
    // Compile turns a runner object which can be
    // used to run the program.
    auto runner = compile_program(program);

    library::impl::ArrayCPU a(10);
    a.ascending();
    library::impl::ArrayCPU b(10);

    // Evaluate the program. The 
    // function takes in a map, and 
    // corresponding names of the inputs and
    // outputs. Fern will wire these up correctly! 
    runner.evaluate({
        {"output", &b},
        {"input", &a},
    });

    // Make sure we got the right answer.
    for (int i = 0; i < 10; i++) {
        std::cout << "a[" << i << "] = " << a.data[i]
                  << ", b[" << i << "] = " << b.data[i] << std::endl;
        assert(a.data[i] + 1 == b.data[i]);
    }
}
```


It is worthwhile breaking down what the `compile_program` helper
is hiding:

```C++
inline Runner compile_program(Composable program,
                              std::string name = "program.cpp") {
    Runner runner(program);
    Runner::Options options{
        // Name of the generated file
        .filename = name,
        // Path where the file should be put.
        .prefix = "/home/manya/gern/prez/gern_gen",
        // Paths to include
        .include = " -I /home/manya/gern/prez/library/array/impl"
                   " -I /home/manya/gern/prez/library/matrix/impl",
    };
    // Compile the program!
    runner.compile(options);
    return runner;
}
```

`Runner::Options` contains more choices for flags that can be set.