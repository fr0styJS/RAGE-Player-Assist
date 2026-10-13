# RAGE Player Assist

A Windows desktop application that monitors GTA RP (RageMP) in-game chat logs and triggers configurable sound alerts based on detection rules (private messages, name mentions, custom regex patterns).

## Tech Stack

- **Language:** Python 3.13+
- **GUI:** PySide6 (Qt6) - primary UI in `ui/qt_gui.py`; legacy Tkinter UI in `ui/gui.py`
- **Audio:** pygame with Windows MCI fallback
- **Packaging:** PyInstaller for standalone Windows executables
- **CI:** GitHub Actions (`release.yml`) builds and uploads release zips

## Project Structure

```
main.py                  # Entry point: CLI (--console) or GUI
config/app_config.py     # Dataclass-based config, JSON persistence
detections/linehandler.py # Detection engine, rule matching, sound playback
filehandler/readstorage.py # .storage file watcher (polling generator)
ui/qt_gui.py             # Primary PySide6 GUI
sounds/                  # Bundled .wav alert sounds
build_exe.ps1            # Local build script
rage_player_assist.spec  # PyInstaller spec
```

## Key Patterns

- **Config:** `AppConfig` dataclass saved to `app_config.json` via `load_config()`/`save_config()`. Config auto-saves on load to fill defaults.
- **Detection rules:** Three types: `contains` (substring), `mention` (word-boundary name match excluding speaker), `regex` (with flags). Each has cooldown, volume, category.
- **File watching:** `watch_chat()` is a polling generator (0.5s) that reads RageMP's `.storage` JSON file and yields new chat lines via overlap diffing.
- **Threading:** Watcher runs in a `QThread` (Qt GUI) or daemon `Thread` (Tkinter/console). Uses `threading.Event` for clean shutdown.
- **Audio fallback:** Tries pygame first, falls back to Windows MCI (`ctypes.windll.winmm`).
- **Frozen detection:** Uses `sys.frozen` / `sys._MEIPASS` to resolve resource paths in PyInstaller builds.

## Build & Run

```bash
# Development
python main.py              # Launch Qt GUI
python main.py --console    # Console mode
python main.py --debug      # Verbose logging

# Build executable
pip install -r requirements.txt
powershell ./build_exe.ps1
```

## Version

- `SOURCE_APP_VERSION` / `APP_VERSION` are defined in `config/app_config.py` (currently "1.1.0"). `APP_VERSION` comes from `build_metadata.json` when a tagged build has run.
- `APP_NAME` ("RAGE Player Assist") is the on-disk/binary identity used by the exe, installer and `%LOCALAPPDATA%` folder; `APP_DISPLAY_NAME` ("RAGE + FiveM Player Assist") is what the UI shows. Keep them separate — renaming `APP_NAME` breaks the portable updater and installed paths.

## Releases

- Releases are published on the fork: https://github.com/fr0styJS/RAGE-Player-Assist/releases (the upstream `SebzIO` repo is read-only for this account).
- Local flow: `$env:RPA_RELEASE_TAG="vX.Y.Z"; powershell -File .\build_exe.ps1` (the tag must equal `SOURCE_APP_VERSION`), zip `dist\RAGE Player Assist\{RAGE Player Assist.exe,_internal}`, then `gh release create`/`gh release upload` with the standalone exe, the portable zip and notes.
- `GITHUB_OWNER`/`GITHUB_REPO` in `config/app_config.py` drive both the in-app update check and the Help → Release Notes dialog.

## Notes

- Windows-only (uses `ctypes.windll`, `os.startfile`)
- No test suite exists
