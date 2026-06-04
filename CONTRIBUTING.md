# CapStudio Developer & Contribution Guide

Welcome to the **CapStudio** contributor guide! CapStudio is a free, 100% offline AI-powered caption studio for video creators. This guide walks you through setting up your local environment, running tests, resolving dependencies, and building the application for **Windows, macOS, Linux, Android, and iOS**.

---

## 🛠️ Global Prerequisites

No matter your host OS, make sure you have the following toolchains installed:
1. **Flutter SDK**: Ensure you are on the `stable` channel. Run `flutter --version` to check.
2. **Git**: Used for version control and managing submodules.
3. **whisper.cpp Submodule**: Ensure you initialize native submodules after cloning:
   ```bash
   git submodule update --init --recursive
   ```

---

## 📦 General Setup

Run these commands in the root of the project to retrieve dependencies and run code generators:

```bash
# 1. Fetch Flutter/Dart packages
flutter pub get

# 2. Generate Isar database schemas
dart run build_runner build --delete-conflicting-outputs
```

Before committing any code, always verify it passes code styling rules and the test suite:
```bash
# Analyze code structure (must report "No issues found!")
flutter analyze

# Run the full suite of unit/widget tests
flutter test
```

---

## 💻 OS-Specific Guides

### 1. Windows Setup
* **Prerequisites**: Visual Studio (with C++ Desktop development workload).
* **Build App**:
  ```bash
  flutter build windows --release
  ```
  Output folder: `build\windows\x64\runner\Release\`
* **Create Installer**: Download and install **Inno Setup**, then compile the installer script:
  ```powershell
  iscc scripts/capstudio_setup.iss
  ```
* **Unified Tool**: We have provided a PowerShell script to automate cleanups, analysis, tests, submodules, and builds:
  ```powershell
  .\scripts\build_and_update.ps1
  ```

---

### 🤖 Android Setup (Cross-Platform Compile)
Android compilation compiles native C++ wrappers via the NDK. You can compile Android on **Windows, macOS, or Linux**.

* **Prerequisites**:
  - Android Studio installed.
  - Android SDK and NDK installed.
  - Set `ANDROID_HOME` pointing to your Android SDK (e.g. `C:\Users\<user>\AppData\Local\Android\Sdk`).
* **NDK Pinning**:
  Verify the `ndkVersion` inside `android/app/build.gradle.kts` matches your local installed NDK (e.g. `"28.2.13676358"`):
  ```kotlin
  android {
      ndkVersion = "28.2.13676358" // Match your local NDK version
  }
  ```
* **JDK Environment Configuration**:
  Use the JetBrains Runtime (JBR) JDK bundled with Android Studio:
  - *Windows*: Set `JAVA_HOME` to `C:\Program Files\Android\Android Studio\jbr`
  - *macOS*: Set `JAVA_HOME` using `/usr/libexec/java_home`
* **Compiling the App**:
  - **Debug APK** (local emulator/device testing):
    ```bash
    flutter build apk --debug --target-platform android-arm64
    ```
  - **Release APK** (automatically falls back to debug signature if release key env vars are not set):
    ```bash
    flutter build apk --release --target-platform android-arm64
    ```
  - **Signed Release App Bundle** (Google Play Store):
    Set your keystore environment variables and build:
    ```bash
    export KEYSTORE_FILE=/path/to/release.jks
    export KEYSTORE_PASSWORD=your_password
    export KEY_ALIAS=capstudio
    export KEY_PASSWORD=your_password
    flutter build appbundle --release
    ```

---

### 🍏 macOS & iOS Setup (Requires macOS host)
iOS and macOS builds require Xcode and Xcode Command Line Tools.

* **Prerequisites**:
  - macOS machine with Xcode installed.
  - CocoaPods installed: `sudo gem install cocoapods`
* **Step 1: Compile iOS Native Whisper Library**:
  Generate the Apple `xcframework` by compiling the native `whisper.cpp` C++ target:
  ```bash
  chmod +x scripts/build_whisper_ios.sh
  ./scripts/build_whisper_ios.sh
  ```
  This creates `ios/whisper_xcframework/whisper.xcframework`. (Note: The project files are pre-configured to link this path. Do not move or rename it).
* **Step 2: Install CocoaPods dependencies**:
  ```bash
  cd ios
  pod install --repo-update
  cd ../macos
  pod install --repo-update
  cd ..
  ```
* **Step 3: Run / Compile Builds**:
  - **macOS Desktop**:
    ```bash
    flutter build macos --release
    ```
  - **iOS Device / Simulator**:
    ```bash
    flutter run -d ios
    # For release archive
    flutter build ios --release --no-codesign
    ```
* **Unified Tool**: We have provided a Bash controller script to automate cleanups, analysis, tests, and builds:
  ```bash
  chmod +x scripts/build_and_update.sh
  ./scripts/build_and_update.sh
  ```

---

### 🐧 Linux Setup
* **Prerequisites**: Install platform dependencies:
  ```bash
  sudo apt-get update
  sudo apt-get install -y clang cmake ninja-build pkg-config libgtk-3-dev liblzma-dev libstdc++-12-dev
  ```
* **Compile Build**:
  ```bash
  flutter build linux --release
  ```
* **Build .deb Package**:
  Use the automated script to construct a `.deb` binary:
  ```bash
  chmod +x /tmp/create_deb.sh
  /tmp/create_deb.sh
  ```

---

## ⚖️ Code Style Rules & Constraints

Please adhere to these guidelines to prevent build errors and runtime crashes:
* **No CDN Font Loading**: Do not call `GoogleFonts.xyz()` dynamically at runtime. Use bundle assets.
* **Dropdown Widgets**: Use `value:` in `DropdownButtonFormField` rather than the deprecated `initialValue:`.
* **Database Versioning**: Never use the `migrationCallback` parameter within `Isar.open` directly. Custom database initialization code handles migration strategies before database opening.
* **FFmpeg Quoting**: Never join FFmpeg args as a single string containing quotes. Always use `executeWithArguments(List<String>)` to safely isolate paths containing spaces on mobile.
* **No Checked Binaries**: Never commit compiled `.so`, `.a`, `.dll`, or `.xcframework` binaries to git. They must remain in `.gitignore`.
