# Android Reverse Engineering — Gemini Skill

An [open-standard agent skill](https://agentskills.io/home) for **Gemini CLI** and **Android Studio** that decompiles Android APK/XAPK/JAR/AAR files and extracts the HTTP APIs used by the app — Retrofit endpoints, OkHttp calls, hardcoded URLs, authentication patterns — producing structured documentation ready for further analysis.

> **Ported from** [SimoneAvogadro/android-reverse-engineering-skill](https://github.com/SimoneAvogadro/android-reverse-engineering-skill) — a Claude Code plugin by [Simone Avogadro](https://github.com/SimoneAvogadro).  
> Original work licensed under [Apache 2.0](https://github.com/SimoneAvogadro/android-reverse-engineering-skill/blob/master/LICENSE).

## What it does

- **Decompiles** APK, XAPK, JAR, and AAR files using jadx and Fernflower/Vineflower
- **Extracts and documents APIs**: Retrofit endpoints, OkHttp calls, hardcoded URLs, auth headers and tokens
- **Traces call flows** from Activities/Fragments through ViewModels and repositories down to HTTP calls
- **Analyzes** app structure: manifest, packages, architecture patterns
- **Handles obfuscated code**: strategies for navigating ProGuard/R8 output

## Requirements

**Required:**
- Java JDK 17+
- [jadx](https://github.com/skylot/jadx) (CLI)

**Optional (recommended):**
- [Vineflower](https://github.com/Vineflower/vineflower) or [Fernflower](https://github.com/JetBrains/fernflower) — better output on complex Java code
- [dex2jar](https://github.com/pxb1988/dex2jar) — needed to use Fernflower on APK/DEX files

See `android-reverse-engineering/references/setup-guide.md` for detailed installation instructions.

## Installation

This skill follows the [open-standard agent skills](https://agentskills.io/home) format (`SKILL.md`).

### Gemini CLI

Copy the skill to your Gemini skills directory:

```bash
# Clone the repo
git clone https://github.com/Pizza67/android-reverse-engineering-gemini-skill.git

# Copy the skill to Gemini's skill directory
cp -r android-reverse-engineering-gemini-skill/android-reverse-engineering ~/.gemini/skills/
```

Or manually place the `android-reverse-engineering/` folder into:
- **Gemini CLI**: `~/.gemini/skills/`
- **Project-level**: `.gemini/skills/` in your project root

### Android Studio

See the [Android Studio skills documentation](https://developer.android.com/studio/gemini/skills) for how to load custom skills.

## Usage

Once installed, the skill activates automatically on natural language prompts such as:

- "Decompile this APK"
- "Reverse engineer this Android app"
- "Extract API endpoints from this app"
- "Follow the call flow from LoginActivity"
- "Analyze this AAR library"

## Repository Structure

```
android-reverse-engineering-gemini-skill/
├── android-reverse-engineering/
│   ├── SKILL.md                        # Core skill (open-standard format)
│   └── references/
│       ├── setup-guide.md
│       ├── jadx-usage.md
│       ├── fernflower-usage.md
│       ├── api-extraction-patterns.md
│       └── call-flow-analysis.md
└── README.md
```

## References

- [jadx — Dex to Java decompiler](https://github.com/skylot/jadx)
- [Fernflower — JetBrains analytical decompiler](https://github.com/JetBrains/fernflower)
- [Vineflower — Fernflower community fork](https://github.com/Vineflower/vineflower)
- [dex2jar — DEX to JAR converter](https://github.com/pxb1988/dex2jar)
- [Open-standard agent skills](https://agentskills.io/home)
- [Android skills documentation](https://developer.android.com/tools/agents/android-skills)

## Disclaimer

This skill is provided strictly for **lawful purposes**, including but not limited to:

- Security research and authorized penetration testing
- Interoperability analysis permitted under applicable law
- Malware analysis and incident response
- Educational use and CTF competitions

**You are solely responsible** for ensuring that your use of this tool complies with all applicable laws, regulations, and terms of service. The authors disclaim any liability for misuse.

## License

Apache 2.0 — see [LICENSE](LICENSE)
