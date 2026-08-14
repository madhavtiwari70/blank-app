# Divi Demo Console — all 6 current demos

Card #208: pick a demo, edit its data file, click Run, get real results
from real Divi execution. No notebook, no code changes.

## Important context

`cluster_maxcut` and `molecular_ground_state` (previously in divi-demos)
were removed in the Aug 3, 2026 "Divi 0.13" refactor commit — the repo's
own README is currently out of date and still lists them. This console
covers the 6 demos that actually exist in divi-demos today.

## Status — all individually verified with real Divi execution

| Demo | Category | Covers |
|---|---|---|
| Spin Dynamics (TFIM) | Time Evolution | Full demo (all 3 physical regimes) |
| Economic Load Dispatch | Optimization · PCE-VQE | 3-generator scenario |
| Quantum-Guided Cluster | QAOA | Full demo |
| Travelling Salesman | QAOA · QUBO | "Part A: Direct QAOA" only |
| Minimum Birkhoff Decomposition | Optimization | Full demo |
| Portfolio Optimization | QAOA | Small synthetic portfolio only |

### How to run it on your own machine

```
curl -LsSf https://astral.sh/uv/install.sh | sh
uv sync
uv run streamlit run streamlit_app.py
```

Some demos have a `backend.use_cloud` flag in their data file — set it to
`true` and set `QORO_API_KEY` in your environment to run on QoroService
instead of the local simulator.

## How it's structured

```
streamlit_app.py                    <- UI + demo registry (add new demos here)
data/
  <demo>.yaml                       <- what someone actually edits
demos/
  <demo>/
    <demo>_original.py              <- the REAL script from divi-demos, unmodified
    <demo>_wrapper.py               <- run_from_config(cfg), reuses the original's
                                        real functions directly
```

**Important — module isolation:** several demos ship their own file named
the same thing (e.g. both `spin_dynamics` and `quantum_guided_cluster` have
a `plotting.py`, with completely different contents). `streamlit_app.py`
only adds the *currently selected* demo's folder to `sys.path`, and clears
any previously-imported demo modules before doing so — this prevents
Python from accidentally loading the wrong same-named file from a
different demo's folder. Don't add all demo folders to `sys.path` at once
if you extend this app; that reintroduces the collision.

## Known extra dependencies

- `docplex` + `cplex` — required only for Minimum Birkhoff Decomposition
- `dimod` + `dwave-neal` — required by Economic Load Dispatch and Portfolio Optimization

## How to extend a demo to cover more of its original scenarios

Several original scripts run 2-3 scenarios back-to-back (e.g. Travelling
Salesman runs a small direct-QAOA case, a larger partitioned case, and a
PCE-compressed case in one script). Each wrapper here covers the core
scenario, not every variant, to keep each config schema clear. To extend:

1. Look at the demo's `_original.py` file — find the extra scenario in its
   `if __name__ == "__main__":` block.
2. Add the extra parameters to that demo's `data/<demo>.yaml`.
3. Extend `run_from_config()` in that demo's `_wrapper.py`.
4. Add a render branch in `streamlit_app.py` if the output shape changes.
