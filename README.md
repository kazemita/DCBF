# DCBF: re-examining the Dual-Core Beamformer simulations

Python code and simulation outputs for the commentary

> *Re-examining the Dual-Core Beamformer: a correct optimum, a fragile search, and amplitudes set by data length*
> (submitted to NeuroImage, Comments and Controversies)

which re-examines the simulations in

> Diwakar, M., Huang, M.-X., Srinivasan, R., et al., 2011. Dual-Core Beamformer for obtaining highly correlated
> neuronal networks in MEG. *NeuroImage* 54, 253–263. https://doi.org/10.1016/j.neuroimage.2010.07.023

The repository contains an independent implementation of the Dual-Core Beamformer (DCBF) and its modified Powell
search, the scripts that repeat the simulations behind Tables 1 and 2 of that paper, the additional analyses reported
in the commentary, and the raw outputs of every run, so that all numbers and both figures can be regenerated.

## Requirements

Python 3.10 or later with NumPy, SciPy and Matplotlib (`pip install -r requirements.txt`). No MEG toolbox is needed.

## Quick start (about one minute)

All raw results are included in `results/`, so the summaries and figures can be regenerated without re-running any
simulation:

```bash
python3 setup_model.py                 # forward model -> model.npz (about 15 s)
python3 aggregate.py N5000 results/log_N5000.txt results/log_N5000_reps.txt   # Table 1, pair A
python3 aggregate.py N5000_B results/log_N5000_B.txt                          # Table 1, pair B (likewise C)
python3 aggregate_t2.py results/t2_*.txt                                      # Table 2, all pairs
python3 aggregate_three.py             # three correlated sources
python3 aggregate_compare.py           # least-squares fit vs DCBF on the same data
python3 paper_table1_check.py          # internal consistency of the published Table 1
python3 paper_table2_check.py          # internal consistency of the published Table 2
python3 make_figures.py                # Figures 1 and 2 -> figures/
```

## What is in the repository

| File | Purpose |
|---|---|
| `dcbf.py` | Forward model (spherical conductor, Sarvas 1987; 306-channel Vectorview-like array), 7 mm source grid, SVD reduction of each lead field to two orientations, the DCBF (eqs. 9–15 of the paper), the modified Powell search with random restarts and a taboo list (eq. 16), and an exhaustive pair search |
| `sim.py` | Simulator: 30 Hz sinusoidal dipoles (10 and 20 nAm along the two orientations), white sensor noise at a set SNR, separate noise-only segment for the noise covariance; Fourier amplitude read-out |
| `setup_model.py` | Builds `model.npz` (grid of 5,278 nodes and reduced lead fields) |
| `run_table1.py` | Table 1 conditions: SNR 4 to 0.25, fully correlated pair |
| `run_table2.py` | Table 2 conditions: SNR 2, correlation 100/86.6/75/50% (phase shift of one dipole, r = cos φ) |
| `amp_vs_length.py` | Amplitude at the true pair versus data length (1,000–20,000 samples), SNR and correlation |
| `paired_Z.py` | Pseudo-Z versus correlation with identical noise across correlations (paired design) |
| `inspect_misses.py` | Where the wrong pair lies in the Table 2 realisations where the DCBF picked a wrong pair |
| `run_three.py`, `run_three2.py` | Three correlated sources (30 Hz, phases 0/22.5/45°, SNR 0.25): DCBF, and least-squares fits |
| `lsq_methods.py`, `run_compare.py` | Least-squares localisation with alternating re-estimation, run on the same data as the DCBF |
| `aggregate*.py`, `paper_table*_check.py`, `make_figures.py` | Summaries, consistency checks and figures |
| `results/` | Raw outputs: one JSON line per simulated realisation |
| `figures/` | Figures 1 and 2 of the commentary (created by `make_figures.py`) |

## From the commentary to the code

