# Free Local AI Caption Studi

> [!IMPORTANT]
> **License & Source Code:** CapStudio is a **proprietary, closed-source personal project**. The codebase is privately owned, but leverages and acknowledges open-source packages, fonts, and tools (detailed in the Credits section below) to deliver its features.

---

## 💻 Platform Support Matrix

| Platform | User Setup Experience | Speech-to-Text (Whisper) | Video Rendering (FFmpeg) | Local DB (Isar) |
| :--- | :--- | :--- | :--- | :--- |
| **Windows x64** | 🟢 **Out-of-the-Box** | Auto-Downloads (AVX2 with generic fallback) | Auto-Downloads (Gyan.dev builds) | Local Embedded (Isar) |
| **Google Android** | 🟢 **Out-of-the-Box** | Pre-Bundled JNI (`libwhisper.so`) | Pre-Bundled Library (`FFmpegKit`) | Local Embedded (Isar) |
| **Apple iOS** | 🟢 **Out-of-the-Box** | Pre-Bundled Framework (`xcframework`) | Pre-Bundled Library (`FFmpegKit`) | Local Embedded (Isar) |
| **Web Browser** | 🟢 **Out-of-the-Box** | WASM/JS (`transformers.js` / `whisper.wasm`) | WASM/JS (`ffmpeg.wasm` client) | SharedPreferences JSON shim |
| **macOS (Intel/M-chips)** | 🟡 **Manual Link Required** | Manual Homebrew linking (`brew install whisper-cpp`) | Auto-Downloads (Evermeet build) | Local Embedded (Isar) |
| **Linux (Ubuntu/Arch/etc.)** | 🟡 **Manual Link Required** | Manual distribution compile & link | Auto-Downloads (Static release) | Local Embedded (Isar) |

### Setup Experience Details

* **🟢 Out-of-the-Box (Zero Manual Configuration)**: 
  * **Windows**: Dynamically detects CPU capabilities on boot, auto-downloads the correct AVX2 or generic fallback `whisper-cli.exe` and `ffmpeg.exe` binaries directly, and configures them.
  * **Android**: Emojis, database (`Isar`), transcription (`libwhisper.so` JNI), and audio/video mixing (`FFmpegKit`) are compiled directly into the application package.
  * **iOS**: Same as Android—Whisper (`xcframework`) and `FFmpegKit` are compiled directly inside the app bundle.
  * **Web**: Uses client-side WebAssembly. Video files are parsed in-browser via standard HTML5 media and Web Audio APIs, transcribing via transformers.js and exporting subtitles directly.
* **🟡 Manual Setup Required (Due to OS Restrictions)**: 
  * **macOS**: Apple's strict application sandboxing prevents the app from downloading and executing raw binary files in user directories. Users must install the tools via Homebrew (`brew install whisper-cpp`) and link the path in Settings.
  * **Linux**: Due to the wide variety of package managers and `libc` versions across distributions, the app cannot host a single precompiled binary. Users compile or install `whisper-cpp` from their distribution's repositories and link it.

---

## 🛠️ Step-by-Step Installation & Setup

