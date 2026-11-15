# PRD — Doppler-Based Vehicle Speed Estimation Using Frequency Analysis

> Source of truth for scope: `Doppler_Vehicle_Speed_Estimation_Claude_Project_Handoff.docx` (the "Handoff").
> Course: DSP Lab, LNMIIT Jaipur (CCE), AY 2026–2027. The title above must not change.

## 1. Overview
Simulate a sampled low-frequency Doppler/baseband signal from a CW Doppler radar for a known vehicle
radial speed, estimate the Doppler frequency with a windowed FFT peak detector, convert it to speed,
and evaluate how disturbances (AWGN, simplified synthetic rain-related clutter), window choice,
FFT size / observation length, and STFT parameters affect the estimate. Measure runtime with tic/toc.

## 2. Goals
- G1 Correct, validated baseline estimator (clean signal).
- G2 Controlled comparative study: windows x disturbances x FFT resolution x duration.
- G3 STFT spectrogram for constant and time-varying speed.
- G4 Reproducible runtime analysis (tic/toc, repeated, mean ± std).
- G5 Every number in the report traces to code + saved config + seed.

## 3. Non-goals (do NOT add)
FMCW, range estimation, multi-target tracking, direction/sign detection, machine learning,
computer vision, real hardware data, babble/audio noise, unapproved noise models.
Web dashboard (React/FastAPI) is Phase 2 and only after the user explicitly approves it.

## 4. Physical model (see Handoff §2)
- fD = 2 * v_r * fc / c ; v_r = fD * c / (2 * fc) ; theta = 0 so v_r = v.
- x[n] = A*cos(2*pi*fD*n/fs + phi). Input to DSP is baseband Doppler, not the RF carrier.
- Real single channel -> speed MAGNITUDE only. Enforce fD < fs/2.

## 5. Default parameters (configurable, from `default_config.m`; confirm with user)
| Parameter | Default | Note |
|---|---|---|
| fc | 24.125e9 Hz | 24 GHz K-band ISM, typical traffic radar. Illustrative. |
| c | 299792458 m/s | |
| Reference speeds | 20, 40, 60, 80, 100, 120 km/h | -> 894 ... 5365 Hz |
| Speed validity range | 5–200 km/h | search band derived from this |
| fs | 20000 Hz | Nyquist 10 kHz; 200 km/h = 8941 Hz (still < Nyquist) |
| Duration | 0.2 s (N = 4000) | main experiments |
| Amplitude / phase | A = 1, phi = 0.3 rad | non-zero phase on purpose |
| NFFT | 4096 | |
| Seed | 42 | `rng(seed,'twister')` |
| AWGN SNR | 20, 10, 5, 0 dB | add -5 dB only if asked |
| Window | rect, hann, hamming, blackman | periodic by default |
| STFT | L = 512, hop H = 128 (75 % overlap) | |
Useful constants at 24.125 GHz: 44.707 Hz per km/h; 0.02237 km/h per Hz.

## 6. Features (v1) and acceptance criteria
Map to Handoff FR-01…FR-09.
- F1 Config + validation (FR-01): invalid config (fs<=0, fD>=fs/2, negative duration...) raises a clear error.
- F2 Signal generation (FR-02): measured tone frequency matches Doppler equation to numerical tolerance.
- F3 Conditions (FR-03): `clean`, `awgn`, `rain` (labelled "illustrative synthetic rain-related radar clutter").
- F4 Windows (FR-04): rect/hann/hamming/blackman, same length, coded from formulas (no toolbox).
- F5 Estimator (FR-05): one-sided spectrum, DC excluded, expected-band search, returns fD, v (m/s, km/h),
  abs/percent error, and a `valid` flag. Failed/uncertain peak is reported, never silently returned.
- F6 STFT (FR-06): manual STFT, correct time/frequency axes, power in dB.
- F7 Runtime (FR-07): tic/toc per stage, algorithm only (no plotting/I/O), repeats with warm-up.
- F8 Export (FR-08/09): CSV + figures + config/seed saved for every experiment.

## 7. Experiments (Handoff §8)
E1 baseline sweep | E2 AWGN vs SNR | E3 rain clutter | E4 windows | E5 FFT resolution & zero padding |
E6 STFT constant vs varying speed | E7 runtime vs N and NFFT.
CSV columns: test_id, ref_kmh, condition, snr_db_or_csr_db, window, N, fs, nfft, fD_est_hz, v_est_mps,
v_est_kmh, abs_err_kmh, pct_err, valid, runtime_s, seed.

## 8. Tech stack
MATLAB (R2021a or newer; confirm user's version). NO toolbox required: no `hann`, `hamming`, `blackman`,
`awgn`, `spectrogram`, `findpeaks`. Plain scripts + functions + assert-based tests.
Git for handoff. (Phase 2 only: Python 3.11, NumPy, SciPy, FastAPI, React, Plotly.)

## 9. Folder structure
```
matlab/   default_config, validate_config, doppler_from_speed, speed_from_doppler,
          generate_doppler_signal, make_window, apply_window, estimate_doppler_fft,
          estimate_vehicle_speed, add_awgn, add_rain_clutter, compute_stft,
          benchmark_runtime, run_experiments, main_simulation
matlab/tests/run_all_tests.m
results/figures  results/tables  docs/
```

## 10. Testing (Handoff §12)
Assert-based scripts, run via `run_all_tests`. Must cover: Doppler equation, on-bin tone, Nyquist rejection,
speed conversion, one-sided spectrum (no mirror peak), DC exclusion + no-valid-peak, window length/finite,
seed reproducibility, achieved SNR, STFT ridge (constant & moving), timing excludes plotting.

## 11. Team and individual contributions (graded)
Fill in real names. Suggested split so each member has a defensible, separate contribution:
- Member 1: physical model, signal generation, estimator, windows (T-001…T-010), E1/E4.
- Member 2: disturbances, STFT, runtime, experiment runner (T-011…T-019), E2/E3/E5/E6/E7.
- Both: literature review, report sections, viva. Each member must be able to explain ALL code.

## 12. Report rules (AI content + plagiarism < 10 %)
- Report text is written by the students in their own words. Models produce code, tables, plots
  and bullet-point FACTS only; they do not write report paragraphs.
- Cite only papers/books the students have actually opened. Seed list to verify (not final):
  M. I. Skolnik, *Introduction to Radar Systems*; M. A. Richards, *Fundamentals of Radar Signal Processing*;
  A. V. Oppenheim & R. W. Schafer, *Discrete-Time Signal Processing*;
  F. J. Harris, "On the use of windows for harmonic analysis with the DFT," Proc. IEEE, vol. 66, no. 1, 1978.
  Add 4–6 recent CW Doppler / traffic radar papers found by the students themselves.
- Contribution claim (honest): a controlled comparative evaluation of windowing, FFT resolution and
  selected disturbances on a baseline CW Doppler estimator. Doppler speed estimation itself is NOT claimed as new.

## 13. Limitations to state (Handoff §13)
Simulation only; clutter is synthetic and not a validated EM model; no range; magnitude only; angle needs
cos(theta) correction; single-peak estimator fails if clutter dominates; zero padding adds no information;
runtime depends on machine/software.

## 14. Out of scope / future
Real radar data, I/Q signed Doppler, FMCW, better clutter model, web dashboard, MATLAB–Python parity checks.
