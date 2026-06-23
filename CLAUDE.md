# OpenRGBdoodle

C++ Qt app (QMake). Controls RGB hardware via reverse-engineered protocols. Plugin API v4.

## Quick Reference

| What | Where |
|------|-------|
| Device controllers | `Controllers/<Brand>/` (~164) |
| Qt UI | `qt/` |
| Plugin interface | `OpenRGBPluginInterface.h` |
| Network SDK | `NetworkServer.cpp`, `NetworkClient.cpp` |
| Resource/device mgmt | `ResourceManager.cpp` |
| CLI | `cli.cpp` |
| Build file | `OpenRGB.pro` |

## Build

```bash
# Linux
qmake OpenRGB.pro && make -j$(nproc)

# Windows (MSVC)
qmake OpenRGB.pro && nmake

# Deps: Qt5/6, libusb-1.0, hidapi, libudev, libmbedtls
# pip install openrgb-python         # Python SDK
# pip install pyopenrgb              # alt SDK
# pip install openrgb-client         # CLI client
```

## Local Drive Permissions

**Z:\ drive has full read/write access.** No prompts required for any operation under `Z:\`.

Windows paths map as:
- `Z:\` → `/mnt/z/` (WSL) or `Z:/` (native Windows)

## Token Efficiency Rules

1. **Always grep/glob before reading** — never read a file to find a symbol
2. **Parallel tool calls** — batch all independent reads/searches in one message
3. **Reference by path:line** — never describe code location in prose
4. **No comments** unless the WHY is non-obvious (hidden constraint, workaround, subtle invariant)
5. **No docstrings** — well-named identifiers document themselves
6. **Skip generic answers** — search GitHub/pip for a real tool before explaining concepts
7. **Compact CLI output** — use `--quiet`, `--porcelain`, `-q` flags where available
8. **Prefer Edit over Write** — Edit sends only the diff

## Recommended Tools & Plugins

### OpenRGB Python Ecosystem (pip)
```bash
pip install openrgb-python    # Primary SDK — RPC to OpenRGB server
pip install pyserial          # Serial device access
pip install hidapi            # Direct HID access
pip install libusb1           # Direct USB
pip install construct         # Binary protocol parsing
```

### OpenRGB Plugins to Check
- `OpenRGB-Effects-Plugin` — visual effects engine
- `OpenRGB-Scheduler-Plugin` — timed profiles
- `OpenRGB-Ambient-Plugin` — screen ambient lighting
- `OpenRGB-E1.31-Plugin` — DMX/sACN protocol bridge
- `OpenRGB-GoodNight-Plugin` — auto-dim at night

Search: `https://gitlab.com/OpenRGBDevelopers`

### MCP Servers (add to `.claude/settings.json`)
- `github` — repo search, issue tracking, PR management
- `Google_Drive` — sync profiles/configs to cloud

### VS Code Extensions
- `ms-vscode.cmake-tools` — CMake integration
- `ms-vscode.cpptools` — IntelliSense for C++
- `twxs.cmake` — CMake syntax

## 3DS Max Integration

**Daily scan target:** GitHub repos matching `3dsmax rgb` | `maxscript lighting` | `3dsmax openrgb`

Known integration paths:
- MaxScript → OpenRGB HTTP API (port 6800 by default)
- Python in Max (3DS Max 2022+): `import openrgb_python`
- MAXScript socket to OpenRGB SDK
- Scan results stored in `~/.claude/github_scan.md`

```maxscript
-- Trigger OpenRGB profile via HTTP
python.Execute "import urllib.request; urllib.request.urlopen('http://localhost:6800/profile/load?name=rendering')"
```

## Unreal Engine Integration

**Daily scan target:** GitHub repos matching `unreal openrgb` | `ue5 rgb plugin` | `unreal lighting peripheral`

Known integration paths:
- UE Python API 5.x → openrgb-python SDK
- Blueprint → HTTP request to OpenRGB server
- `FGenericPlatformProcess::CreateProc` to invoke openrgb CLI
- DMX/E1.31 via OpenRGB E1.31 plugin + UE DMX plugin

```python
# UE Python → OpenRGB
import unreal, subprocess
subprocess.Popen(["openrgb", "--profile", "gameplay"])
```

UE plugins to watch:
- `DMX Engine` (built-in) — route to OpenRGB E1.31 plugin
- `ChromaSDK` — Razer Chroma, bridges some OpenRGB devices

## Daily GitHub Scan

Automated by `SessionStart` hook in `.claude/settings.json`. Runs at most once per 24h.
Results written to `~/.claude/github_scan.md`.

Manual trigger:
```bash
rm ~/.claude/github_scan.timestamp && claude  # restart session
```

Search queries executed:
1. `openrgb plugin updated:>YESTERDAY` — new/updated OpenRGB plugins
2. `3dsmax rgb OR 3ds-max lighting plugin` — 3DS Max tools
3. `unreal openrgb OR ue5 rgb peripheral` — Unreal integrations
4. `openrgb python updated:>YESTERDAY` — Python ecosystem updates

## Controller Development Pattern

Each controller lives in `Controllers/<Brand>/<Brand>Controller.{h,cpp}` + `<Brand>ControllerDetect.cpp`.

```
Controllers/
  BrandName/
    BrandNameController.h         # class : public RGBController
    BrandNameController.cpp
    BrandNameControllerDetect.cpp # registers with ResourceManager
```

Minimal new controller checklist:
- [ ] Subclass `RGBController`
- [ ] Implement `SetupZones()`, `ResizeZone()`, `DeviceUpdateLEDs()`, `UpdateZoneLEDs()`, `UpdateSingleLED()`, `SetCustomMode()`
- [ ] Register in `Detector.h` and `OpenRGB.pro`
- [ ] Add PID/VID to `DeviceDetector.h`

## Commit Convention

Follow existing style: `Controllers: BrandName: Description of change`
Linear history — rebase, never merge.
