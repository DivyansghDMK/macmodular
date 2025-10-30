# 🚀 AWS S3 Setup - Step by Step Guide

**Follow these exact steps in AWS Console to set up S3 for ECG reports upload**

---

## **Step 1: Create S3 Bucket**

1. **Go to AWS Console** → Search "S3" → Click "S3" service
2. **Click "Create bucket"** (orange button, top right)
3. **Fill in details:**
   - **Bucket name:** `ecg-reports-your-name` (must be unique - try your name/company)
     - Example: `ecg-reports-deckmount` or `ecg-reports-company-initials`
   - **AWS Region:** Choose closest to you (e.g., `us-east-1`, `ap-south-1`)
4. **Object Ownership:** Leave default (ACLs disabled)
5. **Block Public Access settings:**
   - ✅ Check "Block all public access" (keep it private)
   - Uncheck any public access boxes
6. **Bucket Versioning:** Disable (optional)
7. **Default encryption:** Enable (recommended) - AES256
8. **Click "Create bucket"** at bottom

---

## **Step 2: Create IAM User**

1. **Go to AWS Console** → Search "IAM" → Click "IAM" service
2. **Click "Users" in left sidebar**
3. **Click "Create user"** (blue button)
4. **User name:** `ecg-reports-uploader`
5. **Provide user access to the AWS Management Console:** ❌ Uncheck this
6. **Click "Next"**
7. **Set permissions:**
   - Click "Attach policies directly"
   - Search: `AmazonS3FullAccess`
   - ✅ Check "AmazonS3FullAccess"
   - Click "Next"
8. **Add tags:** Skip (optional)
9. **Click "Create user"**
10. **IMPORTANT - Save Credentials Now!**
    - Click on the user name `ecg-reports-uploader`
    - Click "Security credentials" tab
    - Click "Create access key"
    - Choose: "Application running outside AWS"
    - Click "Next"
    - Click "Create access key"
    - **COPY BOTH VALUES NOW!**
      - **Access Key ID:** (starts with `AKIA...`)
      - **Secret Access Key:** (long random string)
      - Click "Done"
    - ⚠️ **You won't see the secret key again! Save it!**

---

## **Step 3: Create .env File in ECG Project**

1. **Go to your ECG project folder:**
   ```bash
   cd /Users/deckmount/Downloads/modularecg-main
   ```

2. **Create `.env` file** in project root:
   ```bash
   touch .env
   ```

3. **Add this content to `.env` file:**
   ```bash
   # AWS S3 Configuration for ECG Reports
   CLOUD_UPLOAD_ENABLED=true
   CLOUD_SERVICE=s3

   # Replace with YOUR values from Step 2
   AWS_S3_BUCKET=ecg-reports-your-name
   AWS_S3_REGION=us-east-1
   AWS_ACCESS_KEY_ID=AKIAxxxxxxxxxxxxxxxx
   AWS_SECRET_ACCESS_KEY=your_secret_key_here
   ```

   **Replace with your actual values:**
   - `AWS_S3_BUCKET` → Your bucket name from Step 1
   - `AWS_S3_REGION` → Your region from Step 1
   - `AWS_ACCESS_KEY_ID` → From Step 2
   - `AWS_SECRET_ACCESS_KEY` → From Step 2

---

## **Step 4: Install boto3 (AWS SDK)**

Run this in terminal:
```bash
cd /Users/deckmount/Downloads/modularecg-main
pip install boto3
```

---

## **Step 5: Test Upload**

1. **Generate a test ECG report** in the app
2. **Click "☁️ Sync to Cloud" button** on dashboard
3. **Check for success message**
4. **Verify in AWS S3:**
   - Go to S3 Console
   - Click your bucket
   - Look in `ecg-reports/YYYY/MM/DD/` folder
   - You should see your PDF report!

---

## **📋 Quick Checklist**

- [ ] Created S3 bucket: `ecg-reports-your-name`
- [ ] Created IAM user: `ecg-reports-uploader`
- [ ] Saved Access Key ID and Secret Key
- [ ] Created `.env` file with your credentials
- [ ] Installed boto3: `pip install boto3`
- [ ] Tested upload with sync button

---

## **🔒 Security Notes**

✅ **What uploads:**
- PDF ECG reports only
- Metric JSON files only
- Patient name, age, ECG metrics

❌ **What doesn't upload:**
- Session logs
- Crash logs
- Debug files
- Config files
- Raw ECG data

---

## **💰 Estimated Cost**

For ~100 reports/month (1MB each):
- **Storage:** ~$0.002/month
- **Upload:** ~$0.0005/month
- **Total:** ~$0.003/month (less than $0.05/year)

**Extremely low cost for ECG backup!**

---

## **✅ Done!**

Your ECG Monitor app will now upload reports to AWS S3 when you click the "☁️ Sync to Cloud" button on the dashboard!

**Location in S3:** `ecg-reports/2025/10/28/ECG_Report_20251028_143022.pdf`

