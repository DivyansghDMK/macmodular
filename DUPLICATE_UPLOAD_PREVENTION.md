# Duplicate Upload Prevention

## Overview

Your ECG Monitor application now includes **automatic duplicate upload prevention**. Files that have already been successfully uploaded to the cloud will not be uploaded again, saving bandwidth, storage costs, and time.

## How It Works

### 1. Upload Tracking
- Every successful upload is logged in `reports/upload_log.json`
- The log includes:
  - Filename
  - Upload timestamp
  - Cloud service used
  - Upload status
  - File metadata

### 2. Duplicate Detection
Before uploading any file, the system checks:
- **For Reports (PDFs/JSON)**: Checks if the filename has been uploaded before
- **For User Signups**: Checks if the username OR serial number has been uploaded before

### 3. Smart Skipping
If a duplicate is detected:
- Upload is skipped automatically
- A message is logged: `"File 'filename' has already been uploaded to cloud - skipping duplicate upload"`
- Status returned: `"already_uploaded"`

## What Gets Protected

### Protected from Duplicates:
✅ ECG Report PDFs  
✅ ECG Report JSON files  
✅ Metrics JSON files  
✅ User signup data  

### Still Uploads Normally:
- New reports (not previously uploaded)
- Modified reports (different filename)
- Different report formats

## User Interface

When you try to upload a file that's already been uploaded:

```
ℹ️ File 'ECG_Report_20251110_123456.pdf' has already been uploaded to cloud - skipping duplicate upload
```

For user signups:

```
ℹ️ User signup for 'john_doe' already uploaded - skipping duplicate
```

## Benefits

1. **Cost Savings** - No duplicate storage charges
2. **Bandwidth Efficiency** - Saves network bandwidth
3. **Time Savings** - Faster upload process
4. **Clean Cloud Storage** - No duplicate files cluttering your cloud storage

## Managing Upload History

### View Uploaded Files

```python
from utils.cloud_uploader import get_cloud_uploader

uploader = get_cloud_uploader()

# Get list of all uploaded files
uploaded_files = uploader.get_uploaded_files_list()
print(f"Uploaded files: {uploaded_files}")

# Get recent upload history (last 50)
history = uploader.get_upload_history(limit=50)
```

### Clear Upload Log (Allow Re-uploads)

If you need to re-upload files (e.g., after fixing corrupted uploads):

```python
from utils.cloud_uploader import get_cloud_uploader

uploader = get_cloud_uploader()

# Clear the log (creates automatic backup)
result = uploader.clear_upload_log()
print(result['message'])
```

**⚠️ Warning**: Clearing the upload log will allow all files to be re-uploaded. The system creates an automatic backup before clearing.

## Upload Log Location

The upload log is stored at:
```
/reports/upload_log.json
```

## Backup System

When you clear the upload log, an automatic backup is created:
```
/reports/upload_log.json.backup_YYYYMMDD_HHMMSS
```

## Technical Details

### File Matching Logic

**For Reports:**
- Matches based on filename (e.g., `ECG_Report_20251110_123456.pdf`)
- Checks both `local_path` and `metadata.filename` fields

**For User Signups:**
- Matches based on username OR serial number
- Prevents duplicate user registrations

### Error Handling

If the upload log cannot be read:
- System assumes file is **not uploaded** (safer approach)
- Upload proceeds normally
- Error is logged for debugging

### Status Codes

The upload system returns these status codes:

| Status | Meaning |
|--------|---------|
| `success` | File uploaded successfully |
| `already_uploaded` | File was previously uploaded, skipped |
| `skipped` | File type not eligible for upload |
| `disabled` | Cloud upload is disabled |
| `error` | Upload failed with error |

## Examples

### Example 1: First Upload (Success)

```python
result = uploader.upload_report("ECG_Report_20251110_123456.pdf")
# Result: {"status": "success", "url": "https://...", ...}
```

### Example 2: Duplicate Upload (Skipped)

```python
result = uploader.upload_report("ECG_Report_20251110_123456.pdf")
# Result: {"status": "already_uploaded", "message": "File '...' has already been uploaded..."}
```

### Example 3: New File (Success)

```python
result = uploader.upload_report("ECG_Report_20251110_234567.pdf")
# Result: {"status": "success", "url": "https://...", ...}
```

## Configuration

No additional configuration is needed. The feature works automatically with your existing cloud upload settings.

## Troubleshooting

### Problem: Files not uploading that should upload

**Solution**: Check if the file was previously uploaded:
```python
uploader = get_cloud_uploader()
uploaded_files = uploader.get_uploaded_files_list()
print("Previously uploaded:", uploaded_files)
```

### Problem: Want to force re-upload

**Solution**: Clear the upload log:
```python
uploader = get_cloud_uploader()
result = uploader.clear_upload_log()
```

### Problem: Upload log corrupted

**Solution**: Delete or rename `reports/upload_log.json` - system will create a new one

## Security & Privacy

- Upload log is stored locally on your machine
- No sensitive data (passwords, keys) are stored in the log
- Only metadata about uploads is tracked
- Log can be deleted at any time

## Performance Impact

- **Minimal**: Checking for duplicates adds <100ms per upload attempt
- **Memory**: Upload log uses minimal disk space (~1KB per 50 uploads)
- **Network**: Saves significant bandwidth by preventing duplicate uploads

## Future Enhancements

Planned improvements:
- [ ] Hash-based duplicate detection (detect renamed files)
- [ ] Configurable duplicate detection rules
- [ ] Upload retry for failed uploads
- [ ] Cloud-side duplicate detection

## Support

If you experience any issues with duplicate upload prevention, check:
1. `reports/upload_log.json` exists and is readable
2. File permissions are correct
3. Cloud upload is enabled in `.env`
4. Error messages in the console output

---

**Note**: This feature was added on November 12, 2025, to improve efficiency and reduce cloud storage costs.

