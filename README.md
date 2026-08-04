# Vendored DPMM (DPMM.jl v0.1.0)

Patched copy of [`ekinakyurek/DPMM.jl`](https://github.com/ekinakyurek/DPMM.jl)
kept under `simulations/vendor/DPMM` so the package ships with the paper code
and any compatibility patches travel to the HPC cluster.

## Why patch

`DPMM.jl` (module `DPMM`) is the only DPM package that installs on the modern
Julia used by this project (`1.11.x`). It is old code that uses symbols
removed from modern `Distributions`/Julia, so the vendored copy is patched:

- `src/DPMM.jl` — drop the removed imports (`GLOBAL_RNG`, `ZeroVector`,
  `NoArgCheck`, `multiply!`, `unwhiten_winv!`) and define compatibility
  shims:
  - `const GLOBAL_RNG = Random.GLOBAL_RNG`
  - `ZeroVector(T, n) = zeros(T, n)`
  - `unwhiten_winv!(W, x) = PDMats.unwhiten!(inv(W), x)`
- `src/Core/niw.jl` — `_wishart_genA!(rng, A, df)` (modern Distributions
  drops the `p` argument).
- `src/Core/dirichletmultinomial.jl` — relax `randlogdir`/`_rand!` from
  `Random.MersenneTwister` to `Random.AbstractRNG` (the default `GLOBAL_RNG`
  is now a `TaskLocalRNG`).
- `Project.toml` — drop unused `AbstractPlotting`/`ArgParse` deps
  (`AbstractPlotting` was removed from the General registry).

Usage (baseline): `DPMM.fit(X; algorithm=DPMM.SplitMergeAlgorithm, α=..., T=...)`
with **columns = observations**.

## Install in the simulations environment (local and HPC)

The package is developed as a path dependency so the local `Manifest.toml`
points at the vendored copy. On a fresh clone (HPC), re-bind it:

```julia
import Pkg
Pkg.activate("simulations")
Pkg.develop(path="simulations/vendor/DPMM")
Pkg.resolve()
```

Then `simulations/Project.toml` lists `DPMM` and the scripts can
`using DPMM`.