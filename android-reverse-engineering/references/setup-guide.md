# Setup Guide: Dependencies for Android Reverse Engineering

## Java JDK 17+

jadx requires Java 17 or later.

### Windows (PowerShell)

**Option 1: winget (recommended)**

```powershell
winget install Microsoft.OpenJDK.17
```

**Option 2: Chocolatey**

```powershell
choco install openjdk17
```

**Option 3: Manual**

1. Go to <https://adoptium.net/temurin/releases/?version=17>
2. Download the `.msi` installer for Windows x64
3. Run the installer — it adds Java to `PATH` automatically

### Ubuntu / Debian

```bash
sudo apt update
sudo apt install openjdk-17-jdk
```

### Fedora

```bash
sudo dnf install java-17-openjdk-devel
```

### Arch Linux

```bash
sudo pacman -S jdk17-openjdk
```

### macOS (Homebrew)

```bash
brew install openjdk@17
```

After installation on macOS, follow the symlink instructions printed by Homebrew, or add to your shell profile:

```bash
export PATH="/opt/homebrew/opt/openjdk@17/bin:$PATH"
```

### Verify

```bash
java -version
# Should show version 17.x or higher
```

---

## jadx

jadx is the Java decompiler used to convert APK/JAR/AAR files to readable Java source.

### Windows (PowerShell)

**Option 1: GitHub Releases (recommended)**

The snippet below automatically resolves the latest version, so it will always work regardless of future releases:

```powershell
# Resolve latest version tag and download
$release = Invoke-RestMethod "https://api.github.com/repos/skylot/jadx/releases/latest"
$version = $release.tag_name.TrimStart("v")
$url = "https://github.com/skylot/jadx/releases/download/v$version/jadx-$version.zip"

Invoke-WebRequest -Uri $url -OutFile "$env:TEMP\jadx.zip"
Expand-Archive "$env:TEMP\jadx.zip" -DestinationPath "$env:USERPROFILE\jadx" -Force

# Add to PATH for current session
$env:PATH += ";$env:USERPROFILE\jadx\bin"

# Add to PATH permanently (takes effect in new terminals)
[Environment]::SetEnvironmentVariable("PATH", $env:PATH + ";$env:USERPROFILE\jadx\bin", "User")
```

**Option 2: Chocolatey**

```powershell
choco install jadx
```

**Option 3: Scoop**

```powershell
scoop install jadx
```

### GitHub Releases (Linux/macOS)

1. Go to <https://github.com/skylot/jadx/releases/latest>
2. Download the `jadx-<version>.zip` file (not the source archive)
3. Extract and add to PATH:

```bash
unzip jadx-*.zip -d ~/jadx
export PATH="$HOME/jadx/bin:$PATH"
# Add the export line to your ~/.bashrc or ~/.zshrc for persistence
```

### Homebrew (macOS / Linux)

```bash
brew install jadx
```

### Build from source

```bash
git clone https://github.com/skylot/jadx.git
cd jadx
./gradlew dist
# Binaries will be in build/jadx/bin/
export PATH="$(pwd)/build/jadx/bin:$PATH"
```

### Verify

```powershell
# Windows
jadx.bat --version
```

```bash
# Linux / macOS
jadx --version
```

---

## Fernflower / Vineflower (optional, recommended)