| Statement in the commentary | Script | Output |
|---|---|---|
| True pair is the global maximum at every SNR (21 of 21 exhaustive searches) | `run_table1.py` (third argument `1`) | `results/log_N5000*.txt` (`brute_ok`) |
| Restarts per success and search failures at SNR 3–4 (Fig. 1A) | `run_table1.py` | `results/log_N5000*.txt`, `results/log_extra*.txt` |
| Orientation and amplitude accuracy, drift with SNR | `run_table1.py`, `aggregate.py` | `results/log_N5000*.txt` |
| Wrong pair at 75% and 50% correlation | `run_table2.py`, `inspect_misses.py` | `results/t2_*.txt` |
| Pseudo-Z rises as correlation falls (Fig. 1B) | `paired_Z.py` | `results/pairedZ_*.npy` |
| Amplitude law (1 − m/N)·√((1 + r)/2) (Fig. 2A) | `amp_vs_length.py` | `results/amp_vs_length.txt` |
| Published Table 2 amplitudes follow √((1 + r)/2) (Fig. 2B) | `make_figures.py` | `figures/Figure2.*` |
| 1,000-sample comparison (amplitude errors 25–34%) | `run_table1.py 1000 1 0` | `results/log_N1000.txt` |
| Three correlated sources: two found, 48% amplitude suppression | `run_three.py`, `aggregate_three.py` | `results/three.txt` |
| Least-squares fit finds the true pair in all 137 realisations | `run_compare.py`, `aggregate_compare.py` | `results/compare.txt` |

## Re-running the simulations

Run from the repository folder after `python3 setup_model.py`. Times are for a two-core machine; the Powell search
(1,000 restarts) dominates and takes about one minute per realisation. Setting `OPENBLAS_NUM_THREADS=1` and running
several commands in parallel is the fastest option.

```bash
# Table 1 (arguments: samples, realisations, exhaustive check 0/1, first realisation, tag, source pair)
python3 run_table1.py 5000 1 1 0 "" A   > results/log_N5000.txt
python3 run_table1.py 5000 4 0 1 _reps A > results/log_N5000_reps.txt
python3 run_table1.py 5000 1 1 0 _B B   > results/log_N5000_B.txt
python3 run_table1.py 5000 1 1 0 _C C   > results/log_N5000_C.txt
SNRS=4.0,3.0 python3 run_table1.py 5000 10 0 5 _extraA A > results/log_extraA.txt
SNRS=4.0,3.0 python3 run_table1.py 5000 5 0 1 _extraB B  > results/log_extraB.txt
SNRS=4.0,3.0 python3 run_table1.py 5000 5 0 1 _extraC C  > results/log_extraC.txt
python3 run_table1.py 1000 1 0          > results/log_N1000.txt

# Table 2 (arguments: samples, realisations, exhaustive check 0/1, first realisation, source pair)
python3 run_table2.py 5000 5 1 0 A > results/t2_A.txt
python3 run_table2.py 5000 1 1 0 B > results/t2_B.txt
python3 run_table2.py 5000 1 1 0 C > results/t2_C.txt
CORRS=0.866,0.75,0.5 python3 run_table2.py 5000 10 0 5 A > results/t2_A_extra.txt
CORRS=0.866,0.75,0.5 python3 run_table2.py 5000 5 0 1 B  > results/t2_B_extra.txt
CORRS=0.866,0.75,0.5 python3 run_table2.py 5000 5 0 1 C  > results/t2_C_extra.txt

# Additional analyses
python3 amp_vs_length.py > results/amp_vs_length.txt     # about 10 min
python3 paired_Z.py                                      # about 3 min
python3 inspect_misses.py
python3 run_three.py 5 > results/three.txt               # about 25 min
python3 run_three2.py  > results/three2.txt
python3 run_compare.py > results/compare.txt             # about 15 min
```

Random seeds are fixed in each script, so re-running reproduces the stored outputs.

## Differences from the original study

* A spherical head model and a volumetric 7 mm grid (5,278 nodes) replace the subject-specific boundary-element model
  and cortical grid of the original paper, which were not available.
* Source locations, data length and noise realisations are not reported in the paper and were chosen here (three
  source pairs, 5,000 samples at 1 kHz). The data-length analysis shows how amplitudes and pseudo-Z depend on this choice.
* Absolute pseudo-Z values and search counts should therefore not be compared one-to-one with the paper; trends and
  failure modes are the comparison of interest.

## Citation

If you use this code, please cite the commentary (reference to be added on publication) and this repository.

## Licence

[MIT]
