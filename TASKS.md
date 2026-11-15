# TASKS
Markers: [ ] todo, [~] in progress, [x] done, [!] blocked (add reason).
Format: `- [ ] T-XXX: task — Done when: check`
Rule: ONE task at a time. After each task: run `run_all_tests`, update this file, overwrite PROGRESS.md, commit.
All speeds in km/h at the API surface unless a name ends in `_mps`. Frequencies in Hz.

## M0 — Setup
- [ ] T-001: Create `matlab/default_config.m` (returns struct with defaults from PRD §5) and `matlab/tests/run_all_tests.m` (runs every `test_*.m` in tests/, prints PASS/FAIL, errors on failure) with one sanity test `test_config.m` — Done when: `run_all_tests` prints 1 PASS.

## M1 — Physics and baseline signal
- [ ] T-002: `doppler_from_speed.m` and `speed_from_doppler.m` (inputs km/h or Hz, plus fc, c) + `test_doppler_equation.m` — Done when: 100 km/h at 24.125 GHz gives 4470.685 Hz (±0.001) and round trip error < 1e-9.
- [ ] T-003: `validate_config.m` (checks fs>0, duration>0, A>0, fc>0, max Doppler < fs/2, NFFT>=1) + `test_validate_config.m` — Done when: valid config passes, 4 bad configs each raise an error with a clear message.
- [ ] T-004: `generate_doppler_signal.m` (returns x, t, fD, N) + `test_signal.m` — Done when: peak of a zero-padded FFT of the clean signal is within 1 bin of fD, and length N = round(fs*duration).

## M2 — Estimator
- [ ] T-005: `estimate_doppler_fft.m` (window already applied by caller; NFFT; one-sided; exclude DC; search in [fmin,fmax]; returns fD_est, f axis, P, `valid`) + `test_estimator.m` — Done when: on-bin tone returns exact bin frequency; no mirror peak selected; pure DC/zeros input returns valid=false.
- [ ] T-006: `estimate_vehicle_speed.m` (fD_est -> m/s, km/h, abs error, percent error; ref=0 gives NaN percent error, not a crash) — Done when: tests for known fD and zero-ref pass.
- [ ] T-007: `main_simulation.m` baseline demo: clean signal at one speed, plots (time waveform, one-sided spectrum with detected peak marked), prints a results line, runs E1 sweep over the 6 reference speeds, saves `results/tables/E1_baseline.csv` and figure — Done when: running it produces the CSV and figures with no errors. (REVIEW POINT 1 — user pastes outputs to strong model.)

## M3 — Windows
- [ ] T-008: `make_window.m` (rect, hann, hamming, blackman; formulas typed out; `periodic`/`symmetric` option; default periodic) + `test_windows.m` — Done when: length correct, all finite, hann/hamming/blackman match known first/middle values, no toolbox calls.
- [ ] T-009: `apply_window.m` (returns xw and coherent gain) — Done when: gain of rect = 1, hann ≈ 0.5.
- [ ] T-010: `run_E4_windows.m`: off-bin tone (choose speed so fD is not on a bin), 4 windows, plots of window shape, windowed waveform, dB spectrum; metrics: fD_est, abs/pct error, main-lobe width, peak side-lobe level; saves `E4_windows.csv` and figures — Done when: CSV has 4 rows and figures saved. (REVIEW POINT 2)

## M4 — Disturbances
- [ ] T-011: `add_awgn.m` (SNR defined as mean power of the clean signal divided by noise variance, in dB; uses `rng(seed,'twister')`; returns y and the noise) + `test_awgn.m` — Done when: achieved SNR within 0.3 dB of target for N=1e5 samples; same seed gives identical samples.
- [ ] T-012: `add_rain_clutter.m` — PROPOSED model in DECISIONS D-009. Confirm with user BEFORE coding. Done when: user-approved model implemented, labelled "illustrative synthetic rain-related radar clutter", parameters documented in file header, seeded, `test_clutter.m` passes.
- [ ] T-013: `run_E2_awgn.m` (SNR 20/10/5/0 dB, several seeds, e.g. 50 trials each: mean and std of error, success rate of valid peak) — Done when: `E2_awgn.csv` + error-vs-SNR plot saved.
- [ ] T-014: `run_E3_clutter.m` (clutter strength sweep; record whether target peak stays the largest; show time + spectrum per condition) — Done when: `E3_clutter.csv` + figures saved. (REVIEW POINT 3)

## M5 — STFT, FFT resolution, runtime
- [ ] T-015: `compute_stft.m` (manual framing, window, hop, returns S, t, f; power in dB) + `test_stft.m` — Done when: constant tone gives a ridge within 1 bin of fD in every frame; axes correct.
- [ ] T-016: `run_E6_stft.m`: constant speed and linearly varying speed (e.g. 40 -> 100 km/h). Phase must be the integral of the instantaneous frequency (cumsum), NOT cos(2*pi*f(t)*t) — Done when: two spectrogram figures saved, ridge flat vs ramping.
- [ ] T-017: `run_E5_resolution.m`: (a) fixed N, NFFT = N…8N (zero padding), (b) varying duration; record bin spacing fs/NFFT, error, runtime — Done when: `E5_resolution.csv` + plot saved.
- [ ] T-018: `benchmark_runtime.m` + `run_E7_runtime.m` (stage-wise tic/toc: generate, window, FFT, peak+speed; NFFT = 256…4096, 30 repeats after 5 warm-ups; mean ± std; no plotting inside timed region) — Done when: `E7_runtime.csv` + plot saved. (REVIEW POINT 4)

## M6 — Final evaluation
- [ ] T-019: `run_experiments.m` runs E1–E7, writes all CSVs, saves config + seed to `results/tables/config_used.txt` — Done when: single command reproduces all results.
- [ ] T-020: `docs/results_summary.md` generated from the CSVs: tables and one-line FACTS only (no report prose) — Done when: every number matches a CSV row.
- [ ] T-021: `docs/viva_questions.md`: 25 likely viva questions with short answers, grounded in this code — Done when: covers Doppler eq., zero padding, leakage, windows, SNR, STFT, complexity vs measured time. (REVIEW POINT 5)

## Phase 2 — Optional web dashboard (DO NOT START without user approval)
- [ ] T-101: Ask user whether the React/FastAPI dashboard is required. If not, delete this section.
