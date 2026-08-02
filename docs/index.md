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
- **Positive-definite linear algebra** — matrix-vector products, solves and
  inverses over whole fields of small symmetric matrices.
- **Regularisers** — absolute / membrane / bending energies on multi-channel
  fields and vector flows, including the fused accumulate forms
  (`field_addmatvec`, `flow_subdiag_`, …) and a Gauss–Seidel relaxation solver.

It works with **NumPy** and **PyTorch** arrays, and the PyTorch interface is
autograd-enabled.

!!! warning "CPU today"

    Every operation above ships and runs **on the CPU**. The CUDA backend
    compiles and links, but no GPU wheel is published yet, so
    `fastfields.cupy` and CUDA `torch` tensors are not usable — see
    [Status](#status) and [Roadmap](#roadmap).

## Status

What you actually get from a `pip install`, per operation family and per
backend. ✓ = works today; *planned* = on the [roadmap](#roadmap), not
installable yet.

| operation family | `.numpy`<br>CPU | `.torch`<br>CPU | `.torch`<br>CUDA | `.cupy`<br>CUDA |
| --- | :--: | :--: | :--: | :--: |
| **Distance transforms** — `dt_euclidean`, `dt_l1` | ✓ | ✓ | planned | planned |
| **Point → mesh distance** — `dt_mesh` | ✓ | ✓ | planned | planned |
| **Point → spline distance** — `dt_spline_table` / `_brent` / `_gaussnewton` | ✓ | planned | planned | planned |
| **Posdef linear algebra** — `sym_matvec`, `sym_solve`, `sym_invert`, … | ✓ | ✓ | planned | planned |
| **Resampling** — `resample`, `restriction`, `spline_coeff` | ✓ | ✓ | planned | planned |
| **Pushpull** — `pull`, `push`, `count`, `grad` | ✓ | ✓ | planned | planned |
| **Field regularisers** — `field_matvec`, `field_diag`, `field_kernel`, `field_relax`, `field_precond`, + the `add`/`sub` accumulate forms | ✓ | ✓ | planned | planned |
| **Flow regularisers** — `flow_matvec`, `flow_diag`, `flow_kernel`, `flow_relax`, `flow_precond`, + the `add`/`sub` accumulate forms | ✓ | ✓ | planned | planned |

`fastfields.any` dispatches on the type of the array you hand it, so it reaches
every ✓ above. Two deliberate exceptions: `dt_spline_*` dispatches only for
NumPy and CuPy arrays (the backends that implement it), and the shape-based
builders `field_diag` / `field_kernel` / `flow_diag` / `flow_kernel` have no
array to dispatch on, so call those on a concrete backend.

**Why is the whole CuPy column *planned*?** `fastfields.cupy` is written,
installable and importable, but every one of its ops needs a **CUDA build of the
native `libfastfields` library**, and the published wheels are CPU-only. The
same applies to CUDA `torch` tensors: they reach the device-dispatch layer and
raise. See the [roadmap](#roadmap) for where the CUDA work stands.

**Autograd (PyTorch).** `sym_matvec`, `sym_solve`, `resample`, `restriction`,
`spline_coeff`, `pull`, `push`, `field_matvec` and `flow_matvec` are backed by
`torch.autograd.Function` and differentiate. `sym_invert`, the distance
transforms, `count` and `grad` are not differentiable — they raise if an input
requires grad, or detach.

## Install

Only the **CPU** lane of the [wheel index](https://fastfields.github.io/whl/) is
published today (everything else resolves from PyPI):

```sh
pip install fastfields[numpy] \
    --extra-index-url https://fastfields.github.io/whl/cpu/
```

Swap in `fastfields[torch]` for the PyTorch interface, or install both — they
share the `fastfields` namespace and merge into one import.

## Use it

`fastfields.any` picks the right backend from whatever array you pass:

```python
import numpy as np
from fastfields import any as ff

mask = np.zeros((256, 256), "float32")
mask[128, 128] = 1.0

dist = ff.dt_euclidean(mask)      # Euclidean distance transform
```

Prefer a specific framework? Use its package directly for a typed API — with
autograd on PyTorch.

## Packages

| install | import | status | docs |
|---|---|---|---|
| `fastfields` | `fastfields.any` | ✓ CPU | [](https://fastfields.github.io/fastfields/) |
| `fastfields-numpy` | `fastfields.numpy` | ✓ CPU | [](https://fastfields.github.io/fastfields-numpy/) |
| `fastfields-torch` | `fastfields.torch` | ✓ CPU | [](https://fastfields.github.io/fastfields-torch/) |
| `fastfields-cupy` | `fastfields.cupy` | needs a CUDA build ([roadmap](#roadmap)) | [](https://fastfields.github.io/fastfields-cupy/) |

Install only the backends you need — they share the `fastfields` namespace and
merge into one import.

## Roadmap

Tracked and largely written, but **not shipped** — do not plan around these yet:

- **GPU / CUDA.** The `cu118`, `cu126` and `cu128` folders exist on the wheel
  index but hold no wheels, so passing one as an `--extra-index-url` resolves to
  nothing. The CUDA library itself compiles and links under `nvcc`, with host
  launchers for every module — but there is **no GPU in CI**, so none of it has
  ever been executed, and it should be treated as unvalidated. Until a CUDA
  wheel ships, `fastfields.cupy` and CUDA `torch` tensors cannot work.
- **Point → spline distance on PyTorch.** `dt_spline_table`, `dt_spline_brent`
  and `dt_spline_gaussnewton` are NumPy- and CuPy-only for now.
- **More operators.** Reweighted-least-squares (RLS) regularisers, the pushpull
  Hessian, and tetrahedron rasterisation exist in the C++ tree but are not yet
  exposed to Python.

The internal, per-layer porting status (kernels → impl → lib, and which modules
are CPU-tested vs. compile-only) is tracked in `MIGRATION.md` in the
[`fastfields-lib`](https://github.com/fastfields/fastfields-lib) hub repo.

Source lives at [github.com/fastfields](https://github.com/fastfields).
