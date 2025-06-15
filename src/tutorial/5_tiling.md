# Tiling a multi-function program 


Finally, we can start rewriting tiling programs
and producing fused variants.

Let's go back to the annotation function that we saw 
[previously](./1_trivial.md).

```C++
   Annotation getAnnotation() override { 
        Variable x("x");
        return annotate(
            // Tilable tells us what dimensions of the output
            // can be tiled.
            Tileable(x = Expr(0), output["size"], step,
                     // Produces indicates what subset (pointed at 
                     // by the fields we saw in the last chapter) 
                     // of the output we are describing.
                     Produces::Subset(output, {x, step}),
                     // What subsets of the input do we need to 
                     // consume to produce the required output subset?
                     Consumes::Subset(input, {x, step})));
    }
```

Now to generate a fused version, we simply tile the output's dimension! As expected, only fields marked as tileable can be tiled.

```C++
#include "helpers.h"
#include "library/array/impl/cpu-array.h"

using namespace gern;

int main() {

    // ***** PROGRAM DEFINITION *****
    // Our first, simple program.
    auto input = mk_array("input");
    auto temp = mk_array("temp");
    auto output = mk_array("output");
    Variable t("t");

    gern::annot::add_1 add_1;

    Composable program({
        // Tile the output!
        // t indicates what the tile
        // size should be
        Tile(output["size"], t)(
            add_1(input, temp),
            add_1(temp, output)),
    });

    // ***** PROGRAM EVALUATION *****
    library::impl::ArrayCPU a(10);
    a.ascending();
    library::impl::ArrayCPU b(10);
    int64_t t_val = 2;

    auto runner = compile_program(program);
    runner.evaluate(
        {
            {"output", &b},
            {"input", &a},
            {"t", &t_val}, // <-- t must also be provided now!
        });

    // SANITY CHECK
    for (int i = 0; i < 10; i++) {
        assert(a.data[i] + 2 == b.data[i]);
    }
}
```

Finally, we can look at the generated tiled program:

```C++
void function_7(library::impl::ArrayCPU &input, library::impl::ArrayCPU &output, int64_t t) {
    for (int64_t _gern_x_2_6 = 0; (_gern_x_2_6 < output.size); _gern_x_2_6 = (_gern_x_2_6 + t)) {
        auto _query_output_8 = output.query(_gern_x_2_6, t);

        library::impl::ArrayCPU temp = library::impl::ArrayCPU::allocate(_gern_x_2_6, t);

        auto _query_input_9 = input.query(_gern_x_2_6, t);

        library::impl::add_1(_query_input_9, temp);

        library::impl::add_1(temp, _query_output_8);

        temp.destroy();
    }
}
```