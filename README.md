# Timmes C++ Nuclear Networks

[English](README.md) | [简体中文](README.zh-CN.md) | [日本語](README.ja.md)

<p align="center">
  <img src="assets/timmes-network-pixel-banner.png" alt="Pixel-art stellar nuclear reaction network" width="900">
</p>

Validated C++20 ports of the classic Timmes hardwired nuclear reaction networks for stellar hydrodynamics. This repository contains the repaired network module used by ARCH, with fixed species registration and no runtime dependency on `pynucastro`.

The aprox-series nuclear-physics implementation is self-contained and does not require an external network-generation or runtime library. It is suitable for OpenMP and process-local MPI integration; the CUDA device path is currently verified for `aprox13`, while all four networks are verified on CPU/OpenMP.

> **Status:** private research snapshot. No open-source license is granted by this repository at this time; see [NOTICE.md](NOTICE.md).

## Included networks

| Network | Species | Registered nuclei/particles |
| --- | ---: | --- |
| `iso7` | 7 | He-4, C-12, O-16, Ne-20, Mg-24, Si-28, Ni-56 |
| `aprox13` | 13 | Alpha chain from He-4 through Ni-56 |
| `aprox19` | 19 | H-1, He-3, the alpha chain, N-14, Fe-54, neutron and proton |
| `aprox21` | 21 | `aprox19` plus Cr-56 and Fe-56 |

The implementation includes analytic reaction-rate expressions, Timmes screening, nuclear masses and binding energies, RHS assembly, frozen-screening composition Jacobians, and the exact analytic Timmes temperature row and column. Each `.inc` file is C++ included by its matching network header; it is not an external binary or generated runtime dependency.

## Repository layout

```text
src/physics/network/
├── timmes_common/   shared constants, screening, rate assembly and AD support
├── iso7/            7-species network
├── aprox13/         13-species alpha-chain network
├── aprox19/         19-species network
└── aprox21/         21-species network
```

The detailed implementation and maintenance notes are in [`src/physics/network/TIMMES_NETWORKS_TECHNICAL_NOTE.zh-CN.md`](src/physics/network/TIMMES_NETWORKS_TECHNICAL_NOTE.zh-CN.md).

This is an **ARCH integration module**, not a general standalone burn solver. The nuclear-rate and RHS formulas have no external network-library dependency, while `TimmesNetworkSupport.h` intentionally uses ARCH's runtime parameters, species manager, and dense/sparse matrix wrappers. To reproduce the validated integration, place this tree at `src/physics/network/` in ARCH and wire the selected `Net*.h/.cpp` files into ARCH's network dispatch and CMake source list. The exact integrated revision is linked under [Provenance](#provenance-and-citation).

## Validation

The source snapshot was validated on JAIST with GCC 13 in a full Debug ARCH build with HDF5 enabled.

| Network | Species | RHS relative error | Jacobian relative error | LHS relative error |
| --- | ---: | ---: | ---: | ---: |
| `iso7` | 7 | `2.383749e-15` | `2.940236e-15` | `2.856278e-15` |
| `aprox13` | 13 | `3.596869e-14` | `2.850247e-13` | `2.850309e-13` |
| `aprox19` | 19 | `9.991735e-15` | `5.467477e-15` | `5.492012e-15` |
| `aprox21` | 21 | `3.997144e-14` | `1.092265e-14` | `1.091782e-14` |

All direct RHS, full Jacobian and backward-Euler LHS comparisons are below the requested `1e-12` threshold. The largest value is the `aprox13` LHS error, `2.850309e-13`.

A three-step burning run with NSE bypass disabled was also compared at 1 and 16 OpenMP threads. All 20 HDF5 datasets were bitwise identical at output steps 0--3, and newly produced Ne20 confirmed that the network/ODE/LU path was exercised.

CellularDet one-step smoke runs with Helmholtz EOS, `BE_NR`, DenseLU, RK3, HLLC and PPM completed for both larger networks and reported the correct values:

- `aprox19`: `Species Count: 19`
- `aprox21`: `Species Count: 21`

More detail is available in [docs/VALIDATION.md](docs/VALIDATION.md).

## Current scope and limitations

- The parity results cover analytic rates with full screening and weak rates disabled (`weak_on = 0`).
- Weak-rate hooks `irpen`, `irnep`, and `irn56ec` remain zero stubs because the current ARCH interface does not provide the electron degeneracy input required by the original routines.
- Timmes `sneut5` neutrino-loss physics is not included in this snapshot.
- The original tabulated-rate mode is not included; this port uses the analytic-rate path.
- OpenMP and MPI use the same process-local, re-entrant network functions. MPI domain decomposition remains the host application's responsibility.
- CUDA device qualification and cross-architecture validation currently cover `aprox13`; `iso7`, `aprox19`, and `aprox21` are not yet claimed as device-verified CUDA networks.
- This repository preserves the repaired network module. Build-system, OpenMP scheduling, ODE, EOS and dispatch changes remain in the linked ARCH integration revision.

## Provenance and citation

The rate and RHS expressions were translated directly from the supplied `public_iso7.f90`, `public_aprox13.f90`, `public_aprox19.f90`, and `public_aprox21.f90` sources by F. X. Timmes. The official source page is [Cococubed: Nuclear Reaction Networks](https://cococubed.com/code_pages/burn.shtml). These equations are not generated from `pynucastro`.

The C++ host interfaces were developed for [Shiro-Akane/ARCH](https://github.com/Shiro-Akane/ARCH). The validated integrated source is ARCH commit [`f637597`](https://github.com/Shiro-Akane/ARCH/commit/f63759752c2d0c0d8332cd109febb88a6843b7d9).

When using the network method scientifically, cite:

> F. X. Timmes, “Integration of Nuclear Reaction Networks for Stellar Hydrodynamics,” *The Astrophysical Journal Supplement Series* 124(1), 241–263 (1999). [doi:10.1086/313257](https://doi.org/10.1086/313257)

## Related project

[Shiro-Akane/ARCH](https://github.com/Shiro-Akane/ARCH) — the stellar-hydrodynamics project for which this repaired network integration was developed.
