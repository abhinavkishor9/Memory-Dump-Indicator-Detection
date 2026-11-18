# Investigation Notes 
## 1. Investigation Overview

The objective was to examine a synthetic memory-like binary file and identify process, command-line, network, and persistence-related indicators.

The investigation followed a static analysis approach using `strings`, `grep`, `xxd`, and `sha256sum`.

No executable content was run during the investigation.

## 2. Environment Verification

The following environment and tool versions were confirmed:

| Tool | Version |
|---|---|
| Python | 3.12.3 |
| GNU strings | GNU Binutils 2.42 |
| GNU grep | 3.11 |
| xxd | 2023-10-25 |
| SHA-256 utility | GNU coreutils 9.4 |

## 3. Sample Generation

The sample was generated using:

```text
tools/create_memory_sample.py
```

Execution:

```bash
python3 tools/create_memory_sample.py
```

Observed output:

```text
Created: samples/synthetic_memory.bin
Size: 306 bytes
```

The file was identified using:

```bash
file samples/synthetic_memory.bin
```

Observed:

```text
samples/synthetic_memory.bin: data
```

This confirms that the file was identified as generic binary data. It does not establish that the file is a genuine memory dump.

## 4. Sample Integrity

SHA-256 was calculated using:

```bash
sha256sum samples/synthetic_memory.bin
```

Observed hash:

```text
0a4acd7b96ca0cfadc670c1c42ce8e26b41f2c67f300173725889f407dbadbda
```

The hash was saved in:

```text
evidence/sample-hash.txt
```

## 5. String Extraction

Command:

```bash
strings -a samples/synthetic_memory.bin
```

The following strings were extracted:

```text
LAB28_SYNTHETIC_MEMORY_IMAGE
PROCESS: powershell.exe
COMMAND_LINE: powershell.exe -NoProfile -EncodedCommand VABlAHMAdAA=
NETWORK_URL: hxxp://update.example.invalid/payload
REMOTE_IP: 192.0.2.10
REGISTRY_REFERENCE: HKCU\Software\Microsoft\Windows\CurrentVersion\Run
PROCESS: notepad.exe
MEMORY_ANALYSIS_END
```

The output was preserved in:

```text
evidence/extracted-strings.txt
```

## 6. Indicator Investigation

### 6.1 Process-related indicators

Command:

```bash
grep -niE 'PROCESS:|powershell.exe|notepad.exe' evidence/extracted-strings.txt
```

Observed:

```text
2:PROCESS: powershell.exe
3:COMMAND_LINE: powershell.exe -NoProfile -EncodedCommand VABlAHMAdAA=
7:PROCESS: notepad.exe
```

**Finding:** The file contains references to `powershell.exe` and `notepad.exe`.

The PowerShell name also appears within the command-line string.

**Limitation:** These strings do not establish that either process was running on an endpoint.

### 6.2 Command-line indicators

Command:

```bash
grep -niE 'COMMAND_LINE:|EncodedCommand|NoProfile' evidence/extracted-strings.txt
```

Observed:

```text
3:COMMAND_LINE: powershell.exe -NoProfile -EncodedCommand VABlAHMAdAA=
```

**Finding:** A PowerShell command-line reference containing `-NoProfile` and `-EncodedCommand` was present.

**Investigation relevance:** Encoded PowerShell arguments may warrant further examination in a real investigation.

**Limitation:** No independent decoding or execution evidence was collected. The string alone does not establish malicious behaviour.

### 6.3 Network-related indicators

Command:

```bash
grep -niE 'NETWORK_URL:|REMOTE_IP:|hxxp://|192\.0\.2\.10' evidence/extracted-strings.txt
```

Observed:

```text
4:NETWORK_URL: hxxp://update.example.invalid/payload
5:REMOTE_IP: 192.0.2.10
```

**Finding:** A defanged URL and an IP address were present in the sample.

The URL uses the reserved `.invalid` domain. The IP address `192.0.2.10` belongs to the documentation range.

**Limitation:** No DNS resolution or network connection was attempted. No communication with an external host was established.

### 6.4 Persistence-related indicator

Command:

```bash
grep -niE 'REGISTRY_REFERENCE:|CurrentVersion\\Run' evidence/extracted-strings.txt
```

Observed:

```text
6:REGISTRY_REFERENCE: HKCU\Software\Microsoft\Windows\CurrentVersion\Run
```

**Finding:** The file contains a reference to the Windows Registry Run key.

**Investigation relevance:** Run keys are commonly examined during Windows persistence investigations.

**Limitation:** The extracted reference does not establish that a Registry key or value was created or modified.

## 7. Hexadecimal Inspection

Command:

```bash
xxd samples/synthetic_memory.bin | head -n 20
```

The hexadecimal output showed:

- ASCII representations of the predefined strings.
- Null-byte separators between the regions.
- The encoded PowerShell command-line string.
- The URL, IP address, and Registry reference in the binary content.

The complete output was saved in:

```text
evidence/memory-hex-dump.txt
```

## 8. Evidence Summary

| Evidence | Location | Purpose |
|---|---|---|
| Synthetic binary | `samples/synthetic_memory.bin` | Original investigation sample |
| Generator script | `tools/create_memory_sample.py` | Documents sample creation |
| SHA-256 hash | `evidence/sample-hash.txt` | Sample integrity reference |
| Extracted strings | `evidence/extracted-strings.txt` | Printable indicator evidence |
| Hexadecimal dump | `evidence/memory-hex-dump.txt` | Binary-level inspection |

## 9. Findings Classification

### Confirmed observations

- A 306-byte synthetic binary file was generated.
- Its SHA-256 hash was recorded.
- Eight predefined strings were extracted.
- Process and command-line references were identified.
- A defanged URL and documentation-range IP address were identified.
- A Windows Registry Run key reference was identified.
- Hexadecimal inspection showed the expected byte content and null separators.

### Investigation leads

- The PowerShell command-line reference would warrant further examination in a real memory investigation.
- The Registry Run key reference could be correlated with actual Registry evidence.
- Process and network indicators could be compared with endpoint telemetry if genuine evidence were available.

### Not established

- Actual PowerShell execution.
- Running or hidden processes.
- Successful network communication.
- Registry modification or persistence.
- Process injection or malicious code execution.
- Malware presence or endpoint compromise.

