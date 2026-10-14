# CUDA Billiards Simulation

A ten-ball billiard simulation implemented in serial C++ and CUDA, with OpenGL visualization and CUDA–OpenGL buffer interoperability.

This academic project explores collision response, friction, GPU kernel decomposition, and sharing geometry buffers between computation and rendering. It demonstrates GPU programming and simulation engineering.

## Features

- One cue ball and nine object balls on a 2.40 m × 1.20 m table.
- Normal and tangential collision impulses, spin, cushion response, sliding friction, and rolling friction.
- An IFR-inspired collision model and post-collision motion described in the accompanying report using Kim's model.
- Serial CPU collision resolution and motion updates.
- CUDA pairwise collision kernels, atomic accumulation of collision deltas, and parallel ball updates.
- GPU-generated geometry written into mapped OpenGL VBOs.
- Console output for physics steps per second and accumulated rendering time.

## Implementation

The serial version checks all 45 unique ball pairs sequentially and updates each ball on the CPU. The CUDA version computes pair contributions on the GPU, accumulates changes with `atomicAdd`, applies them to ball states, and updates motion. CUDA maps shared OpenGL buffers, generates geometry, then unmaps them before OpenGL draws the scene. The CPU still initializes the application, launches kernels, synchronizes, and manages the display loop.

Both versions use a 1/240 s physics time step. The supplied source snapshot defaults to a 50 s benchmark, three physics steps per loop batch, and one run. The historical report contains different benchmark durations.

## Reported results

These values are transcribed from the supplied report; they have not been reproduced from this repository snapshot.

| Report duration | Serial steps/s | CUDA steps/s | Throughput ratio |
|---|---:|---:|---:|
| 100 s | 222.089 | 543.904 | 2.45× |
| 300 s | 191.333 | 1517.945 | 7.93× |
| 720 s | 190.710 | 1475.836 | 7.74× |

**Interpretation:** these are application-level observations, not a verified isolated CPU-versus-GPU physics benchmark. The supplied serial loop uses a wall-clock accumulator, while the CUDA loop advances as quickly as it can. Rendering paths, numeric precision, and collision update ordering also differ. An equivalent unpaced workload is needed before attributing the ratio solely to GPU acceleration. See [benchmark notes](benchmarks/README.md).

## Build and run

### Dependencies

- C++17 compiler and CMake 3.24 or newer.
- OpenGL with GLU and compatibility-profile support; FreeGLUT.
- For the CUDA executable: NVIDIA CUDA Toolkit, a supported host compiler, GLEW, and an NVIDIA GPU supporting CUDA–OpenGL interoperability.

On Windows, use Visual Studio with the Desktop development with C++ workload and an installed CUDA Toolkit compatible with that compiler. Supply x64 FreeGLUT and GLEW include/library paths through your dependency setup or CMake discovery variables. Third-party libraries are not bundled.

### CPU build

```sh
cmake -S . -B build -DBUILD_CUDA=OFF
cmake --build build --config Release
```

### CPU and CUDA build

```sh
cmake -S . -B build -DBUILD_CUDA=ON
cmake --build build --config Release
```

If dependency discovery fails, specify `GLUT_INCLUDE_DIR`, `GLUT_glut_LIBRARY`, `GLEW_INCLUDE_DIR`, and `GLEW_LIBRARY` as appropriate for your installation. Use libraries matching your compiler and architecture. Shared-library builds need their DLLs on PATH or beside the executable.

For Visual Studio builds:

```powershell
.\build\Release\billiards_serial.exe
.\build\Release\billiards_cuda.exe
```

For single-configuration builds, executables normally reside directly in `build/`.

The simulation starts automatically; `R` resets it and `Esc` exits. Results are printed to the console when the benchmark finishes. The CUDA source uses Windows `system("pause")`; Linux runtime portability is not verified.

On systems with integrated and discrete GPUs, the OpenGL context must run on the same NVIDIA GPU used by CUDA. This snapshot selects CUDA device 0; automatic matching is a future improvement.

## Files

- `src/serialCode.cpp`: serial CPU implementation.
- `src/CudaCode.cu`: CUDA implementation with corrected formatting errors.
- `CMakeLists.txt`: build configuration added during repository preparation.
- `benchmarks/reported_results.csv`: historical report values.
- `benchmarks/README.md`: methodology limitations and reproducibility guidance.
- `docs/report.pdf`: supplied academic report, unchanged.
- `docs/presentation.pptx`: supplied presentation, unchanged.
- `docs/PREPARATION_NOTES.md`: exact preparation changes and validation status.

## Validation status

Source and report inspection, source-copy comparison, result transcription checks, and archive integrity checks were completed. Compilation and GPU execution were not performed in the preparation environment because CUDA, CMake, and OpenGL development dependencies were unavailable. The new build configuration needs validation on a configured workstation.

## Credits and reuse

Project prepared for Gözde Özcan's portfolio. The accompanying report lists Gözde Özcan and Esin Onbaşıoğlu as authors; its authorship is preserved. Model references are provided in that report. No publication venue, DOI, or peer-reviewed publication status is claimed.

No open-source license has been selected. Public visibility does not grant unrestricted reuse; choose an appropriate license only after confirming rights to the code and accompanying documents.
