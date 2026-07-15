# Validation record

This document records the verification scope of this source snapshot. It is not a claim about untested configurations.

## Environment

- Host: JAIST Linux server
- Compiler: GCC 13
- Host application: ARCH, full Debug build
- I/O: HDF5 enabled
- Integrated revision: [`Shiro-Akane/ARCH@68f401c`](https://github.com/Shiro-Akane/ARCH/commit/68f401cb0edf2bc0bedb3734d9d4cd569d6c56fb)

## Species registry

| Selected network | Expected | Observed |
| --- | ---: | ---: |
| `iso7` | 7 | 7 |
| `aprox13` | 13 | 13 |
| `aprox19` | 19 | 19 |
| `aprox21` | 21 | 21 |

Names and ordering were checked against the corresponding network headers.

## Direct Fortran parity

Analytic rates and RHS outputs were compared with builds of the supplied Timmes `public_*.f90` sources. The tested configuration used full screening with weak rates disabled.

| Network | Maximum relative difference |
| --- | ---: |
| `iso7` | `3.342993e-16` |
| `aprox13` | `5.266553e-13` |
| `aprox19` | `1.033907e-13` |
| `aprox21` | `2.516835e-13` |
| **Overall** | **`5.266553e-13`** |

## Jacobian check

Forward-mode automatic-differentiation Jacobians were compared with numerical finite differences.

| Network | Maximum relative disagreement |
| --- | ---: |
| `iso7` | `1.558187e-6` |
| `aprox13` | `8.517464e-7` |
| `aprox19` | `4.329912e-6` |
| `aprox21` | `1.928217e-6` |

Finite-difference truncation and cancellation make this a different error metric from direct Fortran parity. The values above therefore do not replace or weaken the `1e-12` physics-parity check.

## CellularDet smoke test

A one-step CellularDet run used Helmholtz EOS, `BE_NR`, DenseLU, RK3, HLLC and PPM. Both tested larger networks completed and produced HDF5 output:

- `aprox19`: `Species Count: 19`
- `aprox21`: `Species Count: 21`

## Exclusions

- Weak-rate hooks `irpen`, `irnep`, and `irn56ec`
- `sneut5` neutrino losses
- Tabulated reaction-rate mode

These exclusions are documented in all three project home pages and must be revisited before results depending on those paths are treated as validated.
