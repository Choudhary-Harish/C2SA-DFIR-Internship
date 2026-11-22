# Week 1–2 Digital Forensics Acquisition Report

**Case ID:** DFIR-2026-001  
**Investigator:** Harish Choudhary  
**Environment:** Kali Linux  
**Purpose:** Educational DFIR training  
**Report date:** 22 November 2026

## 1. Case Overview

This report documents examination of two self-created virtual disk images. The EXT4 image was used for acquisition verification and file-system inspection. The NTFS image was used for MFT analysis, deleted-file recovery, Autopsy examination and ExifTool metadata extraction.

These are synthetic laboratory images, not evidence acquired from a third-party computer.

| Evidence ID | Acquired image | File system | Size | SHA-256 |
|---|---|---|---:|---|
| EVD-001 | `evidence_working.dd` | EXT4 | 256 MiB | `16a3adf89f254f91567c4677a3d937d01e9c107ff84673f2996247848e6623be` |
| EVD-002 | `ntfs_evidence.dd` | NTFS | 256 MiB | `53779d8ba0a946bd933915b72f2b68de215b1cf329ac8681571d319ccb85e88e` |

## 2. Acquisition and Chain of Custody

### EVD-001 — EXT4

- Source device recorded in acquisition notes: `/dev/loop0`
- Acquired image: `evidence_working.dd`
- Acquisition date/time recorded: 18 November 2026, 16:35:35 IST
- Tool: `dc3dd` 7.2.646
- Size: 256 MiB
- Bad sectors: 0
- MD5: `af0dd2387b6c604bace59276fa785401`
- SHA-256: `16a3adf89f254f91567c4677a3d937d01e9c107ff84673f2996247848e6623be`

A later verification addendum identifies `test_disk.img` as the candidate source image and records matching hashes. However, the historical mapping of `/dev/loop0` to that file was not independently recorded at acquisition time. This limitation is retained in the addendum.

### EVD-002 — NTFS

- Source image: `ntfs_test_disk.img`
- Acquired image: `ntfs_evidence.dd`
- Tool: `dc3dd`
- Size: 268,435,456 bytes
- Sectors acquired: 524,288
- Bad sectors: 0
- MD5: `45e67de945924f65e11f7c467c79db9d`
- SHA-256: `53779d8ba0a946bd933915b72f2b68de215b1cf329ac8681571d319ccb85e88e`

Acquisition records and hash manifests are preserved under `logs/`.

## 3. Task 1.2 — Partition and File-System Inspection

### EXT4 examination

`mmls` identified a DOS partition table:

- Start sector: 2048
- End sector: 524287
- Length: 522240 sectors
- Partition type: Linux (0x83)

`fsstat` identified EXT4:

- Volume ID: `6f8a30f4e75d499298438a19d3b8dc8c`
- Block size: 1024 bytes
- Inode size: 256 bytes
- Root inode: 2

`fls`, `istat` and `icat` were used to inspect files and inode records.

| Inode | Path | Status | Result |
|---:|---|---|---|
| 32644 | `Documents/report.txt` | Allocated | Content recovered |
| 32645 | `Downloads/activity.txt` | Allocated | Content recovered |
| 32646 | `Temp/session.txt` orphan entry | Not allocated | Zero-size inode; content not recovered |

Recovered contents:

- `report.txt`: `Confidential Project Alpha - Internal Research`
- `activity.txt`: `User downloaded suspicious_sample.zip`

Inode 32646 had zero size and no recoverable direct data blocks. Its original content could not be recovered. The available evidence does not establish that this orphan inode contained the token later recovered from the separate NTFS image.

### NTFS examination

`mmls` identified the partition starting at sector 2048. `fsstat` confirmed NTFS:

- Volume serial: `4D3CC6A673E1649F`
- First MFT cluster: 4
- MFT mirror: 32639
- MFT entry size: 1024 bytes
- Index record size: 4096 bytes
- Sector size: 512 bytes
- Cluster size: 4096 bytes

The MFT was extracted using:

`icat -o 2048 ntfs_evidence.dd 0 > MFT`

The extracted MFT is preserved in `evidence/MFT`.

## 4. Task 1.3 — NTFS MFT and Timestamp Analysis

Sleuth Kit `fls` and `istat` were used to examine three file records.

| MFT entry | File | Status |
|---:|---|---|
| 67 | `Documents/report.txt` | Allocated |
| 68 | `Downloads/activity.txt` | Allocated |
| 69 | `Temp/session.txt` | Deleted / not allocated |

### SI and FN timestamps

The following values are converted from IST to UTC by subtracting 5 hours and 30 minutes. They are based on the recorded `istat` output.

| Entry | Timestamp field | UTC timestamp |
|---:|---|---|
| 67 | SI Created / FN Created | 2026-11-21 14:12:49.462217500 |
| 67 | SI Modified / MFT Modified | 2026-11-21 14:12:49.462823700 |
| 67 | SI Accessed / FN Accessed | 2026-11-21 14:12:49.462217500 |
| 68 | SI Created / FN Created | 2026-11-21 14:12:49.462888800 |
| 68 | SI Modified / MFT Modified | 2026-11-21 14:12:49.462926400 |
| 68 | SI Accessed / FN Accessed | 2026-11-21 14:12:49.462888800 |
| 69 | SI Created / FN Created | 2026-11-21 14:12:49.462946400 |
| 69 | SI Modified / MFT Modified | 2026-11-21 14:12:49.462966100 |
| 69 | SI Accessed / FN Accessed | 2026-11-21 14:12:49.462946400 |

