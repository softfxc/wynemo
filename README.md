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
