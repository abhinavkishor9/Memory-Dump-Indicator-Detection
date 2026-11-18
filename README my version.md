# Memory-Dump-Indicator-Detection
## Overview
### 1. Introduction to Memory Forensics

Memory forensics is the examination of data acquired from a computer's volatile memory (RAM).

RAM contains information associated with the operating system and running applications. Unlike traditional disk artefacts, memory can preserve evidence of activity that may not be available through normal file-system inspection.

Memory analysis is useful during incident response because it can help investigators understand what was happening on a system at the time of acquisition.

### 2. Why Analyse Memory?

A memory investigation may help identify:

- Running and terminated processes.
- Process command-line arguments.
- Loaded DLLs and modules.
- Network connections and related structures.
- Suspicious memory regions.
- Code injection indicators.
- Other volatile system artefacts.

These findings can help reconstruct activity that may otherwise be difficult to investigate.

### 3. What Are Indicators?

Indicators are identifiable artefacts that may provide useful information during an investigation.

Examples include:

| Indicator | Investigation relevance |
|---|---|
| Process name | Helps identify processes requiring review |
| Command-line argument | Provides context about process execution |
| URL | May identify a referenced network resource |
| IP address | May help identify network-related activity |
| Registry path | May indicate a persistence-related configuration reference |
| DLL name | May identify a module requiring further analysis |

An indicator should be evaluated in context. Its presence alone does not prove malicious activity.

### 4. String Extraction

Binary files may contain readable text embedded among non-printable bytes.

The `strings` utility extracts sequences of printable characters.

Example:

```bash
strings -a samples/synthetic_memory.bin
```

The `-a` option scans the complete file.

String extraction is useful for initial triage, but it does not reconstruct the complete structure of a memory image.

### 5. Searching for Indicators

After extracting strings, `grep` can be used to search for specific keywords or patterns.

Example:

```bash
grep -niE 'powershell.exe|notepad.exe' evidence/extracted-strings.txt
```

Useful search categories include:

- Process names.
- Suspicious command-line arguments.
- URLs and IP addresses.
- Registry references.
- DLL names and other relevant artefacts.

Searching extracted strings is a basic triage technique, not a substitute for structured forensic analysis.

### 6. PowerShell Command-Line Analysis

PowerShell supports command-line options such as:

```text
-NoProfile
-EncodedCommand
```

These options can be relevant during investigations.

However, encoded commands and unusual arguments are not automatically malicious.

An analyst should examine the command content, execution context, parent process, user account, timestamps, and supporting endpoint telemetry before reaching a conclusion.

### 7. Network Indicator Analysis

Memory-related artefacts may contain network information such as:

- IP addresses.
- URLs.
- Hostnames.
- Connection-related structures.

These indicators may help identify communication with external systems.

However, finding an IP address or URL as a string does not establish that a connection occurred.

The analyst should correlate such indicators with network logs, process information, and other forensic evidence.

### 8. Persistence-Related References

Windows persistence investigations often examine locations such as:

```text
HKCU\Software\Microsoft\Windows\CurrentVersion\Run
```

A reference to this Registry location may be relevant during triage.

However, a string containing a Registry path does not establish that a key or value exists or was modified.

Actual Registry evidence and supporting system artefacts are required.

### 9. Hexadecimal Analysis

The `xxd` utility displays binary content in hexadecimal and ASCII formats.

Example:

```bash
xxd samples/synthetic_memory.bin
```

Hexadecimal inspection helps analysts:

- Examine raw byte sequences.
- Identify readable content.
- Observe null bytes and separators.
- Compare byte-level content.
- Understand how a binary artefact is structured.

### 10. SHA-256 Integrity Verification

SHA-256 produces a cryptographic hash that can be used as an integrity reference.

Example:

```bash
sha256sum samples/synthetic_memory.bin
```

The hash can be recorded before and after analysis to help verify that the original sample has not changed.

### 11. Structured Memory Analysis

Basic string extraction is only an initial step.

For genuine Windows memory images, tools such as Volatility 3 can provide structured analysis.

Examples of relevant Volatility 3 plugins include:

- `windows.pslist` – Lists process information.
- `windows.psscan` – Scans for process structures.
- `windows.cmdline` – Retrieves process command-line information.
- `windows.netscan` – Examines network-related structures.
- `windows.dlllist` – Lists loaded DLLs.

These tools require a suitable memory image and appropriate forensic context.

### 12. Lab Application

