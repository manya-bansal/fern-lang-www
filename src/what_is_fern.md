

# What is Fern?

Fern optimizes pipelines built from sequences of function calls. In particular,
Fern [*fuses*](fusion.md) function calls, and produces code that evaluates
operations back-to-back on small chunks of data as opposed to entire outputs.
Fern accomplishes this by requiring functions to declare how they produce and
consume data. 

Fern's core idea is that that fused code naturally separates concerns: inner
loops handle computation and hardware-specific logic, while outer loops manage
data movement. Fern uses black-box subroutines to perform computations and uses
simple dataflow analysis to generate outer loops.

Our [paper](https://dl.acm.org/doi/10.1145/3729292) on *Lightweight and
Locality-Aware Composition of Black-Box Subroutines* discusses Fern's design
philosophy in detail.

### Getting Started 

To get started with Fern, take a look at the [installation](install.md)
instructions.

The [tutorial](tutorial/overview.md) walks through several example Fern programs
to help you understand how it works in practice.

### Talk

To get more of an idea of Fern, you can our CppCon talk. Fern has evolved quite
a bit since then, but the design philosophy at the heart of Fern has remained
the same.

<div style="text-align: center;">
    <iframe width="560" height="315" src="https://www.youtube.com/embed/pEcOZDRXhNM?si=8q2EeHAD2LNppECr" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
</div>



