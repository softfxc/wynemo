_**BETA VERSION:**_
What the app does
•	Opens a .exe file through the Android Storage Access Framework.
•	Loads the file into memory and displays the PE 32 header and other relevant information.
•	Native inspection code is written in C/C++ and compiled with CMake; Kotlin calls the native function via JNI (nativeInspectPe).
Main technologies
Component	Version
Android Studio	2025.3
Android Gradle Plugin	8.8.2 (updated from 8.5.2)
Gradle Wrapper	8.10.2
Kotlin	2.0.21
compileSdk / targetSdk	35
NDK	27.0.12077973 (alternative 26.1.10909125 also tested)
CMake	3.22.1
JDK (embedded in Android Studio)	JBR 21.0.9
Project layout
Wynemo/
├─ app/
│   ├─ build.gradle.kts
│   └─ src/main/
│       ├─ AndroidManifest.xml
│       ├─ java/com/wynemo/app/MainActivity.kt
│       └─ res/layout/activity_main.xml
├─ native/
│   ├─ CMakeLists.txt
│   ├─ include/wynemo/
│   └─ src/
├─ build.gradle.kts
├─ gradle.properties
└─ settings.gradle.kts
Wynemo/
├─ app/
│   ├─ build.gradle.kts
│   └─ src/main/
│       ├─ AndroidManifest.xml
│       ├─ java/com/wynemo/app/MainActivity.kt
│       └─ res/layout/activity_main.xml
├─ native/
│   ├─ CMakeLists.txt
│   ├─ include/wynemo/
│   └─ src/
├─ build.gradle.kts
├─ gradle.properties
└─ settings.gradle.kts
Core functionality (MainActivity)
1.	Shows the status of the native core at launch.
2.	Opens a file selector.
3.	Reads the selected file as a binary array.
4.	Calls nativeInspectPe(data, fileName).
5.	Presents the returned information on the screen.
Build environment requirements (Windows)
1.	Android Studio installed (default path C:\Program Files\Android\Android Studio).
2.	JDK – use the bundled JBR (C:\Program Files\Android\Android Studio\jbr).
3.	Android SDK – must contain: 
o	platform tools
o	platforms;android 35
o	build tools;35.0.0
o	cmake;3.22.1
o	ndk;27.0.12077973 (or the older 26.1 version).
4.	Gradle version 8.10.2 installed locally (the wrapper will be generated).
Quick setup steps (PowerShell)
powershell
# 1. Set Java to the embedded JBR
$env:JAVA_HOME = "C:\Program Files\Android\Android Studio\jbr"
$env:Path = "$env:JAVA_HOME\bin;C:\Gradle\gradle-8.10.2\bin;$env:Path"
java -version   # should show openjdk 21.x

# 2. Fix gradle.properties (replace any empty org.gradle.java.home line)
@"
org.gradle.jvmargs=-Xmx2048m -Dfile.encoding=UTF-8
android.useAndroidX=true
kotlin.code.style=official
android.nonTransitiveRClass=true
org.gradle.java.home=C:/Program Files/Android/Android Studio/jbr
"@ | Set-Content "C:\Users\Usuario\Downloads\Wynemo\gradle.properties" -Encoding UTF8

# 3. Install command line tools if they are missing
$Sdk = "$env:LOCALAPPDATA\Android\Sdk"
if (-not (Test-Path "$Sdk\cmdline-tools\latest\bin\sdkmanager.bat")) {
    # download and unpack the latest command line tools package into $Sdk\cmdline-tools\latest
    # (implementation left to the user)
}

# 4. Accept licences and install required SDK packages
& "$Sdk\cmdline-tools\latest\bin\sdkmanager.bat" --sdk_root=$Sdk --licenses
& "$Sdk\cmdline-tools\latest\bin\sdkmanager.bat" --sdk_root=$Sdk `
    "platform-tools" "platforms;android-35" "build-tools;35.0.0" `
    "cmake;3.22.1" "ndk;27.0.12077973"

