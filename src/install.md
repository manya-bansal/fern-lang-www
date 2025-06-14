# Install

> Fern is now supported by the Gern repository. Gern adds support for GPUs and
> contains Fern-style CPU support. Gern is under active development.

To download Fern, head to the GitHub repository at
https://github.com/manya-bansal/gern.

## Getting the Repository

```bash
git clone git@github.com:manya-bansal/gern.git
```

## Installing Dependencies

### VCPKG

Fern uses `vcpkg` to manage its dependencies. Download `vcpkg` and set
`VCPKG_ROOT`.

Instructions on downloading `vcpkg` are available
[here](https://learn.microsoft.com/en-us/vcpkg/get_started/get-started?pivots=shell-powershell#1---set-up-vcpkg).

### CMake

Fern uses `cmake` as its build tool. Make sure `cmake` has been installed.

Instructions for downloading CMake are available
[here](https://cmake.org/download/).

## Building Fern

Now that `VCPKG_ROOT` is set and you have downloaded the repository, build the
repository from the `gern` directory:

`
$ cmake -DGern_CUDA_ARCH=<89,90..,etc> --preset dev
$ cmake --build build/dev
`

If `-DGern_CUDA_ARCH` is not set, none of the GPU kernels will be run during tests.

## Running Tests

To run tests, run:

` $ ctest --test-dir build/dev `



