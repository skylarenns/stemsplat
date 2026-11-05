# stemsplat

**Mac-only stem splitter for Apple Silicon.**  
The best and easiest to use app for stem splitting.

<img width="1430" height="892" alt="Screenshot 2026-04-09 at 9 35 16 AM" src="https://github.com/user-attachments/assets/20aac5ad-c93f-4ab5-b3e3-78d664b7214d" />

## Quickstart

1. Download the latest `.zip` from the [Releases page](https://github.com/Skytheredhead/stemsplat/releases) (the newest published release is v0.4.2; `main` is v0.4.3)
2. Extract it
3. Open the `.app`

## Getting started from source

Source builds target Apple Silicon macOS. CI uses Python 3.10 (the hashed `*-macos-arm64.lock` files are compiled for 3.10; `pyproject.toml` allows 3.10–3.13) and Node 22. FFmpeg must be on `PATH` (`brew install ffmpeg`); uploads are probed with `ffprobe`.

```bash
# install (dev lock includes the runtime lock plus test/build tooling such as PyInstaller)
python3.10 -m venv venv
source venv/bin/activate
python -m pip install --require-hashes -r requirements-dev-macos-arm64.lock
npm ci --ignore-scripts

# run the desktop app (serves http://127.0.0.1:9876, or the next free port)
scripts/stemsplat-launch          # or: python launcher.py [--no-browser]

# run tests
STEMSPLAT_DISABLE_BACKGROUND_THREADS=1 python -m unittest discover -s tests -p 'test_*.py'
npm run lint
npx playwright install chromium && npm run test:e2e   # starts launcher.py with python3 from PATH

# build the macOS app (development-only, ad-hoc signed) -> dist/Stemsplat.app
./build_app.sh
```

`python tools/run_test_battery.py` runs the broader local battery (add `--include-build` to finish with `build_app.sh`). Models are not downloaded automatically in v0.4.3; see the hardening status below.

## What It Does

- Runs fully locally on your Mac
- No cloud uploads
- Batch queue processing
- Optional passcode-protected HTTPS LAN/mobile access (off by default)
- Multiple export formats
- Previous-files history so recent outputs are easy to reopen or reuse

## Why This Exists

Most online stem splitters either suck or are behind paywalls, and UVR was a pain to do anything.
I made Stemsplat to fill in the gaps UVR left. 

email stemsplat@gmail.com for bugs/feature requests

## Stems

- Vocals
- Instrumental
- Vocals + instrumental (select both)
- Drums
- Bass
- Guitar
- Other
- Background vocals
- Full mix
- Full mix faster
- Drum split - 4
- Drum split - 6
- All stems
- Boost harmonies
- Denoise

## Requirements

### For the app
- Apple Silicon Mac (M-series)
- macOS
- Enough free disk space for models and exports (a couple gigs)

### For running or building from source
- Python 3.10–3.13 (3.10 matches CI and the lock files)
- Node 22 and npm (CSS build, lint, Playwright, `build_app.sh`)
- FFmpeg with `ffprobe` on `PATH`

## Model Credits

### Becruily
- Vocals  
  https://huggingface.co/becruily/mel-band-roformer-vocals
- Instrumental  
  https://huggingface.co/becruily/mel-band-roformer-instrumental
- Deux  
  https://huggingface.co/becruily/mel-band-roformer-deux
- Guitar  
  https://huggingface.co/becruily/mel-band-roformer-guitar
- Karaoke / Background vocals  
  https://huggingface.co/becruily/mel-band-roformer-karaoke

### Jarredou
- Denoise  
  https://huggingface.co/jarredou/aufr33_MelBand_Denoise
- BS-Roformer 6-stem  
  https://huggingface.co/jarredou/BS-ROFO-SW-Fixed
- DrumSep 6-stem  
  https://github.com/jarredou/models/releases

### Other sources
- Demucs (htdemucs_ft drums / bass / other, htdemucs_6s)  
  https://github.com/facebookresearch/demucs
- ZFTurbo DrumSep 4-stem  
  https://github.com/ZFTurbo/Music-Source-Separation-Training



## v0.4.3 hardening status

The v0.4.3 lane is `reference_safe`: model selection, overlap, precision, output semantics, and MPS behavior remain unchanged unless the quality gates approve a later promotion.

Automatic model downloads are intentionally blocked until every checkpoint has an immutable revision, exact SHA-256, and documented distribution rights in `models/manifest.json`. Existing unsupported user-provided state-dict checkpoints may be used at the owner’s risk; legacy pickle-based models require an exact approved manifest hash.

A production build additionally requires the full public/private quality report, Developer ID signing/notarization credentials, and signed update metadata. By default `build_app.sh` produces a development-only ad-hoc build; `STEMSPLAT_RELEASE_MODE=production ./build_app.sh` runs the audit and release gates and fails closed without those prerequisites.

See `docs/qa/reconciliation-matrix.md` and `quality/README.md` for the release evidence workflow.

## License

Stemsplat is licensed under the GNU General Public License v3.0 (GPL-3.0). See [`LICENSE`](LICENSE). Model checkpoints come from the third-party projects credited above; see `models/manifest.json` and `packaging/THIRD_PARTY_NOTICES.md`.
