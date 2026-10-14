**Dual-Core Beamformer (DCBF) — re-implementation and Table 1 check**

Re-implementation of Diwakar, Huang et al., *Dual-Core Beamformer for obtaining highly
correlated neuronal networks in MEG*, NeuroImage 54 (2011) 253–263, in Python/NumPy.

## Files

`dcbf.py` : forward model (Sarvas spherical conductor, 306-ch Vectorview-like array), 7 mm source grid, SVD reduction to m×2, the DCBF itself (eqs 9–15), modified Powell search with taboo list (eq 16), brute-force search 
`setup_model.py` : builds the grid (5,278 nodes) and lead fields → `model.npz` 
`sim.py` : sinusoidal dipole pairs + white sensor noise at a set sensor-domain SNR; Fourier amplitude read-out (simulator)
`run_table1.py` : Table 1 experiment. `python3 run_table1.py N_SAMPLES N_REALISATIONS BRUTE(0/1) [REP0] [TAG] [CONFIG A/B/C]` 
`aggregate.py` : pools logs and prints the comparison with the paper's Table 1 
`paper_table1_check.py` : internal-consistency check of the numbers printed in the paper's Table 1 
`log_*.txt` : raw per-run results (one JSON line per SNR × realisation) 

Run: `python3 setup_model.py && python3 run_table1.py 5000 5 1 0 "" A && python3 aggregate.py A log_*.txt`

## Implementation notes

* **DCBF.** `Ldual = [L1 L2]` (m×4 after SVD reduction). `K = Q⁻¹R` with `Q = Ldualᵀ C⁻¹ Ldual`,
  `R = Ldualᵀ C⁻¹ ε C⁻¹ Ldual`; `Z = 1/min eig(K)` (computed as the largest generalised
  eigenvalue of (Q,R) via Cholesky, batched over all partners j); `P = 1/min eig(Q)`,
  `O = Umin`; `p(t) = O (P C⁻¹ Ldual O)ᵀ b(t)`.
* **Powell search.** Random r1 → r2 = argmax f(r1,·) → r1 = argmax f(·,r2) → … until the pair is
  stable. 1000 random restarts. A global taboo list stores every traversed pair; a restart that
  reaches a traversed pair stops and inherits that search's end point.
* **"Average # searches"** is taken as restarts needed per success = 1000 / (number of restarts
  that ended on the true pair); time = that × measured seconds per restart.
* **Simulation (Table 1 conditions).** One pair, 30 Hz, 100 % correlated (same phase), each
  dipole 10 nAm and 20 nAm along its two SVD tangential directions (orientation ratio 0.5),
  SNR = total signal power / total noise power, 1 kHz sampling, separate noise-only segment for ε.

## Deviations from the paper 

* Spherical head instead of subject BEM; volumetric 7 mm grid (5,278 nodes, close to the paper's
  "5000 dipole grid") instead of a FreeSurfer gray-matter surface.
* Dipole locations, segment length, sensor units/noise model and random seeds are not given in
  the paper; they were chosen here (source pairs A/B/C in `run_table1.py`).
* The segment length matters a great deal (see results): N = 5000 samples reproduces the
  paper's amplitude level; N = 1000 gives ~28 % underestimation.

## Table 2 (varying intra-pair correlation)

`run_table2.py` : Table 2 experiment: SNR 2, 30 Hz, 10/20 nAm, correlation set by phase-shifting dipole 2 (r = cos φ). `CORRS=... python3 run_table2.py N_SAMPLES N_REAL BRUTE REP0 CONFIG` 
`aggregate_t2.py` : pools `t2_*.txt` logs and prints the comparison with the paper
`inspect_misses.py` : shows where the best pair landed in realisations where it was not the true pair 
`paired_Z.py` : pseudo-Z versus correlation with identical noise across correlations (paired comparison) 
`paper_table2_check.py` : internal-consistency check of the paper's Table 2 

Run: `python3 run_table2.py 5000 5 1 0 A > t2_A.txt && python3 aggregate_t2.py t2_*.txt`
