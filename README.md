# 🎨 CapStudio — Free Local AI Caption Studio

CapStudio is a professional, **100% offline, free, and local-first AI caption editor** for video creators. Built with Flutter, Riverpod, and Isar DB, it empowers creators to generate high-fidelity, word-level animated captions, mix immersive sound effects, and burn in gorgeous subtitle templates without subscription fees or data leaving the host device.

---

## 🚀 Key Architectural Advantages

* **Local-First speech-to-text**: Orchestrates local process executions of `whisper.cpp` and `FFmpeg` to transcribe video audio directly on your processor.
* **Intelligent Auto-Save**: Features a silent, debounced 3-second auto-save pipeline using transactional Isar DB records.
* **Resilient Asset Mirroring**: Implements dynamic download failovers inside `PackDownloadService`. If primary asset paths go offline, the client seamlessly cycles through organizational backup mirrors on-the-fly.
* **Zero CDN Dependency**: All 48 designer and Noto multi-language fonts are fully bundled inside the binary asset package to ensure complete offline independence.
* **Windows VC++ Guarding**: Low-level runtime checks query system DLL folders (`System32`/`SysWOW64`) on Windows to warn creators and link directly to official Microsoft Visual C++ redistributable packages if missing.
* **Polyphonic Wave Synthesis**: Dynamically synthesizes all 38 classic editor sound effects entirely in pure Dart PCM bytes at runtime, bypassing external download needs.

---

## 💻 Platform Support & Capabilities

| Platform | Codebase State | Speech-to-Text (Whisper) | Audio/Video processing (FFmpeg) | Local DB (Isar) | Subtitle Burn-In & Export |
| :--- | :---: | :---: | :---: | :---: | :---: |
| **Windows x64 / Win32** |  Ready |  Auto-Downloads (AVX & no-AVX fallback) |  Auto-Downloads |  Local Embedded |  FFmpeg Filters |
| **macOS (Apple Silicon & Intel)** |  Ready | ⚠️ Manual Brew linking |  Auto-Downloads |  Local Embedded |  FFmpeg Filters |
| **Linux (Ubuntu/Fedora/Arch)** |  Ready | ⚠️ Manual Package Build |  Static Auto-Downloads |  Local Embedded |  FFmpeg Filters |
| **Google Android** | 🚧 Mobile Shell | ❌ Sandboxed (NDK .so pending) | ❌ Executable Blocked | 🚧 Local Embedded | ❌ Executable Blocked |
| **Apple iOS** | 🚧 Mobile Shell | ❌ Sandboxed (Framework pending) | ❌ Executable Blocked | 🚧 Local Embedded | ❌ Executable Blocked |
| **Web Browser** |  Graceful Bypasses | ❌ Sandboxed (JS constraints) | ❌ Executable Blocked | ❌ Sandboxed | ❌ WebNotSupported landing page |

---

## 🛠️ Step-by-Step Installation Guides

