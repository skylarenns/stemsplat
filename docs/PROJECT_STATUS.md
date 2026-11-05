# Stemsplat — where things stand (2026-11-05)

Handoff note written after the November 2026 cleanup. Read this first when you come back.

## TL;DR

- `main` = `origin/main` at `483fcdd`: **v0.4.3 source** (hardening + UI reconciliation, PR #51) plus the GPL-3.0 license. The latest *published* release is still **v0.4.2** (GitHub release `v0.4.2-release`, 2026-04-07). v0.4.3 was never released.
- **CI was red from 2026-09-15 to 2026-11-05** because the pip-audit exceptions expired. Fixed on 2026-11-05: patched dependencies were upgraded. The remaining torch-coupled exceptions (torch, setuptools, filelock) expire **2027-02-05**, and clearing them needs a quality-gated torch/Python upgrade.
- Everything that was uncommitted on disk now lives on **local-only** `archive/2026-11/*` branches. Don't push them: the repo is public and some contain logs, a copyrighted WAV, and server details. A git bundle of all of them is at `~/stemsplat-archive/archive-branches.bundle`.
- The app installed in `/Applications/Stemsplat.app` is a **0.5.0 build**. That build came from uncommitted Desktop-clone work (now `archive/2026-11/desktop-iree-wip`), not from `main`.
- None of the text-to-workflow models met the promotion gates, and none ships from `main`.

## What `main` is (v0.4.3)

A Mac-only (Apple Silicon/MPS) stem splitter. It runs as a FastAPI app in `main.py` (~9k lines) with a pywebview desktop shell (`launcher.py`), a packaged UI in `web/` (`index.html`, `mobile.html`), and Roformer/MDX/Demucs inference in `split/`. Supported modes: vocals, instrumental, deux, guitar, background vocals, full mix, drum split 4/6, all stems, boost harmonies, denoise. See `README.md`.

v0.4.3 added the following (full detail in `reports/v0.4.3-hardening.md` and `docs/qa/reconciliation-matrix.md`):
- `stemsplat/` package: SQLite WAL state store (`state.py`), bounded upload admission (`uploads.py`), LAN HTTPS passcode sessions (`security.py`, off by default), model manifest verification (`model_manifest.py`), and signed updates (`updates.py`).
- The editor/waveform work from `origin/v0.5.0` and the hardening from the gauntlet branch, reconciled through the UI-rebuild branch.
- Reproducible ARM64 lock files, pip-audit/SBOM/secret gates, Playwright E2E tests, and `tools/release_gate.py`.
- `quality.py`, a fail-closed `reference_safe` quality and benchmark harness. It needs external corpora and has **never been run for real**.

**Release blockers** (from the hardening report, all still open): real model hashes/licenses in `models/manifest.json`; the Ed25519 update key; MUSDB18-HQ, Slakh, and private-corpus quality runs plus blind listening; real MPS cold/warm benchmarks against v0.4.2; viewport and accessibility review; LAN certificate trust on macOS/iOS; Developer ID signing and notarization.

## Branch & archive map

All `archive/2026-11/*` branches are local-only. "vs main" counts are commits ahead/behind.

| Branch | What it is | State | Verdict |
|---|---|---|---|
| `main` | v0.4.3 source + GPL | CI green (torch-coupled audit exceptions expire 2027-02-05) | Canonical. |
| `origin/v0.5.0` (= `origin/stemsplat-iree`, `ca51b7c`, Apr 2026) | Editor/trim timeline and waveform caching on top of 0.4.2 | Its editor/waveform functions are already in `main` via the UI rebuild | Absorbed. Safe to delete on GitHub once you're sure. |
| `origin/codex/gauntlet-hardening-…-075057` (Jul 2026) | One commit: upload magic-byte checks, task caps, subprocess/ffprobe timeouts | All of its new functions exist in `main` | Absorbed. Safe to delete. |
| `origin/codex/skytools-api-integration` (Jul 2026, +3/−4) | Owner-scoped `/api/internal/skytools` processing API, plus "negative" outputs (song minus guitar/drums/bass/other) | Not in `main`. Small: ~430 lines in `main.py` and tests | **Salvage candidate.** Negative stems are a real user feature. Rebase onto `main`. Keep the Skytools API only if notbad.tools still needs it. |
| `origin/fix/v0.4.3-qa-hardening` | PR #51 source | Merged | Delete. |
| `archive/2026-11/ui-ux-rebuild-wip` (+15/−4) | `codex/ui-ux-rebuild-20260810` plus uncommitted work: `url_import.py` (URL import, ~1.2k lines), "without" mode tab, Model Manager dialog, trim editor, `docs/ui-parity.md`, about 12 new test files, `uv.lock` | v0.4.3 took "selected paired" pieces from `2d7716e`. The uncommitted layer, including URL import, is **not** in `main` | **Salvage piecemeal.** URL import and the extra tests are the most valuable parts. Port them in small PRs. Don't merge wholesale: its `main.py`/`index.html` diverge heavily. |
| `archive/2026-11/desktop-iree-wip` (branch `stemsplat-iree/v0.5.0` + 139 uncommitted files, 224 files changed) | **The real "0.5.0" product line.** 11 optional 0.5.0 models (Leap Xe, Mega 53, Apollo, Bandit, DrumSep 5, karaoke average, SAM-Audio hook), `stemsplat_runtime/` (backend registry, quality lanes, timing), IREE/compiler-lab tools, perf work (`mask_scratch`, rotary cache, warm model manager), experimental Qwen workflow parser (`stemsplat_workflow.py`, `packaging/workflow-model/`), window chrome/motion polish, cuda_server | Builds and was installed (`/Applications/Stemsplat.app` = 0.5.0). Has turbo/balanced lanes and env flags. Notes: `STEMSPLAT_IREE.md`, `benchmarks/SPEEDUP_REPORT.md`, `benchmarks/MODEL_ADDITIONS_050.md`, `.planning/*` | **Most valuable archive.** It's the source of the app you actually use. Plan a deliberate port onto `main` (models → runtime/perf → UI polish), each step behind flags and checked with `quality.py`. |
| `archive/2026-11/desktop-sprint-stft-cache` | May-30 perf sprint child A: `STEMSPLAT_STFT_CACHE_FIX` | Real 8s run: `match`, no meaningful speedup (STFT/ISTFT are ~5 ms out of ~6 s) | Superseded by iree-wip. Keep the report, then delete. |
| `archive/2026-11/desktop-sprint-band-cache` | Child B: `STEMSPLAT_BAND_CACHE` | Report says opt-in diagnostic only, no speedup claim | Delete after you've read the report. |
| `archive/2026-11/desktop-sprint-mask-scatter` | Child C: `mask_scratch` backend | 8s real: 1.12–1.32x, quality `match`, but noisy and ~2.2x peak MPS memory | Already in iree-wip as `STEMSPLAT_MASK_SCRATCH`. Delete. |
| `archive/2026-11/desktop-sprint-hotspot-cleanup` | Child D: hotspot-probe reporting | Report says "merge-ready", measurement only | Probably in iree-wip. Delete after a diff check. |
| `archive/2026-11/desktop-sprint-warmrun-bench` | Child E: `tools/benchmark_warmrun_roformer.py` | Cold 19.4 s vs warm 6.9 s (2.8x), all `match` | Tool already in iree-wip. Delete. |
| `archive/2026-11/desktop-stash-0` (Mar 2026) | GitHub Desktop stash: bootstrap/installer + output-format work on a Jan-2026 codex branch | Superseded by v0.3–0.4. **Contains a full copyrighted WAV and app logs** | Delete. Never push. |
| `archive/2026-11/desktop-stash-1..4` (Jan 2026) | Only logs and icon files | Nothing useful | Delete. |
| `archive/2026-11/home-rust-workflow-lab` | The old `~/stemsplat` checkout. **Unrelated history** (2025-07 skeleton, no common ancestor). Contains: Rust Axum server + 10 crates, `stemrunner/` Python CLI, a dependency-free web rewrite, the whole workflow-model campaign (`training/workflow/`, `benchmarks/`, `tools/benchmark_*`), notes (`progress.md`, `task_plan.md`, `findings.md`, `QA_AUDIT_REPORT.md`, `why_v2_is_going_to_beat_qwen.md`), and the Fable UI prompt | The Rust skeleton never had real inference (`QA_AUDIT_REPORT.md`: "not release-ready as a working stem separator"). Its applicable fixes were already ported to v0.4.3 (`docs/qa/legacy-skeleton-audit.md`) | **Archive only.** Never merge. Keep it as the record of the workflow-model research. A prototype build of its `stemsplat-server` is still running from `~/Library/Application Support/stemsplat-engineering/releases/20260908/`. |

Inspect a stash branch with `git show --stat <branch>` and `git diff <branch>^1 <branch>`.

## `~/stemsplat-archive/` (~39 GB, moved, not deleted)

| Path | Size | Contents | Safe to delete? |
|---|---|---|---|
| `archive-branches.bundle` | 82 MB | Git bundle of all 13 `archive/2026-11/*` branches | **Keep.** It's your off-repo backup. Copy it somewhere else too. |
| `desktop-clone/` | 31 GB | Former `~/Desktop/Projects/Repositories/stemsplat`: `build/` 23 GB (model-additions-050 8.9 GB, install-backups 7.7 GB, ui/window/motion build dirs, Qwen HF base 1.4 GB, LoRA MLX adapter), `models/` 3.4 GB (5 checkpoints), `venv` + `.venv` 2.1 GB, `dist/` 1.3 GB, `.toolchains/` 1 GB, `.git` | `venv`, `.venv`, `.toolchains`, `dist`, `build/install-backups`, `build/package-backups`, and the `build/*-2026*` UI build dirs are rebuildable. Delete them now. Keep `models/` until you've re-downloaded and hash-pinned the checkpoints. Keep `build/model-additions-050` and the Qwen dirs until the 0.5.0 port is decided. |
| `home-checkout/` | 2.7 GB | `build/` (GLiClass venv/model, V9–V11 MLX/PEFT adapters, finetune outputs), `release-downloads/Stemsplat-0.4.2.zip`, uploads, data, dist, actualweb `node_modules`, ignored training data | `node_modules`, `dist`, `build/gliclass-edge-venv`, and the duplicate `v10-mlx-base*` can go. The 0.4.2 zip is also on GitHub Releases. The rest is rejected-model evidence: delete when you no longer care to reproduce it. |
| `desktop-worktrees/` | 1.6 GB | The 5 sprint worktrees (their `.git` pointers are now broken) | Yes. Their code is on the sprint branches. |
| `ui-rebuild-worktree/` | 1.4 GB | `dist/`, `build/` from the UI rebuild | Yes (code is on `ui-ux-rebuild-wip`). |
| `v043-hardening-worktree/` | 2.1 GB | `.venv`, `dist/` (dev 0.4.3 app), `build/`, `node_modules` | Yes, once you've rebuilt 0.4.3 from `main`. |

Disk you didn't clean up: `/Applications` holds 8 Stemsplat bundles (~4 GB of `*-backup-*` and `.pre-polish` copies). `~/Library/Application Support/stemsplat` is 44 GB (models, outputs, workflow models). `stemsplat-release-backups` is 679 MB.

## Text-to-workflow model campaign (Sep 2026), summary

Goal: turn a plain-English request ("vocals and bass, boost the mix 3 dB, trim the first 10 s") into a closed, validated workflow JSON. Unsafe or ambiguous requests had to come back as `clarify`. **Promotion gates:** ≥95% supported-exact, 100% unsafe rejection, warm p95 <500 ms, measured on a sealed frozen corpus of 3,200 cases (2,240 supported, 960 unsafe).

| Line | Best result | Why not promoted |
|---|---|---|
| Qwen3-0.6B LoRA V3/V5 (shipped as the opt-in experimental parser in the 0.5.0 build) | V5 installed: 30.8% exact / 99.3% reject / 462 ms p95 | Accuracy far below gate. |
| Qwen LoRA V8–V19 (SFT, DPO V15/V16, accuracy-first V17, loudness defaults V18, rich graph V19), trained remotely on an RTX 3060 Ti | V8 87.7% / 90.6% / 2.9 s. V18 88.4% / 78.6% / 2.8 s. V10/V11 overfit or over-clarified on dev | Plateaued near 88%. Unsafe rejection got worse, and latency was 6x the gate. |
| Auto-categorizer V20–V25 (safety gate, typed retrieval, gain fast path; V22 tried Qwen3-4B) | V25 frozen: 88.35% / 98.39%; V18 fallback p95 2.8 s | Still below 95/100, and the fallback is slow. |
| V26–V28 deterministic full-schema graph parser | V28: 6/6 owner prompts (dev), but the one-time owner holdout scored 1/9 (V26) and 2/6 (V27) | No independent frozen score. Deployed only as a **review beta**: `stemsplat-v28-review.service` on the server (loopback :9901), behind notbad.tools preview. |
| Laya encoder V1–V5 | V5 passed dev (96.1/100) and smoke (100/100), then **failed frozen at 44.7% / 90.5%** | Didn't generalize. V2 ("going to beat Qwen") scored 13%. |
| GLiClass Edge hybrid | 40.5% / 67.5% / 27 ms | Fast but wrong. |

Why nothing shipped: no candidate cleared accuracy and safety and latency at the same time on unseen data. Small models were fast but generalized poorly. Qwen was accurate-ish but slow and leaky on safety. The deterministic parser overfit to known phrasing. Full notes are in `progress.md`, `findings.md`, and `benchmarks/*.md` on `archive/2026-11/home-rust-workflow-lab`, with training code in `training/workflow/`.

## Known gaps / open issues

1. Audit exceptions for torch 2.11 / setuptools / filelock expire 2027-02-05. Clearing them needs torch>=2.13 and/or Python 3.11+, run through the quality gates.
2. v0.4.3 is unreleased. Every blocker in its hardening report is still open, and `quality.py` has never run on real audio.
3. The app you actually run (0.5.0) has no committed source. It exists only as `archive/2026-11/desktop-iree-wip`.
4. Negative stems, URL import, and the 0.5.0 optional models are absent from `main`.
5. `main` implements only the `reference_safe` lane. Turbo, adaptive overlap, mask_scratch, and IREE exist only in iree-wip, where turbo candidates failed their drift gates.
6. Possibly still running: the V28 review service, the platform-api beta routing on the server, and the `stemsplat-engineering` Rust prototype on this Mac.
7. Untracked leftovers in this checkout: an empty `rust/uploads/`, `__pycache__/`, and a Python 3.13 `.venv` (CI and the locks target 3.10).

## When you come back: checklist (in priority order)

1. **Get to green.** Create a fresh venv (`python3.10 -m venv .venv && .venv/bin/pip install --require-hashes -r requirements-dev-macos-arm64.lock`), then `npm ci`. Run `STEMSPLAT_DISABLE_BACKGROUND_THREADS=1 .venv/bin/python -m unittest discover -s tests -p 'test_*.py'`, `npm run test:e2e`, and `python tools/pip_audit_gate.py --requirements requirements-macos-arm64.lock`. Before 2027-02-05, plan the quality-gated torch/Python upgrade that clears the remaining audit exceptions.
2. **Decide the product line.** Either (a) ship 0.4.3 by working through the release blockers, or (b) port 0.5.0 from `desktop-iree-wip` onto `main` in stages. Probably (b) first for models, because that's what you already use.
3. **Salvage the small wins.** Rebase negative stems from `origin/codex/skytools-api-integration`. Port `url_import.py` and its tests from `ui-ux-rebuild-wip`.
4. **Run `quality.py` once for real** (`quality/README.md`) to get a v0.4.2 baseline on this Mac. Every later perf or model change needs it (see `AGENTS.md`).
5. **Tidy GitHub.** Delete the merged or absorbed remote branches: `fix/v0.4.3-qa-hardening`, `codex/gauntlet-hardening-…`, and `v0.5.0`/`stemsplat-iree` once you confirm nothing else is needed (`git log main..origin/v0.5.0`, then grep the functions).
6. **Prune local archive branches.** Delete `desktop-stash-*` and the 5 `desktop-sprint-*` branches (the bundle keeps copies). Keep `home-rust-workflow-lab`, `desktop-iree-wip`, and `ui-ux-rebuild-wip` until they're ported.
7. **Reclaim disk.** Remove the rebuildable dirs listed in the archive table and the old `/Applications/Stemsplat-*backup*` bundles. Check whether the V28 server service and the engineering prototype should be stopped.
8. Only then consider workflow-model work again. It needs a new idea and a freshly sealed holdout, not V29.