This lab used a synthetic 306-byte binary artefact containing predefined strings.

The analysis identified:

- PowerShell and Notepad process references.
- A PowerShell command-line reference.
- A defanged URL.
- A documentation-range IP address.
- A Windows Registry Run key reference.

The exercise demonstrated how to extract, search, preserve, and interpret indicators.

### 13. Important Limitations

The lab did not use a genuine memory acquisition.

It did not establish:

- Actual running processes.
- Command execution.
- Network communication.
- Registry modification.
- Process injection.
- Malware execution or endpoint compromise.

The central forensic principle is to distinguish an artefact found in a file from an event proven to have occurred on a system.

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

## Lab Objectives

### Primary Objective

To practise an initial static triage workflow for identifying investigation-relevant indicators within binary data and preserving the supporting evidence.

### Specific Objectives

1. **Prepare the investigation environment:** Create a structured workspace for samples, tools, output, and evidence.

2. **Verify analysis utilities:** Confirm the availability and versions of Python, strings, grep, xxd, and SHA-256 utilities.

3. **Generate a controlled artefact:** Create a synthetic binary file containing predefined process, command-line, network, and persistence-related strings.

4. **Establish sample integrity:** Calculate and preserve the sample's SHA-256 hash.

5. **Extract printable content:** Use string extraction techniques to identify readable information embedded in binary data.

6. **Identify process references:** Search for process names that may be relevant during endpoint investigations.

7. **Examine command-line indicators:** Identify potentially relevant PowerShell arguments and understand why execution context is important.

8. **Identify network references:** Search for URL and IP address strings and distinguish references from confirmed communication.

9. **Identify persistence references:** Locate Registry-related strings and understand the difference between a reference and an actual configuration change.

10. **Perform hexadecimal inspection:** Examine the binary content to understand its byte representation.

11. **Preserve investigation evidence:** Store the extracted strings, sample hash, and hexadecimal dump for later review.

12. **Apply evidence-based interpretation:** Classify confirmed observations, investigative leads, and unknown activity without making unsupported conclusions.

### Expected Learning Outcomes

After completing this lab, the learner should be able to:

- Explain the purpose of memory forensics.
- Perform basic string extraction from binary data.
- Search extracted content for investigation indicators.
- Identify the limitations of static string analysis.
- Preserve sample integrity using SHA-256.
- Examine binary data using hexadecimal representation.
- Explain why genuine memory acquisition and structured forensic tools are necessary for full memory analysis.

## Lab Scenario

### Scenario

A SOC analyst receives a binary artefact during an investigation involving potentially suspicious PowerShell activity.

The available evidence is limited. There is no original endpoint access, no confirmed memory acquisition metadata, and no supporting process or network telemetry.

The analyst must perform an initial examination of the binary file to determine whether it contains indicators that may help guide further investigation.

The focus is on identifying process references, command-line arguments, network-related strings, and persistence-related references.

### Investigation Approach

The analyst first verifies the available analysis tools and prepares a dedicated workspace.

A controlled synthetic memory-like binary is generated to represent a small collection of predefined investigation artefacts.

The analyst then calculates the sample's SHA-256 hash and extracts printable strings using the `strings` utility.

The extracted content is searched using `grep` to identify:

- Process names such as `powershell.exe` and `notepad.exe`.
- PowerShell command-line arguments.
- A defanged URL and an IP address.
- A Windows Registry Run key reference.

The analyst also examines the hexadecimal representation of the binary to understand how the predefined string regions are stored.

### Investigation Questions

1. Which process names are present in the binary?
2. Does the artefact contain a PowerShell command-line reference?
3. Are there any URL or IP address strings?
4. Does the binary contain Registry-related references?
5. Can the extracted strings be correlated with the raw hexadecimal content?
6. Has the original sample been preserved with a recorded SHA-256 hash?
7. Which findings are confirmed, and which require additional evidence?

### Expected Investigation Outcome

The analyst should successfully extract the predefined strings and identify the process, command-line, network, and persistence-related references.

The extracted indicators should be preserved with the sample hash and hexadecimal dump.

The analyst must also document the limitations of the available evidence.

### Scope and Limitations

This exercise uses a synthetic binary artefact rather than a genuine RAM acquisition.

The presence of a process name does not prove that the process was running. A command-line string does not prove execution, a network indicator does not prove communication, and a Registry reference does not prove persistence.

The objective is initial indicator triage and evidence interpretation, not a malware verdict or complete memory-forensics investigation.

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

