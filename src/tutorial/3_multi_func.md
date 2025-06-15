## Calling multiple functions

Now that we know how to write and execute Fern pipelines with one function,
let's turn our attention to longer pipelines. Here's our previous pipeline,
but now with one additional function call: 

```C++
#include "helpers.h"
#include "library/array/impl/cpu-array.h"

int main() {

    // ***** PROGRAM DEFINITION *****
    // Our first, simple program.
    auto input = mk_array("input");
    auto temp = mk_array("temp");
    auto output = mk_array("output");

    gern::annot::add_1 add_1;

    Composable program({
        add_1(input, temp),
        add_1(temp, output), // <--  New Call
    });

    // ***** PROGRAM EVALUATION *****
    library::impl::ArrayCPU a(10);
    a.ascending();
    library::impl::ArrayCPU b(10);

    auto runner = compile_program(program);
    runner.evaluate(
        {
            {"output", &b},
            {"input", &a},
        });

    // SANITY CHECK
    for (int i = 0; i < 10; i++) {
        assert(a.data[i] + 2 == b.data[i]);
    }
}
```

As we begin to start writing longer pipelines, there are some rules about writing pipelines
must observe: 

1. All data-structures can be assigned to only once.
    > This means that we could not used `temp` again as the output of our second call.
2. All intermediates must be used an input at least once.
    > Fern would have thrown an error in the case that `temp` was computed, but never
    > used as input.

There are more rules coming up as we write tiled pipelines!

While our program defintion contains `temp`, you might have noticed that the program
evaluation does not! Indeed, if we look at the generated code, Fern is allocating it:

```C++
void function_5(library::impl::ArrayCPU &input, library::impl::ArrayCPU &output) {
    library::impl::ArrayCPU temp = library::impl::ArrayCPU::allocate(0, output.size);

    library::impl::add_1(input, temp);

    library::impl::add_1(temp, output);

    temp.destroy();
}
```

Fern owns all the temporary/intermediate tensors in a pipeline (and will
optimize for reuse of temporaries and hoisting them out). Therefore, only "true"
inputs and the "true" output is passed in.

At this point, you might have a lingering question: how did Fern figure out what
allocate call to make in the first place?