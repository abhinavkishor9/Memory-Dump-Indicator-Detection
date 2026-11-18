# Timeline 
## Investigation Activity Timeline

### 1. Workspace Preparation

- Created the Lab 28 directory.
- Prepared separate directories for samples, evidence, tools, and output.
- Verified the directory structure.

### 2. Tool Verification

- Confirmed Python 3.12.3.
- Confirmed GNU strings 2.42.
- Confirmed GNU grep 3.11.
- Verified xxd and SHA-256 utilities.
- No additional tools were installed.

### 3. Synthetic Sample Generation

- Created `tools/create_memory_sample.py`.
- Generated `samples/synthetic_memory.bin`.
- Confirmed the generated file size was 306 bytes.
- Identified the file as generic binary data.

### 4. Integrity Verification

- Calculated the sample's SHA-256 hash.
- Recorded the hash in `evidence/sample-hash.txt`.

### 5. String Extraction

- Extracted printable strings using `strings -a`.
- Saved the output in `evidence/extracted-strings.txt`.
- Confirmed that all eight predefined strings were extracted.

### 6. Indicator Investigation

- Searched for process references.
- Identified the PowerShell command-line string.
- Identified the defanged URL and documentation-range IP address.
- Identified the Windows Registry Run key reference.
- Reviewed the matching line numbers.

### 7. Hexadecimal Inspection

- Inspected the binary using `xxd`.
- Observed the ASCII content and null-byte separators.
- Saved the complete hexadecimal dump in `evidence/memory-hex-dump.txt`.

### 8. Evidence Review

- Reviewed the extracted strings.
- Reviewed the indicator search results.
- Preserved the sample hash and hexadecimal output.
- Classified observations, investigative leads, and limitations.

