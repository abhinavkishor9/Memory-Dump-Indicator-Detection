# Lab 28 – Detect Indicators in Memory Dumps

## Overview

This lab demonstrates basic static indicator extraction from a synthetic memory-like binary artefact.

The exercise simulates an initial SOC/DFIR triage scenario in which an analyst receives a binary file and needs to identify potentially relevant process, command-line, network, and persistence-related indicators.

The investigation uses standard Linux utilities to extract printable strings, search for selected indicators, inspect hexadecimal content, and preserve sample integrity using SHA-256.

## Lab Environment

| Component | Details |
|---|---|
| Operating System | Ubuntu 24.04 WSL |
| Python | 3.12.3 |
| GNU strings | GNU Binutils 2.42 |
| GNU grep | 3.11 |
| xxd | 2023-10-25 |
| SHA-256 utility | GNU coreutils 9.4 |

## Investigation Objectives

- Generate a controlled synthetic binary artefact.
- Extract printable strings from the binary.
- Identify process and command-line references.
- Search for network-related indicators.
- Identify persistence-related Registry references.
- Inspect the binary in hexadecimal format.
- Calculate and preserve the SHA-256 hash.
- Interpret the extracted indicators without assuming malicious activity.

## Directory Structure

```text
lab28/
├── evidence/
│   ├── extracted-strings.txt
│   ├── memory-hex-dump.txt
│   └── sample-hash.txt
├── output/
├── samples/
│   └── synthetic_memory.bin
└── tools/
    └── create_memory_sample.py
```

## Sample Information

**Sample:** `synthetic_memory.bin`

**File size:** 306 bytes

**File type:** Generic binary data

**SHA-256:**

```text
0a4acd7b96ca0cfadc670c1c42ce8e26b41f2c67f300173725889f407dbadbda
```

The sample was generated using Python and contains predefined strings separated by null bytes.

## Indicators Identified

| Category | Extracted indicator |
|---|---|
| Process | `powershell.exe` |
| Process | `notepad.exe` |
| Command line | `powershell.exe -NoProfile -EncodedCommand VABlAHMAdAA=` |
| URL | `hxxp://update.example.invalid/payload` |
| IP address | `192.0.2.10` |
| Registry reference | `HKCU\Software\Microsoft\Windows\CurrentVersion\Run` |

## Analysis Methodology

### 1. Sample generation

A Python script generated the synthetic binary file:

```bash
python3 tools/create_memory_sample.py
```

### 2. Integrity verification

The SHA-256 hash was calculated and saved:

```bash
sha256sum samples/synthetic_memory.bin > evidence/sample-hash.txt
```

### 3. String extraction

Printable strings were extracted using:

```bash
strings -a samples/synthetic_memory.bin > evidence/extracted-strings.txt
```

### 4. Indicator searches

The extracted strings were searched for:

- Process names and process references.
- PowerShell command-line arguments.
- URL and IP address references.
- Registry persistence-related references.

### 5. Hexadecimal inspection

The binary content was examined and preserved:

```bash
xxd samples/synthetic_memory.bin > evidence/memory-hex-dump.txt
```

## Key Findings

The analysis confirmed that the synthetic binary contained all predefined indicator strings.

The extracted content included a PowerShell command-line reference, two process names, a defanged URL, a documentation-range IP address, and a Windows Registry Run key reference.

The hexadecimal inspection also showed null-byte separators between the predefined regions.

## Important Investigation Limitations

This exercise used a synthetic memory-like binary artefact rather than a genuine RAM acquisition.

The presence of a process name does not prove that the process was running. Similarly, a command-line string does not prove command execution, and a Registry reference does not establish that a Registry key or value was modified.

The URL is defanged and uses the reserved `.invalid` domain. The IP address belongs to the documentation range. No network communication was performed.

No process injection, persistence activity, malware execution, or endpoint compromise was established.

## Learning Outcomes

- Practised extracting printable strings from binary data.
- Used grep to identify investigation-relevant indicators.
- Understood the value of SHA-256 for sample integrity.
- Examined binary content using hexadecimal representation.
- Learned to distinguish an extracted artefact from proof of system activity.
- Understood the limitations of basic static memory-like file analysis.

## Conclusion

The lab successfully demonstrated a basic indicator-extraction workflow using a controlled synthetic binary artefact.

The extracted strings provided several investigation leads, but they were not sufficient to establish malicious activity or compromise.

A genuine memory-forensics investigation would require an actual memory acquisition and structured analysis using appropriate forensic tools.