The small differences between SI and FN timestamps do not, by themselves, prove timestomping. No conclusive timestomping evidence was identified from this manual comparison.

**Limitation:** MFTECmd CSV timeline generation was not completed. The documented analysis uses Sleuth Kit output (`fls`, `istat` and `icat`).

## 5. Task 1.4 — Autopsy Examination

Autopsy 2.24 was used to create case `DFIR2026001`.

- Host: `DFIR-TEST-HOST`
- Investigator: `HarishChoudhary`
- Timezone configured: Asia/Kolkata
- Image: `ntfs_evidence.dd`
- MD5 verification: matched `45e67de945924f65e11f7c467c79db9d`

The deleted file `C:/Temp/session.txt` was identified as MFT entry `69-128-2`. Its recovered content was:

`Temporary access token: TEST-12345`

Recovered content hashes reported by Autopsy:

- MD5: `92c10d5e55fdada47effd6f1ce70bdaa`
- SHA-1: `1e3923beb52c79db4147aafebd7fe86b3ebbc048`

Keyword searches found ASCII hits for `TEST-12345` and `suspicious_sample.zip`. Autopsy associated the matching cluster with `$MFT`. These string hits do not prove that a separate ZIP file existed or that a download actually occurred.

File Type Sorting processed 34 allocated files and directories. It reported 16 skipped non-files, zero extension mismatches and 18 categories. Two unknown items were identified as NTFS metadata streams `$Quota:$Q` and `$Secure:$SII`; an unknown classification alone is not evidence of malicious activity.

**Limitations:** The examination used a self-created NTFS training image rather than the challenge image specified as the preferred exercise in the training material. Only one deleted artifact was recovered, so the target of at least three deleted or hidden artifacts was not met. Autopsy 2.24 also reported an `$OrphanFiles` parser warning. Full rubric compliance is not claimed.

## 6. Task 1.5 — ExifTool Metadata Extraction

ExifTool 13.55 was used to inspect three recovered text files.

| Recovered artifact | MFT entry | Original status | Size |
|---|---:|---|---:|
| `recovered_session.txt` | 69 | Deleted | 35 bytes |
| `recovered_report.txt` | 67 | Allocated | 31 bytes |
| `recovered_activity.txt` | 68 | Allocated | 33 bytes |

ExifTool identified plain-text ASCII files (`text/plain`) with Unix LF line endings. No author, GPS, embedded office-document or image metadata was present in these text artifacts.

Filesystem timestamps on the recovered copies reflect their recovery/copy handling time and must not be treated as original source-file timestamps. Original NTFS timestamps for `session.txt` were examined separately with `istat`.

### SHA-256 of recovered artifacts

| Artifact | SHA-256 |
|---|---|
| `recovered_session.txt` | `79f73f35f4c65eb189bebf4dcc3e5418f536257d235d1b41746b1839d693751a` |
| `recovered_report.txt` | `24e774059e16641c8ae60d870aac8aff7d84baf1d19c8a9f13643443dc6959ee` |
| `recovered_activity.txt` | `ecf982dd8443e3928c1af97d0ae70567f626a55687ed3cfcdb999ee0bd8d619b` |

### Observations

- The deleted NTFS `session.txt` contained a synthetic temporary-token-like string.
- `report.txt` contained a sample confidential-project sentence.
- `activity.txt` contained a reference to a suspicious ZIP filename. The text alone does not establish that the ZIP existed or was downloaded.
- PDF- and image-specific metadata extraction was not demonstrated because no such recovered artifacts were available.

## 7. Limitations and Outstanding Requirements

1. Historical `/dev/loop0` source mapping for EVD-001 was not independently recorded.
2. The deleted EXT4 orphan inode had zero size and no recoverable content.
3. MFTECmd CSV timeline generation was not completed.
4. The Autopsy examination used a self-created image rather than the prescribed challenge image.
5. Only one deleted NTFS artifact was recovered; the three-artifact target was not met.
6. ExifTool inspection was limited to text artifacts; no PDF or image metadata sample was available.
7. Optional Task 1.6 goals (corrupted-volume carving, full `$UsnJrnl` correlation and automated timestomp detector) were not completed.

These limitations are documented to avoid overstating the findings.

## 8. Tools Used

- Kali Linux
- `dc3dd`
- Sleuth Kit: `mmls`, `fsstat`, `fls`, `istat`, `icat`, `blkls`, `ifind`, `ffind`
- Autopsy 2.24
- ExifTool 13.55
- Linux utilities: `sha256sum`, `md5sum`, `strings`, `grep`

## 9. Methodology References

- NIST SP 800-86, *Guide to Integrating Forensic Techniques into Incident Response*.
- ISO/IEC 27037, *Guidelines for identification, collection, acquisition and preservation of digital evidence*.
- RFC 3227, *Guidelines for Evidence Collection and Archiving*.

This is an educational laboratory report, not a formal forensic expert opinion.
