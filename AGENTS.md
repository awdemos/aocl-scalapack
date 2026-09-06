# Agent Notes: AOCL-ScaLAPACK

This repository is the awdemos fork of AMD's optimized ScaLAPACK distribution. It is a C/Fortran HPC linear-algebra library for distributed-memory (MPI) machines, forked from upstream Netlib ScaLAPACK and tuned for AMD Zen cores.

## Project Layout

- `SRC/` — Core ScaLAPACK driver routines (linear systems, least squares, eigenvalue, SVD).
- `PBLAS/` — Parallel BLAS kernels.
- `BLACS/` — Basic Linear Algebra Communication Subprograms (MPI wrappers).
- `TOOLS/` — Auxiliary tools and utilities.
- `TESTING/` — Test harnesses for the library routines.
- `EXAMPLE/` — Usage examples.
- `CMAKE/` — CMake modules and build configuration.
- `FRAMEWORK/` — Framework headers used by internal builds.
- `SLmake.inc.example` — Legacy Makefile include template.
- `BUILD.md` — Detailed build/install instructions.

## Build System

Two build systems are supported. Prefer CMake; the Makefile path is the legacy Netlib workflow.

### CMake (recommended)

Prerequisites: AOCL-BLIS, AOCL-libFLAME, AOCL-Utils, and an MPI library (validated with OpenMPI).

```bash
mkdir build && cd build
export PATH=/path/to/openmpi/bin:$PATH
export LD_LIBRARY_PATH=/path/to/openmpi/lib:$LD_LIBRARY_PATH
cmake ..   -DCMAKE_C_COMPILER=gcc   -DCMAKE_Fortran_COMPILER=gfortran   -DLAPACK_LIBRARIES="/path/to/libflame.a;/path/to/libaoclutils.a"   -DBLAS_LIBRARIES="/path/to/libblis.a"
make -j$(nproc)
```

Options:
- `-DENABLE_ILP64=ON` — 64-bit integer interface (default is LP64).
- `-DENABLE_COVERAGE=ON` — instrument for code coverage.
- `-DENABLE_ASAN_TESTS=ON` — build AddressSanitizer test variants.

### Legacy Makefile

```bash
cp SLmake.inc.example SLmake.inc
# edit SLmake.inc for your compilers, MPI, and library paths
make lib      # build the library only
make exe      # build testing executables
make example  # build examples
make clean    # remove object files
```

## Test / Verify

After building, run the test suite with ctest from the CMake build directory:

```bash
ctest --output-on-failure -j$(nproc)
```

Or use the legacy top-level test script:

```bash
./scalapack_test.sh
```

## Key Conventions

- Fortran sources use fixed-form `.f` and free-form `.f90` files.
- Public routines follow ScaLAPACK naming: `p?gesv`, `p?gesvd`, etc., where `?` is `s`/`d`/`c`/`z`.
- The `CDEFS` option controls Fortran-to-C name mangling (`NoChange`, `UpCase`, `Add_`).
- Keep `README.md`, `BUILD.md`, and `NOTICES` in sync when changing build prerequisites or external dependencies.

## Common Issues

- **MPI not found**: ensure the MPI compiler wrappers (`mpicc`/`mpif90`) are in `PATH` and the MPI runtime is on `LD_LIBRARY_PATH`.
- **LAPACK link errors**: AOCL-libFLAME depends on AOCL-Utils and `libstdc++`; include all three in `LAPACK_LIBRARIES`.
- **AOCC builds**: compile OpenMPI with AOCC first, then point CMake at the AOCC toolchain.

## License

See `LICENSE` and `NOTICES` for AMD and third-party attribution.
