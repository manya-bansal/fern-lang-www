# Tiling Through Reductions

```C++
#include "helpers.h"
#include "library/array/annot/cpu-array.h"
#include "library/array/impl/cpu-array.h"


Similar to the tiling the output, users can
introduce tiling over a "Reducible" parameter. For instance, the reduction function shown below takes a parameter `k` that specifies how many elements of array `a` contribute to each output element in array `b`:

```C++
inline void reduction(ArrayCPU a, ArrayCPU b, int64_t k) {
    for (int64_t i = 0; i < b.size; i++) {
        for (int64_t j = 0; j < k; j++) {
            b.data[i] += a.data[j];
        }
    }
}
```

The annotation for the function indicates that the function can also be
split across `k`. 

```C++
return annotate(Tileable(x = Expr(0), output["size"], step,
                        Computes(
                            Produces::Subset(output, {x, step}),
                            Consumes::Subsets(
                                // Indicate a reducible loop
                                Reducible(r = Expr(0), k, reduce,
                                    SubsetObjMany{
                                    SubsetObj(input, {r, reduce})})))));
```

The key point is that the reducible parameter must be part of the function interface so that users can tile a reducible loop if needed. A reducible loop is similar to a tilable loop, but with one important distinction: its output is accumulated into a value that is staged outside the loop. This external accumulator enables reduction across tiles or iterations.

```C++
using namespace gern;

int main() {
    // ***** PROGRAM DEFINITION *****
    auto input = mk_array("input");
    auto output = mk_array("output");

    gern::annot::reduction reduce;
    Variable len("len");
    Variable tile_size("tile_size");

    Composable program({
        Reduce(len, tile_size)(
            reduce(input, output, len)),
    });

    // ***** PROGRAM EVALUATION *****
    library::impl::ArrayCPU a(10);
    a.ascending();
    library::impl::ArrayCPU b(10);
    int64_t len_val = 10;
    int64_t tile_size_val = 2;

    auto runner = compile_program(program);
    runner.evaluate({
        {"input", &a},
        {"output", &b},
        {"len", &len_val},
        {"tile_size", &tile_size_val},
    }); 
}
```

The program produces the following code:

```C++
void function_9(library::impl::ArrayCPU &input, library::impl::ArrayCPU &output, int64_t len, int64_t tile_size) {
    for (int64_t _gern_r_1_5 = 0; (_gern_r_1_5 < len); _gern_r_1_5 = (_gern_r_1_5 + tile_size)) {
        auto _query_input_10 = input.query(_gern_r_1_5, tile_size);

        library::impl::reduction(_query_input_10, output, tile_size);
    }
}
```