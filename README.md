# APK Security Analyzer

A lightweight, fully offline web tool for static security analysis of Android APK files. Upload an APK, get a structured risk report in your browser. Nothing leaves your machine.

![Python](https://img.shields.io/badge/python-3.9%2B-blue)
![Flask](https://img.shields.io/badge/flask-3.0%2B-green)
![License](https://img.shields.io/badge/license-MIT-blue)

---

## What It Does

The tool runs five analysis modules against any APK you upload:

| Module | What It Checks |
|--------|----------------|
| **Manifest** | Package metadata, SDK versions, exported components, deep link schemes, security flags (`debuggable`, `allowBackup`, `usesCleartextTraffic`) |
| **Permissions** | Classifies all declared permissions by risk level with contextual notes |
| **Secrets** | Scans text resources for hardcoded API keys, staging URLs, tokens, private keys, internal IPs |
| **SDK Fingerprinting** | Identifies 30+ known third-party SDKs from DEX class namespaces — analytics, tracking, payments, security controls |
| **Network Security Config** | Parses `network_security_config.xml` for certificate pinning, expiry dates, cleartext policy, user CA trust |

Results are presented as a scored, graded report (A–F) with a risk matrix broken down by Critical / High / Medium / Low.

---

## Screenshot

```
┌─────────────────────────────────────────────────┐
│  📱 com.example.app  v4.2.1  |  Grade: C        │
├──────────┬──────────┬──────────┬────────────────┤
│ CRITICAL │   HIGH   │  MEDIUM  │   LOW / INFO   │
│    1     │    4     │    7     │       2        │
├──────────┴──────────┴──────────┴────────────────┤
│ 📋 Manifest  🌐 NSC  🔑 Secrets  📦 SDKs  🔒 Crypto │
└─────────────────────────────────────────────────┘
```

---

## Quick Start

```bash
# Clone
git clone https://github.com/yourusername/apk-security-analyzer
cd apk-security-analyzer

# Set up virtual environment
python3 -m venv venv
source venv/bin/activate        # Linux / Mac
# venv\Scripts\activate         # Windows

# Install dependencies
pip install -r requirements.txt

# Run
python3 app.py
```

Open **http://localhost:5000**, upload any `.apk` file, get your report.

---

## Requirements

- Python 3.9+
- No internet connection required after install
- No database, no auth, no external services

Dependencies (installed via pip):

```
flask>=3.0.0
androguard>=4.1.3
pyaxmlparser>=0.3.24
werkzeug>=3.0.0
```

---

## How It Works

### DEX Class Extraction

The tool uses [androguard](https://github.com/androguard/androguard) to parse the compiled DEX bytecode inside the APK and extract class names directly. This is more reliable than smali-level file matching and works on standard Play Store APKs without needing JADX or apktool.

```python
from androguard.misc import AnalyzeAPK
a, d, dx = AnalyzeAPK(apk_path)
classes = set(c.name.lstrip('L').replace(';','').replace('.','/')
              for c in dx.get_classes())
```

### SDK Fingerprinting

Class namespace prefixes are matched against a curated map of 30+ known SDKs. Each match includes a risk classification and specific security note:

```
com/dynatrace  →  Dynatrace [SESSION REPLAY] HIGH
                  Full session replay capable — verify DataCollectionLevel=PERFORMANCE
```

### Secret Scanning

Regex patterns scan all text resources (XML, JSON, properties, JS, YAML) inside the APK ZIP for common secret formats — AWS keys, Firebase keys, staging URLs, hardcoded passwords, private key headers, and more.

### Network Security Config

The tool parses `network_security_config.xml` and checks:
- Is cleartext traffic disabled?
- Are user-installed CA certificates trusted? (trivial MITM risk)
- Is a `pin-set` present?
- If so, has the `expiration` date lapsed? (silently disables pinning)

---

## Analysis Scope & Limitations

This tool performs **static analysis only**:

- No servers are probed
- No network traffic is intercepted  
- No credentials are used
- APK is deleted immediately after analysis completes

What static analysis **cannot** tell you:
- Whether crypto is implemented correctly at the method level (use JADX for that)
- Runtime behaviour (use Frida + Burp for that)
- Whether certificate pins are real SHA-256 hashes vs wildcards

For deeper analysis after running this tool:

```bash
# Decompile to Java source
jadx -d src/ app.apk

# Key follow-up greps
grep -r "onReceivedSslError" src/ --include="*.java" -A 5
grep -r "DataCollectionLevel" src/ --include="*.java"
grep -r "MobileCore.trackAction" src/ --include="*.java" -A 3
grep -r "MD5\|getInstance.*MD5" src/ --include="*.java"
```

---

## Project Structure

```
apk-security-analyzer/
├── app.py                  # Flask app + all analysis modules
├── requirements.txt
├── .gitignore
├── README.md
├── uploads/                # Temp storage — APKs deleted after analysis
│   └── .gitkeep
└── templates/
    ├── index.html          # Upload UI
    └── report.html         # Analysis report
```

---

## Only Analyze APKs You're Authorized to Inspect

This tool is intended for:
- Security professionals assessing apps they are authorized to test
- Developers auditing their own applications
- Researchers analyzing publicly distributed APKs

Static analysis of publicly distributed APKs (downloaded from Google Play Store or APKMirror) is legal in most jurisdictions. Internal enterprise apps, apps behind NDAs, or apps you are not an authorized user of may have different considerations — check your organization's acceptable use policy.

---

## Extending the Tool

### Add a new SDK to fingerprint

In `fingerprint_sdks()`, add an entry to `SDK_MAP`:

```python
'com/newcompany/sdk': ('SDK Name', 'CATEGORY', 'HIGH', 'Risk description'),
```

### Add a new secret pattern

In `scan_secrets()`, add an entry to `PATTERNS`:

```python
'My Secret Type': (r'regex_pattern_here', 'HIGH'),
```

### Add a new crypto signal

In `analyze_crypto()`, add to `GOOD_SIGNALS` or `WARN_SIGNALS`:

```python
'ClassName': 'Description of what it means',
```

---

## Background

This tool was built as part of a **100 Vibe Coding Projects** challenge — shipping one focused, personally useful project per week.

The analysis methodology was developed through hands-on teardown of real-world APKs, documented at [mrdee.in](https://mrdee.in). The five-gate framework (Manifest → Secrets → SDKs → Crypto → Network Security) mirrors the approach used in published mobile security research.

---

## License

MIT License — use freely, attribution appreciated.
