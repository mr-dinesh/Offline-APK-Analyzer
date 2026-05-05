# APK Security Analyzer

A local web tool for static analysis of Android APK files.

## What It Analyzes

| Module | What It Checks |
|--------|----------------|
| Manifest | Package info, SDK versions, security flags (debuggable, allowBackup, cleartext) |
| Permissions | Dangerous permission classification with risk context |
| Secrets | Hardcoded API keys, staging URLs, tokens, private keys |
| SDKs | Third-party SDK fingerprinting — 30+ known SDKs |
| Crypto | Crypto controls (KeyStore, SQLCipher, BiometricPrompt) and WebView warnings |
| Network Security | Certificate pinning, cleartext policy, user CA trust |

## Setup

```bash
# 1. Clone / copy this folder to your machine

# 2. Create virtual environment
python3 -m venv venv
source venv/bin/activate        # Linux/Mac
# venv\Scripts\activate         # Windows

# 3. Install dependencies
pip install -r requirements.txt

# 4. Run
python app.py
```

Open http://localhost:5000 in your browser.

## Usage

1. Open http://localhost:5000
2. Upload any .apk file (up to 150MB)
3. Analysis runs locally — no data leaves your machine
4. APK is deleted immediately after analysis

## Limitations

- Class-level SDK detection only (no method-level analysis)
- Secret scanning covers text resources only — DEX bytecode requires JADX
- Binary AXML manifest parsing may fall back to regex on some APKs
- For deep analysis, follow up with: `jadx -d src/ app.apk`

## Next Steps After Report

```bash
# Deep decompile for method-level findings
jadx -d wf_src/ app.apk

# Key greps after decompile
grep -r "onReceivedSslError" wf_src/ --include="*.java" -A 5
grep -r "DataCollectionLevel" wf_src/ --include="*.java"
grep -r "trackAction" wf_src/ --include="*.java" -A 3
grep -r "MD5\|getInstance.*MD5" wf_src/ --include="*.java"
```

## Project

Part of the 100 Vibe Coding Projects series — mrdee.in
