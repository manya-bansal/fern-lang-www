# Overview


The tutorial walks you through different programs in Fern with increasing
complexity. The interface to Fern has changed from the paper, though it is
equally powerful.

As opposed to thinking about an explicit separation between algorithm and
schedule—where the schedule is a meta-program describing how lowering might
proceed—the new interface is a "sketch" of the loop nest the user wants the
generated program to have. This sketch is then deterministically lowered into
imperative code, with the compiler filling in any details that can be
automatically inferred.

The tutorial is organized as follows:

1. [Making our first function call](./1_trivial.md) 
2. [Running our first function call](./2_running.md)
3. [Writing a Multi-Function Pipeline](./3_multi_func.md)
4. [Understaning Data Structures](./4_multi_dunc_matrix.md)
5. [Tiling Programs](./5_tiling.md)
6. [Understanding Legal Tilings](./6_tiling.md)
7. [Tiling Through Reductions](./7_reductions.md)