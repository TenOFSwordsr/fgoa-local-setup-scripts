# FGO Arcade Local Platform Toolkit

A working folder for getting Fate/Grand Order Arcade running offline on Windows: PowerShell scripts
that fetch and unpack the Cloud23333 local-platform package, and the two community patches applied
on top of it - **FGOAC scooby** (English game text, artwork and launcher) and an **AMD OpenGL
shim**. Only the three root scripts are original; everything below them is downloaded content or
its extracted payload, kept here as the staging area for an install.

**Suggested repo name:** `fgoa-local-setup-scripts`
**Stack:** PowerShell 5.1 (`System.Net.HttpWebRequest`, `Get-FileHash`, 7-Zip via `7z.exe`); payloads include a bundled Python local server and .NET/native binaries
**Status:** active
**Last modified:** 2026-09-16

## What it does

- `download_base_game.ps1` - resilient polling downloader for the five `FGOA_Cloud23333.partN.rar`
  archives from Google Drive (file IDs inline). Resumes with HTTP 206 Range, distinguishes a real
  stream from Google's quota HTML page by `Content-Type` and size, retries up to `MaxLoops` with a
  `PollIntervalMinutes` backoff, and chains into `extract_game.ps1` once all parts are complete.
- `extract_game.ps1` - locates `7z.exe`, validates that `part1.rar` is a real archive rather than a
  truncated HTML response, unpacks the ~30 GB set into `FGO_Arcade`, and integrates the scooby
  payload from `scooby\`.
- `download_patches.ps1` - generic retrying downloader with SHA-256 verification and
  skip-if-hash-matches, used to pull patch and compat-layer releases.
- `scooby/` - FGOAC scooby v1.1.2 as shipped: `Apply-EN-Patch.ps1` (`-InstallRoot`, `-Rollback`),
  `manifest.json` with a per-file SHA-256 inventory, `payload/App/` (the English launcher
  `FGO_Launcher.ps1`, `FGO_EnvironmentCheck.ps1`, `FGO_StartupChecks.ps1`), `payload/Server/`
  (`Start-FGOLocalServer.ps1`, `Stop-FGOLocalServerWhenIdle.ps1`, `tools/fgo_account.py` for local
  Aime account create/grant/reset, `artemis/titles/`), `compat/` (`fgoglcompat.dll` plus two AMD
  `opengl32.dll` shim builds), and the player guides.
- `amd_shim/` - a separate AMD patch release: `FgoAmdPatch.exe`, `game-patch/install.ps1` and
  `gui-action.ps1`, its own `opengl32.dll` + `amdcfg`, `release.json` build metadata and README.
- `FGOA_Cloud23333/V1.00`, `FGOA_Cloud23333/汉化工具` - empty staging targets for downloads.

## Layout

```
download_base_game.ps1   Google Drive multi-part downloader with quota detection
extract_game.ps1         7-Zip extraction and scooby integration
download_patches.ps1     retry + SHA-256 verified fetcher
scooby/                  English patch package: launcher, local server, compat DLLs, guides
amd_shim/                AMD OpenGL patch installer and release metadata
FGOA_Cloud23333/         download staging (currently empty)
```

## Running it

```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File .\download_base_game.ps1
powershell -NoProfile -ExecutionPolicy Bypass -File .\download_patches.ps1
```

Game installs must not sit on drive `E:` or `Y:` (the platform's file hook redirects those to the
cabinet mount and the game fails with ERROR 4104), and the game requires administrator rights.

## Notes

- Not publishable as a repository. It mixes your own three scripts with third-party binaries, a
  fan translation of copyrighted game content, and - in `download_base_game.ps1` - the Google Drive
  file IDs of a redistributed arcade client, while `extract_game.ps1` carries the archive password
  for it. If you want this on GitHub, publish only the three root scripts after removing the
  embedded IDs and the password; the `scooby/`, `amd_shim/` and `FGOA_Cloud23333/` trees should
  never be committed.
- All three root scripts hardcode `C:\Users\Administrator\Downloads\fgoa\...` paths as defaults.
- `scooby/README.md`, `scooby/GUIDE_EN.md`, `scooby/CHANGELOG.md` and `amd_shim/README.en.md` are
  upstream-written and are the real documentation for behaviour, hardware support and error codes.
