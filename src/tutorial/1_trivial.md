# Making a Function Call in Fern

The easiest program to write in Fern is one that makes a single function call:


```C++
#include "helpers.h"

int main() {

    // Our first, simple program.
    auto a = mk_array("a");
    auto b = mk_array("b");

    // Gern object that represents a function.
    gern::annot::add_1 add_1;

    Composable program = {
    // the () operator calls the function, and 
    // generates a "call site".
        add_1(a, b),
    };

    // The program can now be compiled.
    compile_program(program);
}
```

Let's look at the different pieces of the `add_1` object carefully: 

```C++
class add_1 : public gern::AbstractFunction {
public:
    add_1()
        : input(new const ArrayCPU("input")),
          output(new const ArrayCPU("output")) {
    }

    /* getAnnotation() returns the data production
     * and consumption pattern of the function.
     * See Tiling Programs in the tutorial for more
     * details if you'd like to skip ahead!
    */
    Annotation getAnnotation() override { 
        Variable x("x");

        return annotate(
            Tileable(x = Expr(0), output["size"], step,
                     Produces::Subset(output, {x, step}),
                     Consumes::Subset(input, {x, step})));
    }

     /* getFunction() returns the "C++" signature of the
      * Function that we ultimately use to generate code. 
     */
    virtual FunctionSignature getFunction() override {
        FunctionSignature f;
        f.name = "library::impl::add_1"; // <-- Name of the Function
        f.args = {Parameter(input), Parameter(output)}; // <-- Parameters of the function 
        // Also contains template arguments which has been left empty.
        return f;
    }

    /**  The file that the function declration lives in, this is included
     *   in the generated file.
     */
    std::vector<std::string> getHeader() override {
        return {
            "cpu-array.h",
        };
    }

protected:
    AbstractDataTypePtr input;
    AbstractDataTypePtr output;
    Variable end{"end"};
    Variable step{"step"};
};
```

Given our Fern program and the function definition, when we
compile the program, we simply get a function that makes a
single call (we have not tiled our program yet!):


```C++
#include "cassert"
#include "cpu-array.h" // Our include file!

// An automatically generated wrapper function name.
void function_3(library::impl::ArrayCPU& a, library::impl::ArrayCPU& b){
      library::impl::add_1(a, b); // the actual function call!
}
```

Fern writes the generated code into a file which can then be included in a
project, and run like normal.