# Task 1.5 — Document Media Metadata Extraction

## 1. Examination Overview

- Case ID: DFIR-2026-001
- Evidence Image: ntfs_evidence.dd
- Working Directory: /home/harish/DFIR-Lab
- Tool: ExifTool 13.55
- Supporting Tool: The Sleuth Kit
- Purpose: Recover selected NTFS artifacts, examine available metadata and calculate SHA-256 hashes.

## 2. Recovered Artifacts

| Artifact | MFT Entry | Original Status | Size |
|---|---:|---|---:|
| session.txt | 69 | Deleted | 35 bytes |
| report.txt | 67 | Allocated | 31 bytes |
| activity.txt | 68 | Allocated | 33 bytes |

All three files were recovered using icat from ntfs_evidence.dd.

## 3. ExifTool Findings

### recovered_session.txt

- File Type: TXT
- MIME Type: text/plain
- Encoding: us-ascii
- Line Count: 1
- Word Count: 4
- File Size: 35 bytes
- Recovered-copy modification time: 2026-11-22 18:02:40 IST
- Recovered-copy access time: 2026-11-22 18:02:59 IST

Content: Temporary access token: TEST-12345

### recovered_report.txt

- File Type: TXT
- MIME Type: text/plain
- Encoding: us-ascii
- Line Count: 1
- Word Count: 3
- File Size: 31 bytes
- Recovered-copy modification time: 2026-11-22 18:04:39 IST
- Recovered-copy access time: 2026-11-22 18:04:57 IST

Content: Confidential research document

### recovered_activity.txt

- File Type: TXT
- MIME Type: text/plain
- Encoding: us-ascii
- Line Count: 1
- Word Count: 2
- File Size: 33 bytes
- Recovered-copy modification time: 2026-11-22 18:05:50 IST
- Recovered-copy access time: 2026-11-22 18:06:01 IST

Content: Downloaded suspicious_sample.zip

## 4. Original NTFS Timestamp Observation

Original timestamps were examined using istat.

For MFT Entry 69 (session.txt):

- Created: 2026-11-21 19:42:49.462946400 IST
- File Modified: 2026-11-21 19:42:49.462966100 IST
- MFT Modified: 2026-11-21 19:42:49.462966100 IST
- Accessed: 2026-11-21 19:42:49.462946400 IST
- Allocation Status: Not Allocated
- Links: 0

The timestamps reported by ExifTool belong to the recovered copies. They must not be treated as the original file-system timestamps.

## 5. SHA-256 Hashes

| Artifact | SHA-256 |
|---|---|
| recovered_session.txt | 79f73f35f4c65eb189bebf4dcc3e5418f536257d235d1b41746b1839d693751a |
| recovered_report.txt | 24e774059e16641c8ae60d870aac8aff7d84baf1d19c8a9f13643443dc6959ee |
| recovered_activity.txt | ecf982dd8443e3928c1af97d0ae70567f626a55687ed3cfcdb999ee0bd8d619b |

Hash values are recorded in recovered_artifacts_sha256.txt.

## 6. Investigative Observations

1. The deleted session.txt was successfully recovered from MFT Entry 69.
2. The recovered session file contains the text "Temporary access token: TEST-12345".
3. report.txt contains "Confidential research document".
4. activity.txt contains "Downloaded suspicious_sample.zip".
5. The activity.txt string alone does not establish that the ZIP archive existed or was downloaded.
6. ExifTool identified all three recovered artifacts as plain ASCII text files.
7. No author, GPS coordinates, or embedded document-software metadata were reported.

## 7. Limitations

- The examined artifacts were plain-text files, not PDF or image documents.
- Therefore, PDF/image-specific metadata such as author, EXIF, GPS, and embedded software details could not be examined.
- Recovered-copy timestamps differ from original NTFS timestamps.
- This examination used a self-created NTFS training image.

## 8. Conclusion

Three text artifacts were recovered and examined. Their available file metadata and SHA-256 hashes were documented. Original NTFS timestamps were distinguished from recovered-copy timestamps. No unsupported conclusion about the existence of suspicious_sample.zip or malicious activity is made.

Task 1.5 was completed for the available text artifacts, with PDF/image metadata extraction remaining outside the scope of this examination.
