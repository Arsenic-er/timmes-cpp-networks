# Validation record

This document records the verification scope of this source snapshot. It is not a claim about untested configurations.

## Environment

- Host: JAIST Linux CPU/GPU server
- Host application: ARCH with HDF5 and Helmholtz EOS
- Integrated revision: [`Shiro-Akane/ARCH@23cfe9d`](https://github.com/Shiro-Akane/ARCH/commit/23cfe9d999f57043d54544973b999ac3c774ab73)
- Reference source: supplied Timmes `public_iso7.f90`, `public_aprox13.f90`, `public_aprox19.f90`, `public_aprox21.f90`, and `public_nse.tbz`
- Thermodynamic states: `T = 1e9, 2e9, 5e9 K`; `rho = 1e6, 1e8 g cm^-3`

## Species registry

| Selected network | Expected | Observed |
| --- | ---: | ---: |
| `iso7` | 7 | 7 |
| `aprox13` | 13 | 13 |
| `aprox19` | 19 | 19 |
| `aprox21` | 21 | 21 |

Names and ordering were checked against the corresponding network headers.

## Direct original-Fortran parity

The comparison includes the species RHS, complete analytic Jacobian (including the Timmes temperature row and column), backward-Euler `LHS = I - dt * J`, nuclear-energy derivatives, and Helmholtz `cv`.

| Network | RHS relative error | Jacobian relative error | LHS relative error | Helmholtz cv relative error |
| --- | ---: | ---: | ---: | ---: |
| `iso7` | `2.383748949565e-15` | `2.940236373588e-15` | `2.856278354823e-15` | `5.705278821529e-16` |
| `aprox13` | `3.596869184109e-14` | `2.850246815720e-13` | `2.850309327978e-13` | `1.331631293002e-15` |
| `aprox19` | `9.991734850962e-15` | `5.467476955907e-15` | `5.492012250384e-15` | `3.753488177566e-16` |
| `aprox21` | `3.997143540823e-14` | `1.092264838562e-14` | `1.091781737534e-14` | `3.947012873172e-16` |

All reported quantities are below the required `1e-12` threshold. The largest value is the `aprox13` LHS difference, `2.850309327978e-13`.

## Temperature derivative convention

The ODE uses the original Timmes self-heating convention:

```text
dT/dt = enuc / cv
J(T,j) = (d enuc / dX_j) / cv
J(T,T) = (d enuc / dT) / cv
```

The species temperature column and nuclear-energy temperature row use analytic reaction-rate derivatives; no temperature finite difference is used. Matching the original Timmes Jacobian, the implementation does not add quotient-rule derivatives of `cv` to the ODE Jacobian.

## Generic online NSE

`src/physics/nse/nse_solver.h` implements the Timmes Saha equilibrium equations as `NSESolver<NetType>`. The solver reads `NUM_SPECIES`, `AION`, `ZION`, total binding energies, and statistical weights from the selected network; no isotope count or 47-species data table is hard-coded in the algorithm.

The validation covered:

- eight direct comparisons with the original 47-species Timmes program over `T = 2.5e9--1e10 K`, `rho = 1e6--1e9 g cm^-3`, and `Ye = 0.47--0.55`;
- 48 fixed-state solves across all four translated networks;
- 84 variable-`Ye` states for each of `aprox19` and `aprox21` over `Ye = 0.40--0.60`;
- an `aprox19` projection coupled to the real Helmholtz EOS at `rho = 4.322e7 g cm^-3`, `T_old = 4.67e9 K`;
- a 64 x 16 high-temperature CellularDet step at one and sixteen OpenMP threads.

The worst single-species absolute mass-fraction difference from the original 47-species program was `4.897193761622e-13`. The aggregate L1 difference in that state was `1.363608769782e-12`; this aggregate must not be misreported as a sub-`1e-12` quantity. The worst variable-`Ye` mass and charge residuals were `6.49e-16` and `6.98e-13`, respectively. The real-EOS relative energy-closure residual was `7.5678598227e-14`. All 26 datasets from the one- and sixteen-thread CellularDet outputs were bitwise identical.

This is network-constrained NSE. `iso7` and `aprox13` contain only `Z/A = 0.5` nuclei and therefore admit the restricted equilibrium only at `Ye = 0.5`. They must not be treated as replacements for the full 47-species equilibrium set.

## OpenMP determinism

A 64 x 16 `aprox13` case ran for three burning steps with NSE bypass disabled and a `3e9 K` hot state. One-thread and sixteen-thread output had the same 20 HDF5 datasets and was bitwise identical at steps 0, 1, 2 and 3. Newly produced Ne20 confirmed execution of the network, ODE and DenseLU burn path.

Parallel execution is across independent cells with dynamic OpenMP scheduling. Network, Jacobian, ODE and small dense-LU loops use SIMD where appropriate; the implementation does not create nested thread teams for an individual 8--22 equation matrix.

## MPI and CUDA scope

- The network functions use process-local inputs and stack-local work arrays, so they can be called inside an MPI-decomposed host application. MPI communication and domain decomposition are not implemented by this module.
- The device-qualified and cross-architecture validated CUDA network is currently `aprox13`.
- For translated `aprox13`, CPU-vs-CUDA comparisons over the same six states produced RHS `5.207022670405e-13`, Jacobian `5.094162745733e-13`, LHS `5.094258977149e-13`, and Helmholtz `cv` `7.834375032555e-15`.
- CUDA verification of `iso7`, `aprox19`, and `aprox21` is not claimed in this snapshot.

## Exclusions and physical scope

- Weak-rate hooks `irpen`, `irnep`, and `irn56ec`
- `sneut5` neutrino losses
- Tabulated reaction-rate mode
- A resolved, converged cellular-detonation solution

The first three exclusions match the documented analytic-rate validation scope. Local RHS/Jacobian/LHS parity does not by itself validate a long-time multidimensional detonation; ZND/CJ initialization or adequate mesh/AMR convergence remains necessary.
