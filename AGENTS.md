# AGENTS.md

## Project summary
"Doppler-Based Vehicle Speed Estimation Using Frequency Analysis" — a DSP lab project (LNMIIT Jaipur).
We simulate a sampled CW Doppler baseband signal for a known radial speed, estimate fD with a windowed
FFT peak detector, convert to speed, and study AWGN, synthetic rain clutter, windows, FFT resolution,
STFT and runtime. Full spec: PRD.md. Original brief: the Handoff .docx (kept outside git if large).

## Stack (do not change versions or add dependencies without logging a decision)
- MATLAB (user's version, R2021a+). NO toolbox functions: no hann/hamming/blackman/awgn/spectrogram/findpeaks.
- Git. Phase 2 (web) is NOT approved; do not create backend/ or frontend/.

## Commands
- Run all tests (from `matlab/`): `run_all_tests`
- Baseline demo: `main_simulation`
- All experiments: `run_experiments`
- The agent usually cannot run MATLAB. If you cannot execute code, SAY SO, never claim it ran,
  and give the user the exact command to run plus what output to paste back for verification.

## Folder structure and conventions
- `matlab/` one function per file, file name = function name, snake_case.
- `matlab/tests/test_*.m` assert-based; `run_all_tests.m` runs them.
- `results/tables/*.csv`, `results/figures/*.png`, `docs/`.
- Every function: header comment with purpose, inputs (units), outputs (units).
- Units: speed km/h at the API (variables ending `_mps` are m/s), frequency Hz, time s, SNR dB.
- Config comes from `default_config()`; no magic numbers in functions.
- Randomness only via `rng(seed,'twister')`; always save the seed.
- Plot code stays out of algorithm functions; timing excludes plotting and file I/O.
- Keep code short, readable and commented. This is an undergraduate viva project.

## Project-specific hard rules
- CW Doppler only. No FMCW, range, multi-target, direction sign, ML, computer vision, babble audio.
- DSP input is baseband Doppler, not the RF carrier. Enforce fD < fs/2.
- Real single channel gives speed magnitude only. Never claim direction detection.
- Never fabricate results, timings, accuracy, citations, hardware tests or novelty. Label all results "simulated".
- Rain clutter is "illustrative synthetic rain-related radar clutter", never a validated EM model.
- Do NOT write report paragraphs, abstract or literature review. Students write those (AI-content limit).
  You may produce code, CSVs, plots, tables and short factual notes in docs/results_summary.md.
- If a requirement is physically ambiguous or changes scope, ask the user first.

## On session start (always)
1. Read PROGRESS.md, then TASKS.md, then run `git log --oneline -15`.
2. Read DECISIONS.md entries relevant to the area you're working on.
3. Read PRD.md sections only as needed.
4. Pick up the task marked [~], or else the first [ ] task. Work on ONE task at a time.

## Rules
- Never reverse or contradict a decision in DECISIONS.md. If you think it's wrong,
  stop and tell the user why. Don't change it yourself.
- Don't rewrite or refactor code outside your current task.
- Don't remove or weaken tests to make them pass.
- Never put secrets or API keys in code or docs. Use .env and keep .env.example
  updated.
- If you're unsure about a requirement, ask the user. Don't guess.

## After EACH completed task (do this right away, not at the end of the session)
1. Run the tests and the linter. Fix failures before marking the task done.
2. Update TASKS.md (mark [x], or [!] with a reason).
3. OVERWRITE PROGRESS.md using its template: current state, what was just done,
   known issues, the exact next step.
4. If you made a significant design choice (library, architecture, data shape,
   API contract, tradeoff), append it to DECISIONS.md. Don't log routine actions.
5. Commit: `git add -A && git commit -m "T-XXX: <what changed>"`

## If you are about to run out of context or hit a limit
Stop coding right away and update PROGRESS.md with exactly where you stopped,
including half-finished work and files touched.

## Verification loop with the strong model
After each task, end your reply with a block titled `PASTE TO REVIEWER` containing: the task id,
the full code of the files you created/changed, and the exact MATLAB command to run. The user will run it,
paste the console output and figures' descriptions back, and have the strong model check them against PRD/DECISIONS.
