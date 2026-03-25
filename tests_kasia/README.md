# OzaBLAS Table Benchmark

A standalone benchmark for OzaBLAS that generates matrices with a specific condition number and measures performance and accuracy of Ozaki Scheme I and II against native DGEMM.

## Features

- Generates ill-conditioned matrices (condition number 10^8) using SVD construction
- Benchmarks native FP64 DGEMM (hipBLAS/cuBLAS)
- Benchmarks Ozaki Scheme I (12 and 16 slices)
- Benchmarks Ozaki Scheme II (12 and 16 slices)
- Reports performance in TFLOPS and accuracy as relative Frobenius error
- Verbose progress output for debugging hangs

## Prerequisites

1. Build the main OzaBLAS library first:
   ```bash
   cd /path/to/ozablas
   make release-hip    # For AMD GPUs
   # or
   make release-cuda   # For NVIDIA GPUs
   ```

2. Required libraries:
   - **HIP**: hipblas, rocblas, hiprand, rocsolver
   - **CUDA**: cublas, curand, cusolver

## Building

```bash
cd tests_kasia
mkdir build && cd build
cmake .. -DOZABLAS_ENABLE_HIP=ON    # For AMD GPUs
# or
cmake .. -DOZABLAS_ENABLE_CUDA=ON   # For NVIDIA GPUs
make
```

## Usage

```
./table_benchmark [options] [size1] [size2] ...

Options:
  --scheme1, -1    Run Ozaki Scheme I only
  --scheme2, -2    Run Ozaki Scheme II only
  --both, -b       Run both schemes (default)
  --help, -h       Show help
```

### Examples

```bash
# Run both schemes with default sizes (1024, 2048, 4096, 8192)
./table_benchmark

# Run both schemes with custom sizes
./table_benchmark 4096 8192 16384

# Run only Scheme I (useful for debugging large matrices)
./table_benchmark -1 16384

# Run only Scheme II
./table_benchmark --scheme2 4096 8192
```

## Output

The benchmark outputs:
- **stderr**: Verbose progress (useful for debugging hangs)
- **stdout**: Summary table with results

### Sample Output

```
======================================================================================================================
                                              OZABLAS BENCHMARK SUMMARY
                                          Condition Number: 10^8
======================================================================================================================

+-------------+-------------+-----------------------------+-----------------------------+-----------------------------+-----------------------------+
| Matrix Size | Native GEMM |   Ozaki Scheme I (s=12)     |   Ozaki Scheme I (s=16)     |   Ozaki Scheme II (s=12)    |   Ozaki Scheme II (s=16)    |
+-------------+-------------+-----------------------------+-----------------------------+-----------------------------+-----------------------------+
|             | Performance | Performance | Accuracy      | Performance | Accuracy      | Performance | Accuracy      | Performance | Accuracy      |
+-------------+-------------+-------------+---------------+-------------+---------------+-------------+---------------+-------------+---------------+
|        4096 |    45.30 TF |     2.15 TF | 1.23e-14      |     1.45 TF | 4.56e-15      |    32.50 TF | 2.34e-12      |    28.30 TF | 1.89e-13      |
+-------------+-------------+-------------+---------------+-------------+---------------+-------------+---------------+-------------+---------------+

Legend:
  - Performance in TFLOPS (TF)
  - Accuracy: Relative Frobenius error vs native FP64 DGEMM
  - s=12, s=16: Number of slices (more slices = better accuracy, lower performance)
```

## Debugging Hangs

If the benchmark appears to hang, run with verbose output:

```bash
./table_benchmark -1 16384 2>&1 | tee debug.log
```

The stderr output shows exactly where execution is:
```
  Scheme I (s=12, 144 GEMMs per call)...
    Allocating workspace... done
    Warmup (2 iters, 288 total GEMMs)...[1: calling gemm...sync...done][2: calling gemm...sync...done] done
    Timing (50 iters, 7200 total GEMMs)..........10..........20...
```

## Configuration

Edit `table_benchmark.cpp` to change:

```cpp
static const int NUM_WARMUP = 2;       // Warmup iterations
static const int NUM_ITERATIONS = 50;  // Timed iterations
static const int LOG10_COND = 8;       // Condition number 10^8
```