### Prerequisites
1. **Git**: Install [Git](https://git-scm.com/) on your machine.
2. **Editor**: Install [Visual Studio Code](https://code.visualstudio.com/) (with Dart & Flutter extensions) or Android Studio.
3. **Flutter SDK**: Download Flutter SDK (stable channel, version `3.22.0` or later / Dart SDK `3.4.0` or later) from [flutter.dev](https://docs.flutter.dev/get-started/install). Add the `bin` directory to your system `PATH`.
4. **whisper.cpp Submodule**: Ensure you initialize native submodules:
   ```bash
   git submodule update --init --recursive
   ```

### Running the App
1. **Fetch Dependencies**:
   ```bash
   flutter pub get
   ```
2. **Compile Isar Database Schemas** (required on fresh setup or after modifying schema files):
   ```bash
   dart run build_runner build --delete-conflicting-outputs
   ```
3. **Run the Application**:
   ```bash
   flutter run
   ```
   *(Select a device or target: `flutter run -d windows`, `flutter run -d android`, etc.)*

---

## 💻 OS-Specific Developer Setup

### 🟦 Windows
* **VS workload**: Install **Visual Studio** (Community Edition is free) and select the **Desktop development with C++** workload.
* **FFmpeg/Whisper binaries**: The app automatically auto-downloads binaries to your `%APPDATA%\CapStudio\bin` folder on boot. If your CPU lacks AVX2, the app automatically selects a generic non-AVX2 fallback.
* **Compile Build**:
  ```bash
  flutter build windows --release
  ```
  Output folder: `build\windows\x64\runner\Release\`
* **Create Installer**: Open **Inno Setup** and compile the installer script:
  ```powershell
  iscc scripts/capstudio_setup.iss
  ```

### 🤖 Android
* **NDK & Java**: Install Android Studio. Ensure your SDK Manager has Android NDK installed. Match your local NDK version in `android/app/build.gradle.kts` (e.g. `ndkVersion = "28.2.13676358"`). Set your `JAVA_HOME` environment variable to Android Studio's bundled JBR JDK (e.g. `C:\Program Files\Android\Android Studio\jbr`).
* **Compiling APK**:
  ```bash
  # Debug APK
  flutter build apk --debug --target-platform android-arm64
  # Release APK
  flutter build apk --release --target-platform android-arm64
  ```

### 🍏 macOS & iOS
* **Prerequisites**: A Mac running macOS with Xcode and CocoaPods (`brew install cocoapods`).
* **Step 1: Compile Native iOS Whisper Library**:
  Generate the Apple `xcframework` by compiling the native `whisper.cpp` C++ target:
  ```bash
  chmod +x scripts/build_whisper_ios.sh
  ./scripts/build_whisper_ios.sh
  ```
  This creates `ios/whisper_xcframework/whisper.xcframework`.
* **Step 2: Install CocoaPods dependencies**:
  ```bash
  cd ios && pod install --repo-update && cd ../macos && pod install --repo-update && cd ..
  ```
* **Step 3: Run / Compile**:
  * **macOS Desktop**: `flutter build macos --release`
  * **iOS Device/Simulator**: `flutter run -d ios` or `flutter build ios --release --no-codesign`

### 🐧 Linux
* **Prerequisites**: Install platform development libraries:
  ```bash
  sudo apt-get update && sudo apt-get install -y clang cmake ninja-build pkg-config libgtk-3-dev liblzma-dev libstdc++-12-dev libmpv-dev mpv
  ```
* **Compile & Package**:
  ```bash
  # Compile Linux release
  flutter build linux --release
  # Create Debian .deb package
  chmod +x scripts/create_deb.sh
  ./scripts/create_deb.sh
  ```

---

## ⚡ Post-Setup Assets Configuration

### 1. In-App Assets & Emojis Folder
1. Open the app and click the **Settings gear** icon.
2. Scroll to the **Storage & Emoji Packs** section.
3. Select the root `assets` folder of your codebase (`[Your-Root-Folder]/CapStudio/assets`).
4. The app will instantly verify the contents and activate your emoji packs!

### 2. Auto-Downloading FFmpeg & Whisper CLI (One-Click Setup)
1. In the **Executables Configuration** section inside Settings:
2. If FFmpeg or Whisper CLI paths are blank or invalid, you will see a glowing **"AUTO DOWNLOAD"** action button.
3. Click **AUTO DOWNLOAD**!
4. The app will automatically detect your OS, stream the correct pre-compiled zip in the background, extract it in a background thread, set execution permissions, and activate them.

---

## 🔒 Security & Integrity Verification (F-0005)

To maintain the safety of downloaded components while preserving 100% offline privacy:
1. **Binary Magic Bytes Audits**: Verifies downloaded file headers to ensure they match native executable signatures (PE `MZ` on Windows, ELF on Linux, Mach-O on macOS).
2. **OS Signature Verification**: On Windows, the app executes a PowerShell `Get-AuthenticodeSignature` check to verify signatures (rejecting on `HashMismatch`). On macOS, `codesign` verification audits certificates.
3. **Dynamic Manifest Sync (F-0017)**: Available packs metadata and checksums are synchronized from a remote manifest. If offline, the app defaults to its pre-bundled local fallback manifest automatically.

---

## 🧠 Whisper Model Matrix Comparison

CapStudio supports downloading and swapping Whisper models directly inside the application Settings:

| Model ID | Logical Size | English-Only option | Speed Multiplier | Transcription Accuracy | Recommended CPU |
| :--- | :---: | :---: | :---: | :---: | :---: |
| **`tiny`** | ~75 MB | `tiny.en` | **10.0x (Instant)** | 🟦⬜⬜⬜⬜ (Low) | Older Dual-Core Laptop |
| **`base`** (Default) | ~140 MB | `base.en` | **7.0x (Very Fast)** | 🟦🟦⬜⬜⬜ (Moderate) | Intel i3 / Ryzen 3 |
| **`small`** | ~460 MB | `small.en` | **4.0x (Balanced)** | 🟦🟦🟦⬜⬜ (Good) | Intel i5 / Ryzen 5 |
| **`medium`** | ~1.5 GB | `medium.en` | **1.5x (Heavier)** | 🟦🟦🟦🟦⬜ (Excellent) | Intel i7 / Ryzen 7 |
| **`large-v3`** | ~2.9 GB | *Multilingual* | **0.5x (Slowest)** | 🟦🟦🟦🟦🟦 (Maximum) | High-end Multi-core CPU |

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
│   ├── emoji/               # Emoji service, rendering schemas & category index mapping
│   ├── ffmpeg/              # Subprocess execution & local process wrapping
│   ├── fonts/               # Custom font asset linking & loaders
│   ├── logger/              # Rotation file logging & memory ring-buffer
│   ├── settings/            # SharedPreferences configuration maps
│   ├── subtitle/            # Subtitle structures, parsing (SRT/VTT/ASS), and importers
│   ├── utils/               # AppDirs directories & CPU capability detection
│   ├── video/               # Video properties scanner and relink helpers
│   └── whisper/             # whisper.cpp transcription parser CLI wrapper
└── features/
    ├── dashboard/           # User dashboard (under dashboard/presentation/)
    ├── editor/              # Scrubber timeline, style panels, caption overlays (under editor/presentation/)
    ├── exporter/            # Sub-station alpha and video burn-in exporters (under exporter/presentation/)
    ├── onboarding/          # Interactive setup wizards (under onboarding/presentation/)
    └── settings/            # Configuration editors & models picker widgets (under settings/presentation/)
```

---

## 🧪 Verification & Sanity Checks

Run these standard verification routines before making edits or pushing commits:

### 1. Static Analysis
Run the Flutter analyzer to confirm code warning compliance:
```bash
flutter analyze
```
*Expected Output:* `No issues found!` (0 errors, 0 warnings)

### 2. Automated Test Suite
Execute the unit and widget test runner:
```bash
flutter test
```
*Expected Output:* `All tests passed!` (345 tests passed, 0 failures)

---

## 📜 Credits & Open Source Licenses

We are deeply grateful to the open-source community for making these resources available.

### Localization Data
* **CLDR — Common Locale Data Repository**: Emoji names and keyword annotations for all 153 compiled languages. Created by Unicode, Inc. and its contributors. Licensed under the [Unicode License v3](https://unicode.org/license.txt). Source: [cldr.unicode.org](https://cldr.unicode.org)

### Emoji Art & Assets
* **OpenMoji** (Optional Pack): Created by the OpenMoji Project. Licensed under [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/).
* **Google Noto Color Emojis** (Static & 3D Animated): Created by Google LLC. Licensed under [SIL Open Font License 1.1 (OFL-1.1)](https://scripts.sil.org/OFL) and [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/).
* **Microsoft Fluent UI Emojis** (Static & 3D Animated): Created by Microsoft Corporation. Licensed under [MIT License](https://opensource.org/licenses/MIT).

### Typography & Fonts
* **Google Fonts**: Bundles 27 designer fonts (Montserrat, Outfit, Bebas Neue, Anton, Poppins, Raleway, Space Grotesk, Oswald, Righteous, etc.). Licensed under [SIL Open Font License 1.1 (OFL-1.1)](https://scripts.sil.org/OFL).
* **Noto Sans CJK Translation Fonts**: Preinstalled Chinese, Japanese, and Korean (SC/JP/KR/TC) font glyphs. Created by Google LLC & Adobe Inc. Licensed under [OFL-1.1](https://scripts.sil.org/OFL).

### Software Libraries & Binaries
* **whisper.cpp** (Local Speech-to-Text): Created by Georgi Gerganov and contributors. Licensed under [MIT License](https://opensource.org/licenses/MIT).
* **FFmpeg** (Audio & Video Processing): Created by the FFmpeg Project. Licensed under [GNU Lesser General Public License 2.1 (LGPLv2.1)](https://www.gnu.org/licenses/old-licenses/lgpl-2.1.html) or later.
* **Flutter Framework**: Created by Google LLC and contributors. Licensed under [BSD 3-Clause License](https://opensource.org/licenses/BSD-3-Clause).
* **Isar Database**: Created by Simon Leier. Licensed under [Apache License 2.0](http://www.apache.org/licenses/LICENSE-2.0).

