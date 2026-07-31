---
icon: fontawesome/solid/house
---

# fastfields

**fastfields** is a fast, `pip`-installable toolkit for computing on dense
**scalar and vector fields** — the kind of array operations that show up all
over image processing, medical imaging, and geometry:

- **Distance transforms** — Euclidean and L1 distance maps; distance from points
  to a 1-D spline or to a triangle mesh.
- **Resampling** — spline interpolation up/down a grid, its adjoint, and spline
  coefficient prefiltering.
- **Pushpull sampling** — gather/scatter samples at arbitrary coordinates (the
  building block of image warping) with spatial gradients.
- **Positive-definite linear algebra** — fast matrix-vector products, solves and
  inverses over whole fields of small symmetric matrices.
- **Regularisers** — absolute / membrane / bending energies on multi-channel
  fields and vector flows.

It works with **NumPy, PyTorch and CuPy** arrays, runs on **CPU and GPU**, and
the PyTorch interface is **fully differentiable**.

## Install

Grab a build for your hardware from the [wheel index](https://fastfields.github.io/whl/)
(everything else comes from PyPI):

```sh
# GPU (CUDA 12.8)
pip install fastfields[torch] \
    --extra-index-url https://fastfields.github.io/whl/cu128/

# CPU only
pip install fastfields[numpy] \
    --extra-index-url https://fastfields.github.io/whl/cpu/
```

## Use it

`fastfields.any` picks the right backend from whatever array you pass — NumPy,
PyTorch or CuPy:

```python
import numpy as np
from fastfields import any as ff

mask = np.zeros((256, 256), "float32")
mask[128, 128] = 1.0

dist = ff.dt_euclidean(mask)      # Euclidean distance transform
```

Prefer a specific framework? Use its package directly for a typed API — with
autograd on PyTorch and CUDA streams on CuPy.

## Packages

| install | import | docs |
|---|---|---|
| `fastfields` | `fastfields.any` | [API](https://fastfields.github.io/fastfields/) |
| `fastfields-numpy` | `fastfields.numpy` | [API](https://fastfields.github.io/fastfields-numpy/) |
| `fastfields-torch` | `fastfields.torch` | [API](https://fastfields.github.io/fastfields-torch/) |
| `fastfields-cupy` | `fastfields.cupy` | [API](https://fastfields.github.io/fastfields-cupy/) |

Install only the backends you need — they share the `fastfields` namespace and
merge into one import.

Source lives at [github.com/fastfields](https://github.com/fastfields).
