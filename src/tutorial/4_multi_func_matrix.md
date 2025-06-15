# Demystifying Data Structures 

The last piece to unpack before we begin fusing programs is the set of data
structures we've been using in our program definitions. Just like the `add_1`
function, these data structures are also part of the overall program definition
and contain essential information that Fern uses during code generation and
execution.

Let's look at the array object that `mk_array` produces:

```C++
    auto input = mk_array("input");
```

```C++
class ArrayCPU : public gern::AbstractDataType {
public:
    // Array must have a name!
    ArrayCPU(const std::string &name)
        : name(name) {
    }
    // Get the name to use in codegen. 
    std::string getName() const override {
        return name;
    }

    // Get the type to use in codegen. 
    std::string getType() const override {
        return "library::impl::ArrayCPU";
    }

    // The fields of the array are what we will eventually 
    // use to tile our program! In this represenatation
    // x denotes about the start of a subarray, while len
    // denotes the length of the subarray. Gern attaches
    // no semantic meaning to these fields, it just promises
    // to wire then up correctly! The ordering here does play a role!
    std::vector<Variable> getFields() const override {
        return {x, len};
    }
    // How can gern allocate an array?
    FunctionSignature getAllocateFunction() const override {
        return FunctionSignature{
            .name = "library::impl::ArrayCPU::allocate",
            .args = {x, len},
        };
    }
    // How can gern free an array?
    FunctionSignature getFreeFunction() const override {
        return FunctionSignature{
            .name = "destroy",
            .args = {},
        };
    }
    // If gern has a subarray, how can it insert into a parent array?
    FunctionSignature getInsertFunction() const override {
        return FunctionSignature{
            .name = "insert",
            .args = {x, len},
        };
    }

    // If gern wants to extract a subarray, how can it query it?
    FunctionSignature getQueryFunction() const override {
        return FunctionSignature{
            .name = "query",
            .args = {x, len},
        };
    }

protected:
    std::string name;
    Variable x{"x"};
    Variable len{"len"};
};
```

Similar to our array example, we could have instead used a matrix 
data structure. Let's take a look at the fieds of the matrix:

```C++
  std::vector<Variable> getFields() const override {
        return {x, y, l_x, l_y};
  }
```

Here, `x` and `y` point at the start of a submatrix, while 
`l_x` and `l_y` denote its height and width. Similar to the
array, we can write a program that uses the matrix!

```C++
#include "helpers.h"
#include "library/matrix/annot/cpu-matrix.h"
#include "library/matrix/impl/cpu-matrix.h"

    int
    main()
{
    // ***** PROGRAM DEFINITION *****
    auto input = mk_matrix("input");
    auto output = mk_matrix("output");
    auto temp = mk_matrix("temp");

    annot::MatrixAddCPU add_1;

    Composable program({
        add_1(input, temp),
        add_1(temp, output),
    });

    // ***** PROGRAM EVALUATION *****
    library::impl::MatrixCPU a(10, 10, 10);
    a.ascending();
    library::impl::MatrixCPU b(10, 10, 10);

    auto runner = compile_program(program);
    runner.evaluate({
        {"input", &a},
        {"output", &b},
    });

    // ***** SANITY CHECK *****
    for (int i = 0; i < a.col; i++)
    {
        for (int j = 0; j < a.row; j++)
        {
            assert(a(i, j) + 2 == b(i, j));
        }
    }
}
```