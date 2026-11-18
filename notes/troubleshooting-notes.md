# Troubleshooting Notes – Lab 28: Detect Indicators in Memory Dumps

## 1. Workspace Creation

### Command

```bash
mkdir -p ~/malware-yara-labs/lab28/{samples,output,evidence,tools}
```

### Verification

```bash
find . -maxdepth 2 -type d | sort
```

### Observed

```text
.
./evidence
./output
./samples
./tools
```

### Result

All required directories were created successfully. No workspace creation errors occurred.

---

## 2. Tool Verification

### Commands

```bash
python3 --version
strings --version | head -n 1
grep --version | head -n 1
xxd -v
sha256sum --version | head -n 1
```

### Observed versions

```text
Python 3.12.3
GNU strings (GNU Binutils for Ubuntu) 2.42
grep (GNU grep) 3.11
xxd 2023-10-25 by Juergen Weigert et al.
sha256sum (GNU coreutils) 9.4
```

### Result

All required tools were available.

No additional package installation was necessary.

---

## 3. Synthetic Sample Identification

### Command

```bash
file samples/synthetic_memory.bin
```

### Observed

```text
samples/synthetic_memory.bin: data
```

### Explanation

The `file` utility identified the sample as generic binary data.

This is expected because the file was generated as a controlled binary artefact containing predefined strings separated by null bytes.

It is not a genuine memory acquisition.

---

## 4. String Extraction

### Command

```bash
strings -a samples/synthetic_memory.bin
```

### Result

All eight predefined strings were extracted successfully.

The output included:

- Synthetic memory image marker.
- PowerShell process reference.
- PowerShell command-line reference.
- Defanged URL.
- Documentation-range IP address.
- Registry Run key reference.
- Notepad process reference.
- End-of-analysis marker.

### Explanation

The `-a` option scans the complete file for printable strings.

The successful extraction confirms that the intended ASCII content was present in the generated binary.

---

## 5. Indicator Search

### Observation

The grep searches returned the expected matching lines for process, command-line, network, and persistence indicators.

### Explanation

The searches were performed against the extracted strings file rather than repeatedly scanning the binary.

This separates extraction from indicator searching and makes the investigation easier to review.

No grep errors were observed.

---

## 6. Hexadecimal Inspection

### Command

```bash
xxd samples/synthetic_memory.bin | head -n 20
```

### Observation

The output showed the ASCII content represented in hexadecimal form.

Null bytes (`00`) were visible between the predefined string regions.

### Explanation

The hexadecimal view provides a byte-level representation of the sample and helps confirm how the synthetic regions were assembled.

The complete hexadecimal output was preserved separately.

---

## 7. Evidence Preservation

The following evidence files were created:

```text
evidence/
├── extracted-strings.txt
├── memory-hex-dump.txt
└── sample-hash.txt
```

The original binary and its generator script were also retained.

The SHA-256 hash provides an integrity reference for the sample used during this exercise.

---

## Troubleshooting Summary

| Area | Observation | Resolution |
|---|---|---|
| Workspace | Required directories created | No action required |
| Tool availability | All tools available | No installation required |
| File identification | Reported as generic data | Expected for synthetic binary |
| String extraction | All intended strings extracted | Successful |
| Indicator searches | Expected matches returned | Successful |
| Hexadecimal inspection | Binary content displayed | Successful |
| Evidence preservation | Hash, strings, and hex output saved | Successful |

## Key Learning

Static analysis of binary data requires careful interpretation.

Extracted strings can provide useful investigation leads, but they do not independently establish that the associated processes, network connections, or persistence mechanisms were active.

The type and origin of the evidence must always be considered before drawing conclusions.
