---
name: android-reverse-engineering
description: Provides a structured workflow for decompiling Android APK, XAPK,
  JAR, and AAR files using jadx or Fernflower/Vineflower, extracting HTTP API
  endpoints (Retrofit, OkHttp, Volley), and tracing call flows through
  obfuscated Android code. Use this skill when you need to reverse engineer an
  Android app, extract API endpoints, or analyze an APK/AAR library.
license: Apache-2.0
metadata:
  author: Pizza67 (ported from SimoneAvogadro/android-reverse-engineering-skill)
  keywords:
  - android
  - reverse engineering
  - APK
  - jadx
  - decompile
  - API extraction
  - call flow
  - Retrofit
  - OkHttp
  - obfuscation
---

# Android Reverse Engineering

Decompile Android APK, XAPK, JAR, and AAR files using jadx and Fernflower/Vineflower, trace call flows through application code and libraries, and produce structured documentation of extracted APIs.

## Prerequisites

This skill requires **Java JDK 17+** and **jadx** to be installed. **Fernflower/Vineflower** and **dex2jar** are optional but recommended for better decompilation quality.

Check dependencies by running:

```bash
jadx --version
java -version
```

If anything is missing, follow the installation instructions in `references/setup-guide.md`.

## Workflow

### Phase 1: Verify and Install Dependencies

Before decompiling, confirm that the required tools are available.

