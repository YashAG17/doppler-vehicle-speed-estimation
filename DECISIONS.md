# DECISIONS
Append-only. Only significant decisions are logged. Older entries may be condensed into a summary at the top.
Status PROPOSED = needs user confirmation before implementation.

## D-001 · 2026-11-15 · Title and scope fixed
Why: course requires a DSP project; Handoff defines scope. Rejected: FMCW, computer vision, ML (out of brief). Affects: everything.

## D-002 · 2026-11-15 · CW Doppler, baseband input, magnitude-only speed
Why: single-tone CW with a real channel gives |fD| only; keeps viva defensible. Rejected: I/Q direction, range. Affects: generate_doppler_signal, estimator.

## D-003 · 2026-11-15 · MATLAB is the reference; web layer deferred
Why: course lab uses MATLAB; web is optional (Handoff M6). Rejected: building React/FastAPI now. Affects: repo layout. Revisit at T-101.

## D-004 · 2026-11-15 · No MATLAB toolbox dependency
Why: runs on any install; windows, AWGN, STFT, peak search coded from formulas are better for the viva. Rejected: hann/hamming/blackman/awgn/spectrogram/findpeaks. Affects: make_window, add_awgn, compute_stft, estimator.

## D-005 · 2026-11-15 · Default parameters (PROPOSED, confirm with user)
fc=24.125 GHz, fs=20 kHz, duration 0.2 s, NFFT 4096, A=1, phi=0.3 rad, seed 42, speeds 20–120 km/h, SNR 20/10/5/0 dB, STFT L=512 H=128.
Why: fD at 200 km/h = 8.94 kHz < fs/2; bin spacing 4.88 Hz = 0.109 km/h. Rejected: 10.525 GHz (HB100) as less typical for traffic radar. Affects: default_config.

## D-006 · 2026-11-15 · Estimator rules
One-sided spectrum, DC bin excluded, search only in the band implied by the speed validity range (5–200 km/h), bin-max peak by default; failure returns valid=false. Parabolic interpolation is OPTIONAL, off by default, only added if the user approves. Rejected: silent fallback values. Affects: estimate_doppler_fft.

## D-007 · 2026-11-15 · Windows periodic by default
Why: periodic windows are the correct choice for spectral analysis; symmetric available by flag and stated in report. Affects: make_window.

## D-008 · 2026-11-15 · SNR convention
SNR_dB = 10*log10(mean(x_clean.^2) / noise_variance); for a tone of amplitude A that is A^2/2 over sigma^2. Verified by test. Affects: add_awgn, E2.

## D-009 · 2026-11-15 · Rain clutter model (PROPOSED, confirm before T-012)
Band-limited Gaussian-shaped coloured noise centred at a low frequency (default 600 Hz, sigma 200 Hz, made by shaping white noise in the frequency domain), scaled to a configurable clutter-to-signal ratio (CSR, dB), seeded. Labelled "illustrative synthetic rain-related radar clutter, not a validated precipitation-scattering model". Rejected: acoustic rain/babble audio; EM scattering model (too large). Affects: add_rain_clutter, E3.

## D-010 · 2026-11-15 · Report authorship
Students write all report prose; models give code, tables, figures, facts. Why: plagiarism/AI-content must be < 10 %. Affects: docs/, report.

## D-011 · 2026-11-15 · Varying-speed signal for STFT
Phase = 2*pi*cumsum(f_inst)/fs, not cos(2*pi*f(t)*t). Why: the latter gives the wrong instantaneous frequency. Affects: run_E6_stft.