Fernflower is the JetBrains Java decompiler. It produces better output than jadx on complex Java constructs, lambdas, and generics. [Vineflower](https://github.com/Vineflower/vineflower) is the actively maintained community fork and is recommended over upstream Fernflower.

### Windows (PowerShell)

```powershell
# Resolve latest version and download
$release = Invoke-RestMethod "https://api.github.com/repos/Vineflower/vineflower/releases/latest"
$version = $release.tag_name.TrimStart("v")
$url = "https://github.com/Vineflower/vineflower/releases/download/$($release.tag_name)/vineflower-$version.jar"

New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\vineflower"
Invoke-WebRequest -Uri $url -OutFile "$env:USERPROFILE\vineflower\vineflower.jar"

# Set environment variable for current session
$env:FERNFLOWER_JAR_PATH = "$env:USERPROFILE\vineflower\vineflower.jar"

# Set permanently (takes effect in new terminals)
[Environment]::SetEnvironmentVariable("FERNFLOWER_JAR_PATH", "$env:USERPROFILE\vineflower\vineflower.jar", "User")
```

### Vineflower from GitHub Releases (Linux/macOS)

1. Go to <https://github.com/Vineflower/vineflower/releases/latest>
2. Download `vineflower-<version>.jar`
3. Place it and set the environment variable:

```bash
mkdir -p ~/vineflower
mv vineflower-*.jar ~/vineflower/vineflower.jar
export FERNFLOWER_JAR_PATH="$HOME/vineflower/vineflower.jar"
# Add the export to ~/.bashrc or ~/.zshrc for persistence
```

### Build Fernflower from source

```bash
git clone https://github.com/JetBrains/fernflower.git
cd fernflower
./gradlew jar
# Produces: build/libs/fernflower.jar
export FERNFLOWER_JAR_PATH="$(pwd)/build/libs/fernflower.jar"
```

### Homebrew (Vineflower, macOS/Linux)

```bash
brew install vineflower
```

### Verify

```powershell
# Windows
java -jar "$env:FERNFLOWER_JAR_PATH" --version
```

```bash
# Linux / macOS
java -jar "$FERNFLOWER_JAR_PATH" --version
```

> **Note**: Fernflower only works on JVM bytecode (JAR, class files). For APK/DEX files, you also need **dex2jar** (see below) as an intermediate conversion step.

---

## dex2jar (optional, needed for Fernflower on APK files)

Converts Android DEX bytecode to standard Java JAR files.

### Windows (PowerShell)

```powershell
# Resolve latest version and download
$release = Invoke-RestMethod "https://api.github.com/repos/pxb1988/dex2jar/releases/latest"
$asset = $release.assets | Where-Object { $_.name -like "dex-tools-*.zip" } | Select-Object -First 1
Invoke-WebRequest -Uri $asset.browser_download_url -OutFile "$env:TEMP\dex2jar.zip"

Expand-Archive "$env:TEMP\dex2jar.zip" -DestinationPath "$env:USERPROFILE\dex2jar" -Force

# Find extracted folder name and add to PATH
$folder = (Get-ChildItem "$env:USERPROFILE\dex2jar" -Directory | Select-Object -First 1).FullName
$env:PATH += ";$folder"
[Environment]::SetEnvironmentVariable("PATH", $env:PATH + ";$folder", "User")
```

### GitHub Releases (Linux/macOS)

1. Go to <https://github.com/pxb1988/dex2jar/releases/latest>
2. Download and extract:

```bash
unzip dex-tools-*.zip -d ~/dex2jar
export PATH="$HOME/dex2jar:$PATH"
```

### Homebrew (macOS/Linux)

```bash
brew install dex2jar
```

### Verify

```powershell
# Windows
d2j-dex2jar.bat --help
```

```bash
# Linux / macOS
d2j-dex2jar --help
```

### Usage

```powershell
# Windows
d2j-dex2jar.bat -f -o output.jar app.apk
java -jar "$env:FERNFLOWER_JAR_PATH" output.jar decompiled\
```

```bash
# Linux / macOS
d2j-dex2jar -f -o output.jar app.apk
java -jar vineflower.jar output.jar decompiled/
```

---

## Optional Tools

### apktool

Useful for decoding resources (XML layouts, drawables) that jadx sometimes handles poorly.

```powershell
# Windows (Chocolatey)
choco install apktool

# Windows (manual): https://apktool.org/docs/install
```

```bash
# Ubuntu/Debian
sudo apt install apktool

# macOS
brew install apktool

# Manual: https://apktool.org/docs/install
```

### adb (Android Debug Bridge)

Useful for pulling APKs directly from a connected Android device.

```powershell
# Windows (winget)
winget install Google.PlatformTools

# Windows (Chocolatey)
choco install adb
```

```bash
# Ubuntu/Debian
sudo apt install adb

# macOS
brew install android-platform-tools
```

Pull an APK from a device:

```bash
# List installed packages
adb shell pm list packages | grep <keyword>

# Get APK path
adb shell pm path com.example.app

# Pull the APK
adb pull /data/app/com.example.app-xxxx/base.apk ./app.apk
```

---

## Troubleshooting

| Problem | Solution |
|---|---|
| `jadx: command not found` / `jadx.bat` not recognized | Ensure the jadx `bin/` directory is in your `PATH` |
| `Error: Could not find or load main class` | Java is missing or wrong version — verify with `java -version` |
| `Invoke-WebRequest: Not Found` on Windows | The hardcoded version URL is outdated — use the dynamic snippet with `Invoke-RestMethod` instead |
| jadx runs out of memory on large APKs | Increase heap: `jadx -Xmx4g -d output app.apk` or set `JAVA_OPTS="-Xmx4g"` |
| Decompiled code has many `// Error` comments | Try `--show-bad-code` to see partial output, or use `--deobf` for obfuscated apps |
| Fernflower hangs on a method | Use `-mpm=60` to set a 60-second timeout per method |
| Fernflower JAR not found | Set `FERNFLOWER_JAR_PATH` env variable to the full path of the JAR |
| dex2jar fails with `ZipException` | The APK may have a non-standard ZIP structure — try `jadx` instead |
| Windows: `Execution Policy` error running scripts | Run `Set-ExecutionPolicy -Scope CurrentUser RemoteSigned` in PowerShell as admin |
