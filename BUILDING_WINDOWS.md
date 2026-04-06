<!--
SPDX-FileCopyrightText: 2026 Kaidan contributors

SPDX-License-Identifier: CC0-1.0
-->

# Building Kaidan on Windows (Step by Step)

This guide is intentionally detailed and assumes little prior build experience.

## 1) Choose your build method

You have two options:

1. **KDE Craft (recommended)**  
   Best if you want the same dependency flow as KDE CI and the easiest path.
2. **Manual CMake build**  
   For advanced users who already have a full Qt/KF6 development environment.

If you are unsure, choose **KDE Craft**.

---

## 2) Prerequisites (both methods)

Install these first:

1. **Visual Studio 2022** (or Build Tools) with C++ workload:
   - MSVC compiler (x64)
   - Windows SDK
2. **Git**
3. **Python 3**
4. **CMake** and **Ninja**

> Tip: Run commands from a **Developer PowerShell for VS 2022** unless noted otherwise.

---

## 3) Get the source

```powershell
git clone https://invent.kde.org/network/kaidan.git
cd kaidan
```

---

## 4) Method A (recommended): KDE Craft

KDE Craft installs and manages nearly all dependencies needed by Kaidan.

1. Follow Craft setup: <https://community.kde.org/Craft>
2. Build Kaidan:

   ```powershell
   craft kde/kaidan
   ```

3. Open a Craft shell (from your Craft setup) and run Kaidan there so the
   required runtime DLL paths are available.

### Why this method is recommended

* It matches KDE's Windows CI/Craft packaging workflow.
* It reduces manual dependency troubleshooting.

---

## 5) Method B: Manual CMake build (advanced)

Use this only if you already have Qt6 + KF6 + all Kaidan dependencies installed.

### 5.1 Required dependencies to have installed

At minimum, ensure CMake can find:

* Qt 6 modules used by Kaidan
* KDE Frameworks 6 modules (`ECM`, `KIO`, `Kirigami`, `Prison`, `KWindowSystem`, `KIconThemes` on Windows)
* `KDSingleApplication-qt6`
* `KF6KirigamiAddons`
* `QXmppQt6` (with OMEMO)
* `Qt6Keychain`
* ICU
* GStreamer 1.20+ (plus `pkg-config` visibility for `gstreamer-1.0`)

### 5.2 Configure

```powershell
cmake -S . -B build -G Ninja -DCMAKE_BUILD_TYPE=Release
```

If configure fails with missing package errors, set `CMAKE_PREFIX_PATH`
to include your Qt/KF6/dependency install roots:

```powershell
cmake -S . -B build -G Ninja -DCMAKE_BUILD_TYPE=Release `
  -DCMAKE_PREFIX_PATH="C:\Qt\6.x\msvcxxxx_64;C:\kde\prefix;C:\deps\prefix"
```

You can also set specific package directories (example):

```powershell
-DECM_DIR="C:\kde\prefix\lib\cmake\ECM"
```

### 5.3 Build

```powershell
cmake --build build
```

### 5.4 Run

Run from an environment where all required Qt/KF6/GStreamer DLLs are on `PATH`
or deployed next to the executable.

---

## 6) Common Windows build errors and fixes

### Error: `Could not find ECMConfig.cmake`

Cause: `extra-cmake-modules` is not installed/found.  
Fix: install ECM and add its prefix to `CMAKE_PREFIX_PATH` or set `ECM_DIR`.

### Error: missing `KF6Config.cmake` / `KF6KirigamiAddonsConfig.cmake`

Cause: KDE Frameworks/Kirigami Addons missing from your CMake search path.  
Fix: add the matching install roots to `CMAKE_PREFIX_PATH`.

### Error: missing `QXmppQt6Config.cmake` or `Qt6KeychainConfig.cmake`

Cause: package not installed or wrong prefix path.  
Fix: install package and ensure CMake can see its `lib/cmake/...` directory.

### Error: GStreamer / `pkg-config` problems

Cause: `gstreamer-1.0` is required and discovered through `pkg-config`.  
Fix: install GStreamer development files and make sure `pkg-config` can resolve
`gstreamer-1.0` in your shell environment.

---

## 7) CI/toolchain alignment in this repository

To understand what the project expects on Windows:

* `.gitlab-ci.yml` includes the Windows Qt6 CI and Craft packaging jobs.
* `.kde-ci.yml` defines shared dependency requirements used by KDE CI.
* `CMakeLists.txt` contains platform-specific dependency conditions, including
  Windows requirements such as `KF6::IconThemes`.
