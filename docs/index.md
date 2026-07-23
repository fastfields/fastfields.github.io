# fastfields

**fastfields** is a set of layered C++/CUDA libraries for dense **scalar and
vector fields** — distance transforms, spline resampling and pushpull sampling,
fields of small positive-definite matrices, and spatial regularisers — with
thin, stride-aware **numpy / torch / cupy** bindings. It is a rewrite of the
JIT-compiled `jitfields`, dropping the cppyy/torchlib dependency: data crosses
backends via **DLPack** and the Python layer binds with **nanobind**.

## Install

Pick the build that matches your hardware from the
[wheel index](https://fastfields.github.io/whl/) (dependencies still resolve
from PyPI, so pass it as an `--extra-index-url`):

```sh
# CUDA 12.4
pip install fastfields fastfields-torch \
    --extra-index-url https://fastfields.github.io/whl/cu124/

# CPU only
pip install fastfields fastfields-numpy \
    --extra-index-url https://fastfields.github.io/whl/cpu/
```

## Getting started

`fastfields.any` dispatches on the input array type, so the same call works for
numpy, torch or cupy arrays:

```python
import numpy as np
from fastfields import any as ff

x = np.zeros((128, 128), "float32")
x[64, 64] = 1.0
dist = ff.dt_euclidean(x)     # Euclidean distance transform
```

For a typed, backend-specific API (with torch autograd, cupy streams, …) use the
per-backend packages listed below.

## Packages

| package | import | docs |
|---|---|---|
| `fastfields` | `fastfields.any` | [API](https://fastfields.github.io/fastfields/) |
| `fastfields-numpy` | `fastfields.numpy` | [API](https://fastfields.github.io/fastfields-numpy/) |
| `fastfields-torch` | `fastfields.torch` | [API](https://fastfields.github.io/fastfields-torch/) |
| `fastfields-cupy` | `fastfields.cupy` | [API](https://fastfields.github.io/fastfields-cupy/) |
| `fastfields-dlpack` | `fastfields.dlpack` | [API](https://fastfields.github.io/fastfields-bind-py/) |
| `libfastfields` (C++) | — | [docs](https://fastfields.github.io/fastfields-lib/) |

All Python distributions share the `fastfields` **PEP 420 namespace**, so you
install only the backends you need and they merge into one import root.

## Architecture

The stack is one repository per layer, wired by git submodules:

```
fastfields-kernels        voxelwise math (header-only, backend-agnostic, templated)
  ├─ fastfields-cpu-impl   CPU loops  ─ fastfields-cpu-lib   (libfastfields-cpu, dtype dispatch)  ┐
  └─ fastfields-cuda-impl  CUDA loops ─ fastfields-cuda-lib  (libfastfields-cuda, nvcc)           ┴─ fastfields-lib (libfastfields, DLTensor, device dispatch)
                                                                                                    └─ fastfields-bind-py (fastfields.dlpack, nanobind)
                                                                                                         ├─ fastfields-numpy / -cupy / -torch
                                                                                                         └─ fastfields (fastfields.any)
```

- **kernels** compute on a single element; **impl** owns the loops; the **lib**
  layers dispatch dtype/device behind an unsafe-pointer / `DLTensor` ABI.
- The C++ core is **stride-aware**: arrays pass through with their native
  strides (including broadcast views), so the bindings avoid needless copies.

See [the source](https://github.com/fastfields) and each repo's `CLAUDE.md` /
`CONTRIBUTING.md` for details.
