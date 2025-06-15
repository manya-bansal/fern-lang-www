# More Tilings


Let's look at more tilings that Gern allows programmers to write:

```C++
    // Outputs can be double tiled!
    Composable program({
        // More tiling!
        Tile(output["size"], t)(
            Tile(output["size"], t2)(
                add_1(input, temp),
                add_1(temp, output))),
    });
```

The program produces the output, where the computation is tiled twice:


```C++
void function_9(library::impl::ArrayCPU &input, library::impl::ArrayCPU &output, int64_t t, int64_t t2) {
    for (int64_t _gern_x_2_6_8 = 0; (_gern_x_2_6_8 < output.size); _gern_x_2_6_8 = (_gern_x_2_6_8 + t)) {
        for (int64_t _gern_x_2_6 = 0; (_gern_x_2_6 < t); _gern_x_2_6 = (_gern_x_2_6 + t2)) {
            auto _query_output_10 = output.query((_gern_x_2_6 + _gern_x_2_6_8), t2);

            library::impl::ArrayCPU temp = library::impl::ArrayCPU::allocate((_gern_x_2_6 + _gern_x_2_6_8), t2);

            auto _query_input_11 = input.query((_gern_x_2_6 + _gern_x_2_6_8), t2);

            library::impl::add_1(_query_input_11, temp);

            library::impl::add_1(temp, _query_output_10);

            temp.destroy();
        }
    }
}
```


Nested pipelines can also be written, for example:

```C++
    Composable program({
        // More tiling!
        Tile(output["size"], t)(
            Tile(temp["size"], t2)(
                add_1(input, temp)),
            add_1(temp, output)),
    });
```

prododuces the following program:

```C++

void function_9(library::impl::ArrayCPU &input, library::impl::ArrayCPU &output, int64_t t, int64_t t2) {
    for (int64_t _gern_x_2_8 = 0; (_gern_x_2_8 < output.size); _gern_x_2_8 = (_gern_x_2_8 + t)) {
        auto _query_output_10 = output.query(_gern_x_2_8, t);

        library::impl::ArrayCPU temp = library::impl::ArrayCPU::allocate(_gern_x_2_8, t);

        for (int64_t _gern_x_4_6 = 0; (_gern_x_4_6 < temp.size); _gern_x_4_6 = (_gern_x_4_6 + t2)) {
            auto _query_temp_11 = temp.query((_gern_x_4_6 + 0), t2);

            auto _query_input_12 = input.query((_gern_x_4_6 + _gern_x_2_8), t2);

            library::impl::add_1(_query_input_12, _query_temp_11);
        }

        library::impl::add_1(temp, _query_output_10);

        temp.destroy();
    }
}
```

At each point in the program, Fern has a notion of the data structure the user
intends to produce, and each statement yields a data structure. Tilings are
permitted only for the data structure that is the final output of the current
program scope. 