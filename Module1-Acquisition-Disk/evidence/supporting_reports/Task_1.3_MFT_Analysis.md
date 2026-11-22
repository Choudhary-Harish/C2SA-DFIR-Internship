# Task 1.3 — NTFS MFT Analysis

## 1. Objective
Extract and examine the NTFS Master File Table (MFT), compare `$STANDARD_INFORMATION` and `$FILE_NAME` timestamps, identify deleted files, and attempt recovery.

## 2. Evidence
- Evidence image: `ntfs_evidence.dd`
- Partition offset: 2048 sectors
- Extracted MFT: `MFT`
- Filesystem: NTFS
- Case ID: DFIR-2026-001

## 3. Tools Used
- Sleuth Kit: `fls`, `istat`, `icat`
- `dc3dd`
- `sha256sum`

## 4. MFT Extraction
Command:
`icat -o 2048 ntfs_evidence.dd 0 > MFT`

Result: MFT extracted successfully.

## 5. File Identification

| MFT Entry | Filename | Status |
|---|---|---|
| 67 | report.txt | Allocated |
| 68 | activity.txt | Allocated |
| 69 | session.txt | Deleted |

## 6. SI vs FN Timestamp Analysis

- `report.txt`: SI and FN modified timestamps differed by approximately 0.607 milliseconds.
- `activity.txt`: Difference was approximately 0.038 milliseconds.
- `session.txt`: Difference was approximately 0.020 milliseconds.

The timestamp differences are very small. No conclusive evidence of timestomping was identified.

## 7. Deleted File Recovery

MFT Entry: 69  
Filename: `session.txt`  
Parent directory: `Temp`  
Status: Not Allocated

Recovery command:
`icat -o 2048 ntfs_evidence.dd 69`

Recovered content:
`Temporary access token: TEST-12345`

The deleted file's content was successfully recovered from the forensic image.

## 8. Conclusion

The NTFS MFT was extracted and examined successfully. Three test files were identified. SI and FN timestamps were compared, and no conclusive evidence of timestomping was found. The deleted file `session.txt` was identified and its original content was successfully recovered.

## 9. Evidence Integrity

The acquired image's SHA-256 hash was verified:

`53779d8ba0a946bd933915b72f2b68de215b1cf329ac8681571d319ccb85e88e`

The original evidence image was not modified during analysis.