**Required:**
- Java JDK 17+: install via your system package manager or [adoptium.net](https://adoptium.net)
- [jadx](https://github.com/skylot/jadx): `brew install jadx` / `apt install jadx` / download from GitHub releases

**Optional (recommended):**
- [Vineflower](https://github.com/Vineflower/vineflower) or [Fernflower](https://github.com/JetBrains/fernflower) — better output on complex Java code
- [dex2jar](https://github.com/pxb1988/dex2jar) — needed to use Fernflower on APK/DEX files

See `references/setup-guide.md` for detailed installation instructions per OS.

Do not proceed to Phase 2 until all required dependencies are confirmed.

### Phase 2: Decompile

Use jadx (or Fernflower) to decompile the target file. Choose the engine based on the table below.

**Engine selection strategy:**

| Situation | Engine |
|---|---|
| First pass on any APK | jadx (fastest, handles resources) |
| JAR/AAR library analysis | fernflower (better Java output) |
| jadx output has warnings/broken code | both (compare and pick best per class) |
| Complex lambdas, generics, streams | fernflower |
| Quick overview of a large APK | jadx with `--no-res` |

**Decompile with jadx:**

```bash
# Standard decompilation
jadx -d <output-dir> <file.apk>

# With deobfuscation
jadx -d <output-dir> --deobf <file.apk>

# Code only (skip resources, faster)
jadx -d <output-dir> --no-res <file.apk>

# JAR or AAR
jadx -d <output-dir> <file.jar>
jadx -d <output-dir> <file.aar>
```

**Decompile with Fernflower/Vineflower:**

```bash
# JAR/AAR directly
java -jar fernflower.jar <file.jar> <output-dir>

# APK: first convert DEX to JAR with dex2jar, then decompile
d2j-dex2jar.sh <file.apk> -o app.jar
java -jar fernflower.jar app.jar <output-dir>
```

**XAPK files** (ZIP bundles from APKPure and similar stores): extract the archive, identify all `.apk` files inside (base + split APKs), and decompile the base APK first:

```bash
unzip <file.xapk> -d xapk-extracted/
# decompile the base APK
jadx -d <output-dir> xapk-extracted/base.apk
```

See `references/jadx-usage.md` and `references/fernflower-usage.md` for full CLI reference.

### Phase 3: Analyze Structure

Navigate the decompiled output to understand the app's architecture.

**Actions:**

1. **Read AndroidManifest.xml** from `<output-dir>/resources/AndroidManifest.xml`:
   - Identify the main launcher Activity
   - List all Activities, Services, BroadcastReceivers, ContentProviders
   - Note permissions (especially `INTERNET`, `ACCESS_NETWORK_STATE`)
   - Find the application class (`android:name` on `<application>`)

2. **Survey the package structure** under `<output-dir>/sources/`:
   - Identify the main app package and sub-packages
   - Distinguish app code from third-party libraries
   - Look for packages named `api`, `network`, `data`, `repository`, `service`, `retrofit`, `http` — these are where API calls live

3. **Identify the architecture pattern:**
   - MVP: look for `Presenter` classes
   - MVVM: look for `ViewModel` classes and `LiveData`/`StateFlow`
   - Clean Architecture: look for `domain`, `data`, `presentation` packages
   - This informs where to look for network calls in the next phases

### Phase 4: Trace Call Flows

Follow execution paths from user-facing entry points down to network calls.

**Actions:**

1. **Start from entry points**: Read the main Activity or Application class identified in Phase 3.

2. **Follow the initialization chain**: `Application.onCreate()` often sets up the HTTP client, base URL, and DI framework. Read this first.

3. **Trace user actions**: From an Activity, follow:
   - `onCreate()` → view setup → click listeners
   - Click handler → ViewModel/Presenter method
   - ViewModel → Repository → API service interface
   - API service → actual HTTP call

4. **Map DI bindings** (if Dagger/Hilt is used): Find `@Module` classes to understand which implementations are provided for which interfaces.

5. **Handle obfuscated code**: When class names are mangled, use string literals and library API calls as anchors. Retrofit annotations and URL strings are never obfuscated.

See `references/call-flow-analysis.md` for detailed techniques and grep commands.

### Phase 5: Extract and Document APIs

Find all API endpoints and produce structured documentation.

**Search for API calls:**

```bash
# Retrofit annotations
grep -r '@GET\|@POST\|@PUT\|@DELETE\|@PATCH' <output-dir>/sources/

# OkHttp calls
grep -r 'OkHttpClient\|Request\.Builder\|newCall' <output-dir>/sources/

# Hardcoded URLs
grep -rE 'https?://[a-zA-Z0-9./_-]+' <output-dir>/sources/

# Auth patterns
grep -r 'Authorization\|Bearer\|api_key\|apiKey\|token' <output-dir>/sources/

# Volley
grep -r 'StringRequest\|JsonObjectRequest\|JsonArrayRequest' <output-dir>/sources/
```

Then, for each discovered endpoint, read the surrounding source code to extract:
- HTTP method and path
- Base URL
- Path parameters, query parameters, request body
- Headers (especially authentication)
- Response type
- Where it's called from (the call chain from Phase 4)

**Document each endpoint** using this format:

```markdown
### `METHOD /path`

- **Source**: `com.example.api.ApiService` (ApiService.java:42)
- **Base URL**: `https://api.example.com/v1`
- **Path params**: `id` (String)
- **Query params**: `page` (int), `limit` (int)
- **Headers**: `Authorization: Bearer <token>`
- **Request body**: `{ "email": "string", "password": "string" }`
- **Response**: `ApiResponse<User>`
- **Called from**: `LoginActivity → LoginViewModel → UserRepository → ApiService`
```

See `references/api-extraction-patterns.md` for library-specific search patterns and the full documentation template.

## Output

At the end of the workflow, deliver:

1. **Decompiled source** in the output directory
2. **Architecture summary** — app structure, main packages, pattern used
3. **API documentation** — all discovered endpoints in the format above
4. **Call flow map** — key paths from UI to network (especially authentication and main features)

## References

- `references/setup-guide.md` — Installing Java, jadx, Fernflower/Vineflower, dex2jar, and optional tools
- `references/jadx-usage.md` — jadx CLI options and workflows
- `references/fernflower-usage.md` — Fernflower/Vineflower CLI options, when to use, APK workflow
- `references/api-extraction-patterns.md` — Library-specific search patterns and documentation template
- `references/call-flow-analysis.md` — Techniques for tracing call flows in decompiled code

## Disclaimer

This skill is provided strictly for **lawful purposes**, including but not limited to:

- Security research and authorized penetration testing
- Interoperability analysis permitted under applicable law (e.g., EU Directive 2009/24/EC, US DMCA §1201(f))
- Malware analysis and incident response
- Educational use and CTF competitions

**You are solely responsible** for ensuring that your use of this tool complies with all applicable laws, regulations, and terms of service. The authors disclaim any liability for misuse.
