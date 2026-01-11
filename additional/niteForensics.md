# niteCTF Forensics challs (understanding from official writeups)

## Google ADSense

### Description
Who looked at our $11.48 monthly AdSense revenue and decided we needed to hire people to boost it?

### Solve
After extracting the OLE document from the Alternate Data Stream of ` oogleAdsSpecialistResume.pdf`, analysis with `olevba` revealed multiple `VBA` macro modules. Here, most of it was decoy or junk code. The malicious logic was in `Module6`, which was heavily obfuscated using XOR-based string obfuscation. All sensitive strings (URLs, file paths, ADS names, and cryptographic material) were stored as arrays of integers and dynamically reconstructed at runtime using bitwise XOR operations, preventing static analysis tools such as olevba from automatically recovering them. The strings had to be manually decoded by identifying the XOR key and reproducing the logic via Python.The macro enumerates specific PDF files, reads their NTFS Alternate Data Streams, concatenates the extracted data in a predefined order, retrieves an AES key from another ADS, and attempts to decrypt the resulting ciphertext to obtain the next-stage payload.

### Flag:
**Flag:**`nite{1n_th1s_ultr4_4w3s0m3_p3rf3ct_w0rld_w1ll_th3r3_st1ll_b3_ADS_4nd_UAC_BYPASS?}`



## Incident Response 1: My Clematis

### Description
Mizi wants to create a love letter for her girlfriend's birthday. Since she doesn't know how to program, she used AI to vibecode it using a popular protocol. Unfortunately she failed to secure it properly and an attacker gained access to the system through it. Your first task is to find out the following:

1. The CVE used to exploit her system.
2. The full ID of the malicious commit
3. The name of the malicious file introduced in the commit.

Flag format: nite{CVE-XXXX-YYYY_<commit_id>_<malicious_file>}

Drive Link: https://drive.google.com/file/d/18CDQsDXyU-43Vwzjjk4P5RVWHLIhJ2rY/view?usp=sharing

Password to the archive: 5804e0b9d4b522e17d39453b662d80eda606

Password to the VM: 12345678

### Solve
We first boot the provided vm and enumerate the user directories. There are multiple personal project folders under `C:\Users\Mizi`. In `C:\Users\Mizi\Music`, we find a weird Git repository named `WorldCollapsing`. Examining the Git history using `git log` reveals that nearly all commits were authored by Mizi, except for a single earlier commit authored by another user: Luka. This commit (`c0df0ebeb988e991418029e3021fb7f8542068b2`) has a commit message `add images :3`. We analyze the repository contents to find a `.cursor/mcp.json` configuration file, which registers an MCP server configured to execute a PowerShell process at startup using `-ExecutionPolicy Bypass` and pointing to a script disguised as an image file. MCP configurations are implicitly trusted and executed automatically by the Cursor IDE. Opening `31.jpg.ps1` shows that it contains a triple-layer Base64-encoded payload, which self-decodes in a loop and executes dynamically via Invoke-Expression. Inspecting using `git log images/31.jpg.ps1` confirms that it was introduced exclusively in Luka’s commit. The attack uses MCPoison, corresponding to CVE-2025-54135 / CVE-2025-54136. 

### Flag
**Flag:** `nite{CVE-2025-54135/6_c0df0ebeb988e991418029e3021fb7f8542068b2_31.jpg.ps1}`



## Incident Response 2: Wiege

### Description:
The malicious command downloaded and executed a binary. Find out what it did.

Handout is the same as the one used to solve My Clematis.

### Solve:
The script retrieves a Base64-encoded URL, decodes it at runtime, and downloads a password-protected 7-Zip archive `hmm.7z` from a GitHub repository. The archive is extracted using the hardcoded password `hyuluvhyuluvhyu` into a temporary directory under the user’s Downloads folder. Then a contained executable `hash_encoder.exe` is launched in hidden mode. Immediately after execution, the temporary directory and archive are deleted. We analyze `hash_encoder.exe` to reveal a Rust-compiled Windows PE binary. The program coordinates multiple payload-handling stages. The binary downloads two additional resources from GitHub: a secondary executable `caret-ware.exe` and an encoded text payload `part2.txt`. The text payload goes through layered decoding, first via Base64, followed by XOR decryption using the static key ETIN, and is validated as UTF-8 data. Instead of writing this decrypted data directly to disk, the malware implements a custom steganographic storage mechanism using the filesystem itself. It then creates a directory structure under `%APPDATA%\Microsoft\Windows\Start Menu\Programs\Startup`, selecting a randomly named subdirectory from a predefined list of Windows component names, and saving the downloaded executable as `chrome.exe`. The decrypted text payload is encoded using a base-4 scheme: each plaintext byte is split into four 2-bit values, and each value is represented by a file whose name and size are randomly generated until its hash falls into one of four predefined hash ranges. The hash function is a custom rolling hash operating over file metadata, reduced to a 24-bit space and partitioned into four equal ranges, with the top two bits of the hash determining the encoded digit. During reconstruction, the directory tree is traversed, the hash ranges are mapped back to digits in reverse order, and each group of four digits is recombined into the original byte. Applying the inverse XOR operation with the same key fully restores the original UTF-8 payload, which decodes to the final flag string. This stage demonstrates a deliberate use of filesystem-based encoding and nontraditional data exfiltration techniques to evade detection and frustrate conventional static analysis.

## Flag:
**Flag:** `nite{1nsp1r3d_by_he_H3_xD_xqvylj8rtap8}`


