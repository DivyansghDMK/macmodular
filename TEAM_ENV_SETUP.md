# 🔐 Cloud Setup for Team Members

## 🎯 Quick Setup (3 Steps - 5 Minutes)

### Step 1: Get AWS Credentials from Divyansh

**Contact Divyansh securely** (NOT via GitHub/public channels):
- ✅ Signal / WhatsApp (encrypted)
- ✅ In person
- ✅ Password manager (LastPass, 1Password)

**Ask for these 4 values:**
```
1. AWS_ACCESS_KEY_ID     (starts with AKIA...)
2. AWS_SECRET_ACCESS_KEY (40 characters, lowercase letters)
3. AWS_S3_BUCKET         (bucket name, e.g., ecg-reports-bucket)
4. AWS_S3_REGION         (e.g., us-east-1)
```

---

### Step 2: Create Your `.env` File

**Run these commands in the project root:**

```bash
# Navigate to project directory
cd /path/to/modularecg-main

# Copy the template
cp env_template.txt .env

# Edit the file
nano .env  # or use: code .env, vim .env, etc.
```

---

### Step 3: Fill in Your Credentials

**Replace the placeholders with real values from Divyansh:**

#### BEFORE (Template):
```env
CLOUD_SERVICE=s3
AWS_ACCESS_KEY_ID=your_access_key_here
AWS_SECRET_ACCESS_KEY=your_secret_key_here
AWS_S3_BUCKET=your_bucket_name_here
AWS_S3_REGION=us-east-1
```

#### AFTER (Your Real Credentials):
```env
CLOUD_SERVICE=s3
AWS_ACCESS_KEY_ID=AKIAIOSFODNN7EXAMPLE
AWS_SECRET_ACCESS_KEY=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
AWS_S3_BUCKET=ecg-reports-bucket
AWS_S3_REGION=us-east-1
```

**Save and close the file.**

---

### Step 4: Test Your Setup

```bash
python3 test_cloud_connection.py
```

**Expected Success Output:**
```
======================================================================
🎉 SUCCESS! Cloud upload is properly configured!
======================================================================
```

If you see this, **you're ready to go!** 🎉

---

## 📋 Example `.env` File Structure

Your completed `.env` file should look like this:

```env
# Cloud Service
CLOUD_SERVICE=s3

# AWS Credentials (get from Divyansh)
AWS_ACCESS_KEY_ID=AKIA****************
AWS_SECRET_ACCESS_KEY=wJalrXUt*********************EXAMPLEKEY
AWS_S3_BUCKET=ecg-reports-bucket
AWS_S3_REGION=us-east-1
```

---

## 🔒 Security Checklist

### ✅ DO:
- [x] Keep `.env` file LOCAL only (never commit to Git)
- [x] Get credentials via secure channels
- [x] Test connection before working
- [x] Use your own IAM user credentials (if Divyansh provides individual access)
- [x] Delete `.env` if you leave the project

### ❌ DON'T:
- [ ] **NEVER** commit `.env` to GitHub
- [ ] **NEVER** share credentials via email/Slack/Discord
- [ ] **NEVER** screenshot your `.env` file
- [ ] **NEVER** hardcode credentials in Python files
- [ ] **NEVER** share on public channels

---

## 🆘 Troubleshooting

### ❌ Error: "Cloud Not Configured"

**Problem:** App can't find your `.env` file

**Solution:**
```bash
# Check if .env exists in project root
ls -la .env

# If not found, create it:
cp env_template.txt .env
nano .env  # Add credentials
```

---

### ❌ Error: "Access Denied" / "403 Forbidden"

**Problem:** Invalid credentials or no permissions

**Solution:**
1. Double-check credentials (no extra spaces/newlines)
2. Verify you copied the ENTIRE key (40 chars for secret key)
3. Contact Divyansh to verify your IAM user permissions

**Required IAM Permissions:**
- `s3:PutObject` (upload)
- `s3:GetObject` (download)
- `s3:ListBucket` (list files)
- `s3:DeleteObject` (delete)

---

### ❌ Error: "Bucket does not exist"

**Problem:** Bucket name is incorrect

**Solution:**
1. Check spelling (bucket names are case-sensitive)
2. Verify with Divyansh for correct bucket name
3. Make sure AWS_S3_REGION matches bucket region

---

### ❌ Error: "Region error"

**Problem:** Bucket is in a different region

**Solution:**
1. Check bucket region in AWS Console
2. Update `AWS_S3_REGION` in `.env`
3. Common regions:
   - `us-east-1` (N. Virginia)
   - `us-west-2` (Oregon)
   - `ap-south-1` (Mumbai)
   - `eu-west-1` (Ireland)

---

## 🧪 Verify Cloud Upload Works

### Test 1: Generate a Report
```bash
# Run the app
python src/main.py

# Generate a test report
# Wait 5-10 seconds
# Check console for upload confirmation
```

**Expected Console Output:**
```
======================================================================
☁️  AUTOMATIC CLOUD UPLOAD - STARTING
======================================================================
📤 Uploading to: S3
📄 Report: ECG_Report_20251110_123456.pdf
✅ PDF uploaded successfully!
✅ JSON metadata uploaded successfully!
======================================================================
🎉 CLOUD SYNC COMPLETE - Report saved both locally and in cloud!
======================================================================
```

---

### Test 2: Check Admin Panel
```bash
# Login as admin
# Go to Admin Panel > Reports
# Your report should appear within 5-10 seconds
```

---

## 📞 Need Help?

### Team Contacts:
- **Divyansh** - Team Lead, Backend, AWS Setup
- **PTR** - Frontend Development
- **Indresh** - Frontend Development
- **Android Dev** - Mobile App

### Common Questions:

**Q: Can I use my own AWS account?**
A: No, use the shared credentials Divyansh provides. This ensures everyone syncs to the same bucket.

**Q: Do I need separate credentials for each device?**
A: No, use the same `.env` on all your devices. Just don't share it with others.

**Q: What if I accidentally commit `.env` to GitHub?**
A: 
1. Immediately tell Divyansh
2. He'll rotate the credentials
3. Remove `.env` from Git history

**Q: How do I update my credentials?**
A: Just edit your `.env` file and restart the app.

---

## 🎯 Summary

**What you need:**
1. ✅ 4 AWS credentials from Divyansh
2. ✅ 5 minutes to setup
3. ✅ Run test script

**What you get:**
- ✅ Automatic cloud upload every 5 seconds
- ✅ Reports visible in Admin panel
- ✅ User signups synced to cloud
- ✅ Works identically on all devices

**Setup once, works forever!** 🚀

---

## 📄 Files You Need

```
modularecg-main/
├── .env                    ← YOU CREATE THIS (not in Git)
├── env_template.txt        ← Template to copy
├── test_cloud_connection.py ← Test script
├── TEAM_ENV_SETUP.md       ← This guide
└── CLOUD_SETUP_GUIDE.md    ← Full documentation
```

---

**Last Updated:** November 10, 2025  
**Maintained by:** Divyansh  
**Version:** 2.0 (with auto-sync)