# 5. Create local.properties so Gradle can locate the SDK
$Project = "C:\Users\Usuario\Downloads\Wynemo"
$SdkEscaped = $Sdk.Replace('\','\\')
Set-Content -Path "$Project\local.properties" -Encoding ascii -Value "sdk.dir=$SdkEscaped"

# 6. (Optional) Upgrade AGP to 8.8.2 – edit the root build.gradle.kts
$RootGradle = "$Project\build.gradle.kts"
(Get-Content $RootGradle -Raw) `
  -replace 'id\("com.android.application"\) version "8\.5\.2"', `
           'id("com.android.application") version "8.8.2"' |
  Set-Content -Path $RootGradle -Encoding UTF8

# 7. Generate the Gradle wrapper
cd $Project
gradle wrapper --gradle-version 8.10.2

# 8. Build the app (debug)
.\gradlew.bat assembleDebug --warning-mode all
Known error messages and fixes
Message	Typical cause	Fix
sdkmanager not recognized	Command line tools not installed or not on PATH	Run step 3 to install them.
Java version 17 or higher is required	Gradle is using an old Java runtime	Ensure JAVA_HOME points to the JBR 21 directory (step 1).
Value '' given for org.gradle.java.home ... is invalid	Empty org.gradle.java.home line in gradle.properties	Replace it with the full JBR path (step 2).
gradlew.bat not found	Wrapper has not been generated	Run step 7.
Could not resolve com.android.tools.build:gradle:8.8.2 … uses a Java 8 JVM	Gradle started with the wrong JVM	Verify JAVA_HOME and org.gradle.java.home point to the JBR 21 installation.

**What is included in this pre release**
•	All source files for the Android app and the native CMake module.
•	Updated build.gradle.kts files with AGP 8.8.2.
•	A complete PowerShell bootstrap script (the steps above) that sets up the SDK, NDK, CMake, generates the Gradle wrapper, and builds the app.


_**RELEASE:**_

Wynemo is an Android project aimed at running Windows software from an Android device, with two distinct but complementary paths:

**App Runner:** open and launch Windows executables using a backend such as Wine + Box64/Box86.

**ISO:** open disk images and start an installation or boot of a real Windows system inside a virtual machine using a backend such as QEMU system.

## Project Vision

The idea behind Wynemo is not to remain just an executable viewer or a simple experiment with PE headers. The vision is more ambitious:

* to be able to open a `.exe` from Android and eventually see it truly running;
* to allow the app itself to manage dependencies and runtimes;
* to create and reuse environments or containers for different apps;
* to offer a separate path for Windows ISOs and booting a real Windows system inside a VM;
* to download the necessary binaries directly from the UI in a versioned, verifiable, and reproducible way;
* to maintain a clean architecture, where the Android app acts as launcher, environment manager, and runtime manager, while the native backends handle execution.

## Current Project State

Wynemo started as an Android app that:

* opens a `.exe` file using the system picker;
* reads it as a binary;
* passes it to the native core through JNI;
* shows PE information on screen.

From there, the project evolved through several internal betas:

### Wynemo 0.2 beta

First step toward real execution:

* Android UI with the flow `Open .exe -> Inspect / Execute`
* expanded PE32 x86 loader
* section parsing
* function-by-function import parsing
* IAT patching to stubs
* custom mini x86 emulator
* execution from the EntryPoint
* some basic simulated host APIs for `kernel32.dll`, `msvcrt.dll`, and `user32.dll`

Limitations of that stage:

* PE32 x86 only
* no x64
* no full SEH, TLS, SSE/FPU, threads, or real Win32
* many x86 instructions still unimplemented
* useful as a technical lab, but not enough to truly “see apps running”

### Wynemo 0.3 beta

The project pivoted from a “homemade emulator” to a launcher frontend:

* local executable library
* editable profiles
* quick PE architecture and subsystem detection
* launch sessions with logs
* backend selection from Settings

### Wynemo 0.4 beta

Correct architectural separation into two flows:

* `EXE / MSI -> App Runner`
* `ISO -> VM / installation`

This formalizes a key idea: launching Windows apps is not the same as booting a full Windows system.

### Wynemo 0.5 beta

First pieces at the UI and model level:

* Runtime Manager
* Environment Manager
* dependency check before launch
* trash with restore and permanent delete

### Wynemo 0.6 beta

First real foundation for remote runtime distribution:

* remote JSON manifest
* real HTTP/HTTPS download
* installation in private storage
* SHA-256 verification
* ZIP extraction
* files marked as read-only
* remote package catalog with progress and cancellation

## What Wynemo Wants to Be

Wynemo aims to be an Android platform with these capabilities:

* add `.exe`, `.msi`, and `.iso` files to a library;
* create reusable environments;
* assign an environment to each app;
* detect required dependencies and offer to download them from within the app;
* download runtimes;
* use an execution backend for Windows apps;
* use a VM backend for ISOs and full Windows systems;
* handle trash, restore, and cleanup;
* preserve profiles, logs, preferences, and paths.

## 1. Android Layer

Responsible for:

* UI
* app library
* profiles
* environments
* downloads
* dependency management
* settings
* logs
* trash

## 2. Runtime Manager

Responsible for:

* querying the remote catalog
* downloading packages
* verifying integrity
* installing into private storage
* versioning runtimes
* updating / repairing / deleting

## 3. Environment Manager

Responsible for:

* creating reusable environments
* assigning configuration per app
* separating different prefixes or virtual machines
* storing variables, resolution, overrides, and parameters

## 4. App Runner Backend

Focused on:

* `.exe`
* `.msi`
* Wine
* Box64
* Box86 if applicable
* per-environment configuration

## 5. VM / ISO Backend

Focused on:

* `.iso`
* virtual disks
* QEMU system
* booting a real Windows guest

## Why There Are Two Different Modes

Because these are two different problems:

### App Runner Mode

Used to run individual Windows applications.

**Input:**

* `.exe`
* `.msi`

**Expected output:**

* see the app open
* logs
* session control
* use of an assigned environment

### VM / ISO Mode

Used to install or boot a complete Windows operating system.

**Input:**

* `.iso`

**Expected output:**

* create a virtual disk
* boot the installer
* install the system
* preserve the VM for later use

## Core Project Ideas

These are the central ideas Wynemo wants to materialize:

* the app itself should request missing dependencies, instead of forcing the user to find them manually;
* it should be able to create environments;
* it should allow downloading the necessary binaries from the UI;
* there should be a trash system with confirmation before deleting;
* the user should be able to work with both Windows apps and Windows ISOs;
* the project should focus on seeing things actually run, not only reading text or inspecting headers;
* runtimes should be distributed as versioned and verifiable packages.

## Planned UI Functionality

### Library

Supported items:

* `.exe`
* `.msi`
* `.iso`

Each entry can store:

* name
* URI / content path
* arguments
* working directory
* detected architecture
* execution subtype
* assigned environment
* notes

### Runtime Manager

It should show:

* Wine
* Box64
* Box86
* Wine Mono
* QEMU

Operations:

* download
* update
* repair
* uninstall
* view version
* view source
* view hash
* view status

### Environment Manager

Each environment should be able to store:

* name
* mode (`app-runner` or `vm`)
* preferred runtime version
* environment variables
* prefix path
* resolution
* default arguments
* DLL overrides
* notes

### Dependency Preflight

Before launch:

* detect what is missing
* show a dialog
* allow download
* block launch if the minimum runtime is not installed

### Trash

Before moving something to trash:

* show a confirmation notice
* indicate whether it affects an environment
* indicate whether deletion is reversible
* allow later restore
* empty trash manually

## Project Structure

A reasonable structure for Wynemo is this:

```text
Wynemo/
├─ app/
│  ├─ src/main/
│  │  ├─ java/com/wynemo/app/
│  │  ├─ res/
│  │  └─ AndroidManifest.xml
│  └─ build.gradle.kts
├─ core-model/
├─ core-storage/
├─ runtime-api/
├─ runtime-manager/
├─ runtime-winebox/
├─ runtime-qemu/
├─ downloader/
├─ environments/
├─ native/
├─ docs/
├─ gradle/
├─ settings.gradle.kts
└─ build.gradle.kts
```

## Android Project Requirements

The public repository currently describes these base versions:

* Android Studio 2025.3
* Android Gradle Plugin 8.8.2
* Gradle 8.10.2
* Kotlin 2.0.21
* compileSdk / targetSdk 35
* NDK 27.0.12077973
* CMake 3.22.1

## Open in Android Studio

1. Open Android Studio
2. `File > Open`
3. Select the `wynemo` folder
4. Wait for Gradle Sync
5. Install missing SDK components if Android Studio asks for them

## Build from Terminal

### Windows

```powershell
.\gradlew.bat clean
.\gradlew.bat assembleDebug
```

### Linux / macOS

```bash
./gradlew clean
./gradlew assembleDebug
```

Expected APK:

```text
app/build/outputs/apk/debug/
```

## Recommended Windows Setup to Build the Android App

If you build from Windows with Android Studio installed:

```powershell
$env:JAVA_HOME = "C:\Program Files\Android\Android Studio\jbr"
$env:Path = "$env:JAVA_HOME\bin;$env:Path"
java -version
```

Install or verify in the SDK:

* `platform-tools`
* `platforms;android-35`
* `build-tools;35.0.0`
* `cmake;3.22.1`
* `ndk;27.0.12077973`

Then:

```powershell
.\gradlew.bat assembleDebug --warning-mode all
```

## Current Usage Flow

### Add an Executable

1. Open the app
2. Select a file with the system picker
3. Add `.exe` or `.msi` to the library
4. Assign an environment
5. Check dependencies
6. Launch

### Add an ISO

1. Open the app
2. Select a `.iso`
3. Create or choose a VM profile
4. Assign a virtual disk
5. Launch the VM backend

## Real Runtime Downloads

Since 0.6, the architecture already contemplates this:

* configure a remote manifest URL;
* refresh the catalog;
* download a package;
* verify SHA-256;
* install it in private storage;
* extract ZIP if needed;
* mark binaries as read-only;
* expose them to a future backend.

## Publishing Runtimes on GitHub Releases

The planned idea is to publish runtimes at:

```text
https://github.com/softfxc/wynemo/releases
```

### Recommended Asset Naming Convention

```text
wynemo-runtime-box64-android-arm64-v0.4.0.tar.zst
wynemo-runtime-qemu-system-aarch64-android-arm64-v10.2.1.tar.zst
wynemo-runtime-wine-wow64-glibc-arm64-v11.4.tar.zst
SHA256SUMS.txt
manifest.json
```

## Environments

One of Wynemo’s pillars is that not everything should be configured app by app. There should be reusable environments.

### An environment can represent:

* a Wine prefix
* a Wine + Box64 combination
* a VM associated with an ISO
* a special configuration for a specific app

### Recommended fields

* id
* name
* mode
* primary runtime
* auxiliary runtime
* resolution
* variables
* prefixPath
* diskPath
* notes

### Planned operations

* create
* duplicate
* export
* import
* repair
* move to trash
* restore
* permanently delete

## Trash

Wynemo should include trash for usability and safety.

### Expected behavior

When moving something to trash, the app should show a notice like this:

**Move to trash**

This item will no longer be available in the library.
It will not be permanently deleted until the trash is emptied.

If it is an environment, the app should also show:

* how many apps depend on it
* whether the prefix will be preserved
* whether logs or sessions will be preserved
* approximate disk size

## Dependency Support

One of the project’s main ideas is that the user should not have to guess what is missing.

### Planned flow

1. The user presses Launch
2. Wynemo analyzes the profile and the environment
3. Wynemo detects missing dependencies
4. A dialog is shown with:

   * what is missing
   * size
   * version
   * download button
5. After installing dependencies, launch is retried

## Roadmap

### Phase 1

Solid Android launcher:

* library
* profiles
* environments
* trash
* runtime catalog
* real downloads
* integrity verification

### Phase 2

Real App Runner backend:

* Wine integration
* Box64 integration
* Box86 or a later alternative
* prefix management
* real logs
* real launch process

### Phase 3

Real VM backend:

* QEMU system
* virtual disks
* boot from ISO
* Windows installation
* snapshot and storage management

### Phase 4

Complete experience:

* shortcuts
* presets
* import/export of environments
* runtime updates
* diagnostics
* cleanup
* repair

## Building Runtimes in a Linux VM

### Practical recommendation

Use a Linux work VM and compile there:

* Box64
* QEMU
* Wine
* later Box86 if it truly makes sense

### Example VM bootstrap

```bash
sudo apt update
sudo apt install -y git curl wget unzip zip zstd cmake ninja-build build-essential pkg-config python3
```

### Clone Wynemo

```bash
git clone https://github.com/softfxc/wynemo.git
cd wynemo
```

### Download Android NDK

```bash
wget https://dl.google.com/android/repository/android-ndk-r27c-linux.zip
unzip android-ndk-r27c-linux.zip
export ANDROID_NDK="$PWD/android-ndk-r27c"
```

## Example Box64 Build

```bash
git clone --depth=1 --branch v0.4.0 https://github.com/ptitSeb/box64.git
cd box64
mkdir build-android
cd build-android

cmake .. \
  -DCMAKE_BUILD_TYPE=Release \
  -DCMAKE_TOOLCHAIN_FILE="$ANDROID_NDK/build/cmake/android.toolchain.cmake" \
  -DANDROID_ABI=arm64-v8a \
  -DANDROID_PLATFORM=26

cmake --build . -j"$(nproc)"
```

Then package it:

```bash
mkdir -p out/box64/bin
cp box64 out/box64/bin/
tar --zstd -cf wynemo-runtime-box64-android-arm64-v0.4.0.tar.zst -C out box64
sha256sum wynemo-runtime-box64-android-arm64-v0.4.0.tar.zst
```

## Example QEMU Build

```bash
wget https://download.qemu.org/qemu-10.2.1.tar.xz
tar xf qemu-10.2.1.tar.xz
cd qemu-10.2.1
mkdir build
cd build

export CC="$ANDROID_NDK/toolchains/llvm/prebuilt/linux-x86_64/bin/aarch64-linux-android26-clang"
export CXX="$ANDROID_NDK/toolchains/llvm/prebuilt/linux-x86_64/bin/aarch64-linux-android26-clang++"
export AR="$ANDROID_NDK/toolchains/llvm/prebuilt/linux-x86_64/bin/llvm-ar"
export RANLIB="$ANDROID_NDK/toolchains/llvm/prebuilt/linux-x86_64/bin/llvm-ranlib"
export STRIP="$ANDROID_NDK/toolchains/llvm/prebuilt/linux-x86_64/bin/llvm-strip"

../configure --target-list=aarch64-softmmu --disable-werror
make -j"$(nproc)"
```

Package it:

```bash
mkdir -p out/qemu/bin
cp qemu-system-aarch64 out/qemu/bin/
tar --zstd -cf wynemo-runtime-qemu-system-aarch64-android-arm64-v10.2.1.tar.zst -C out qemu
sha256sum wynemo-runtime-qemu-system-aarch64-android-arm64-v10.2.1.tar.zst
```

## GitHub for Publishing Assets

The idea behind Wynemo is to automate runtime publishing with tags, so the app can download versioned packages from GitHub Releases.

### Planned flow

* create tag
* build runtime
* package it
* calculate hash
* upload asset
* update `manifest.json`