### 🟦 Windows
1. CapStudio will automatically identify your CPU capabilities on startup.
2. If your CPU supports **AVX**, it will fetch the standard optimized `whisper-cli.exe`. If not, it falls back to the compatible **Win32 non-AVX** build.
3. **VC++ Dependency Warning**: If your Windows OS is missing the Microsoft Visual C++ Redistributable, an amber warning banner will show in Settings with a direct link to download the official [Microsoft runtime installer](https://aka.ms/vs/17/release/vc_redist.x64.exe).

### 🟥 macOS (Apple Silicon & Intel)
Due to strict sandboxing on macOS, manual linking is required for local CLI triggers:
1. Open your terminal and install Homebrew (if missing):
   ```bash
   /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
   ```
2. Install the `whisper-cpp` package:
   ```bash
   brew install whisper-cpp
   ```
3. Set your Whisper path inside Settings to:
   * **Apple Silicon (M1/M2/M3)**: `/opt/homebrew/bin/whisper-cpp`
   * **Intel Macs**: `/usr/local/bin/whisper-cpp`

### 🟩 Linux (Ubuntu/Debian, Fedora, Arch)
Compile `whisper.cpp` locally to ensure full compatibility with your Linux distribution:
* **Ubuntu/Debian**:
  ```bash
  sudo apt update && sudo apt install -y git build-essential
  git clone https://github.com/ggerganov/whisper.cpp.git && cd whisper.cpp && make
  cp main ~/.config/CapStudio/bin/whisper-cli
  ```
* **Fedora**:
  ```bash
  sudo dnf install -y git make gcc-c++ sdl2-devel
  git clone https://github.com/ggerganov/whisper.cpp.git && cd whisper.cpp && make
  cp main ~/.config/CapStudio/bin/whisper-cli
  ```
* **Arch Linux**:
  ```bash
  sudo pacman -S whisper-cpp
  ln -s /usr/bin/whisper-cpp ~/.config/CapStudio/bin/whisper-cli
  ```

---

## 🧠 Whisper Model Matrix Comparison

CapStudio integrates direct HuggingFace downloading from Settings or directly in-context inside the editor panel:

| Model ID | Logical Size | English-Only option | Speed Multiplier | Transcription Accuracy | Recommended CPU |
| :--- | :---: | :---: | :---: | :---: | :---: |
| **`tiny`** | ~75 MB | `tiny.en` | **10.0x (Instant)** | 🟦⬜⬜⬜⬜ (Low) | Older Dual-Core Laptop |
| **`base`** (Default) | ~140 MB | `base.en` | **7.0x (Very Fast)** | 🟦🟦⬜⬜⬜ (Moderate) | Intel i3 / Ryzen 3 |
| **`small`** | ~460 MB | `small.en` | **4.0x (Balanced)** | 🟦🟦🟦⬜⬜ (Good) | Intel i5 / Ryzen 5 |
| **`medium`** | ~1.5 GB | `medium.en` | **1.5x (Heavier)** | 🟦🟦🟦🟦⬜ (Excellent) | Intel i7 / Ryzen 7 |
| **`large-v3`** | ~2.9 GB | *Multilingual* | **0.5x (Slowest)** | 🟦🟦🟦🟦🟦 (Maximum) | High-end Multi-core CPU |

---

## 🔬 Local Verification & QA Commands

Before pushing commits, run standard code sanity checks:

### 1. Static Analysis
Run the Flutter analyzer to confirm code warning compliance:
```bash
flutter analyze
```
*Expected Output:* `No issues found!`

### 2. Automated Test Suite
Execute the unit and widget test runner:
```bash
flutter test
```
*Expected Output:* `All tests passed!` (including isolated, warning-free EmojiService mock scans and synthesized PCM checks).

---

## 📁 Key File Structure Map

```text
lib/
├── app/
│   ├── routes.dart          # go_router configuration & route guards
│   ├── theme.dart           # Ambient glowing painter & style systems
│   └── theme_provider.dart  # Riverpod notifier for global theme settings
├── core/
│   ├── assets/              # Downstream pack downloader & mirror failover systems
│   ├── audio/               # Polyphonic WAV synthesizer & sound generators
│   ├── database/            # Isar local database initialization & transaction schemas
│   ├── downloader/          # Binary CPU-AVX download pipeline
│   ├── logger/              # rotation file logging & memory ring-buffer
│   ├── settings/            # SharedPreferences configuration maps
│   ├── utils/               # AppDirs directories & CPU capability detection
│   └── whisper/             # whisper.cpp transcription parser CLI wrapper
└── features/
    ├── dashboard/           # User dashboard, recent project card listings
    ├── editor/              # Scrubber timeline, style panels, caption overlays
    ├── exporter/            # Advanced sub-station alpha and video burn-in exporters
    ├── onboarding/          # Interactive setup wizards
    └── settings/            # Configuration editors & models picker widgets
```

---

## 📜 Credits & Licenses

CapStudio is built using various open-source fonts, emojis, and third-party software libraries (including Google Fonts, Noto Sans CJK Fonts, OpenMoji, Microsoft Fluent Emojis, whisper.cpp, and FFmpeg).

For a complete list of attributions and their corresponding licenses, please refer to [CREDITS.md](file:///a:/Projects/Local%20AI%20Caption%20Studio/CapStudio/CREDITS.md).

