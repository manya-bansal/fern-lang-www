# Installation 

> Fern is now supported by the Gern repository. Gern adds support for GPUs and
> contains Fern-style CPU support. Gern is under 🚧 active development 🚧.

Fern is an open-source project available at
[https://github.com/manya-bansal/gern](https://github.com/manya-bansal/gern).

It is written in C++20 and requires CMake version 3.30 or higher to build.

## Getting the Repository

First, clone the repository from github.

```bash
$ git clone git@github.com:manya-bansal/gern.git
```

## Installing Dependencies

### VCPKG

Fern uses [`vcpkg`](https://github.com/microsoft/vcpkg) to manage its
dependencies. Follow the official guide to download and install `vcpkg`: [Get Started with
vcpkg](https://learn.microsoft.com/en-us/vcpkg/get_started/get-started?pivots=shell-powershell#1---set-up-vcpkg)

Once installed, set the `VCPKG_ROOT` environment variable to point to your
`vcpkg` directory.

### CMake

Fern uses `cmake` as its build system. Before building Fern, ensure that `cmake`
is installed on your system. Fern requires cmake 3.30 and above.

You can download the latest version of CMake from the official website: [CMake
Downloads](https://cmake.org/download/)

## Building Fern

Now that `VCPKG_ROOT` is set and you have downloaded the repository, build the
repository from the `gern` directory:

``` bash
$ cmake -DGern_CUDA_ARCH=<89,90..,etc> --preset dev 
$ cmake --build build/dev
```

If `-DGern_CUDA_ARCH` is not set, none of the GPU kernels will be run during
tests.

## Running Tests

To run tests, execute: 

```bash
$ ctest --test-dir build/dev
``` 



