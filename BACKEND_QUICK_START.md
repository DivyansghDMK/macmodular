# Backend Quick Start Guide

## 🎯 What You're Building

A **scalable backend** that:
- Handles **5000 users**
- Stores data in **PostgreSQL** (not JSON files)
- Integrates with **cloud storage** (S3/Azure/GCS)
- Shows data in both **ECG App** and **Admin Portal**
- Provides **REST API** for all operations

---

## 🏗️ Architecture Overview

```
ECG App ──┐
          ├──> Backend API (FastAPI) ──> PostgreSQL Database
Admin ────┘                              └──> Cloud Storage (S3)
```

---

## 📋 Key Components

### **1. Database (PostgreSQL)**
- **Users Table**: All user accounts
- **Devices Table**: ECG/CPAP/BiPAP/Oxygen devices
- **ECG Reports Table**: Report metadata + S3 links
- **ECG Metrics Table**: Real-time metrics
- **Upload Logs Table**: Tracks all uploads (replaces JSON)

### **2. Backend API (FastAPI)**
- **Authentication**: Login, register, JWT tokens
- **User Management**: CRUD operations
- **Report Upload**: Upload reports → Database → S3
- **Admin Endpoints**: Full admin access
- **Real-time**: WebSocket for live data

### **3. Cloud Storage (AWS S3)**
- Stores actual PDF files
- Stores JSON metrics
- Backend generates presigned URLs for access

---

## 🔌 Main API Endpoints

### **Authentication**
```
POST /api/v1/auth/login          # Login
POST /api/v1/auth/register       # Register
GET  /api/v1/auth/me            # Current user
```

### **Reports**
```
POST /api/v1/ecg/reports/upload  # Upload report
GET  /api/v1/ecg/reports         # List reports
GET  /api/v1/ecg/reports/{id}    # Get report
```

### **Admin**
```
GET  /api/v1/admin/users         # List all users
GET  /api/v1/admin/reports       # List all reports
GET  /api/v1/admin/stats         # Statistics
```

---

## 🔄 Data Flow

### **Report Upload:**
1. ECG App uploads PDF → Backend API
2. Backend validates & stores metadata → PostgreSQL
3. Backend uploads file → S3
4. Backend saves S3 URL → Database
5. Response with report_id & cloud_url

### **Admin Portal View:**
1. Admin Portal requests → Backend API
2. Backend queries → PostgreSQL
3. Backend generates presigned URLs → S3
4. Response with reports + download URLs

---

## 🚀 Quick Setup Steps

### **1. Install Dependencies**
```bash
pip install fastapi uvicorn sqlalchemy psycopg2-binary
pip install boto3 redis python-jose[cryptography] passlib[bcrypt]
```

### **2. Set Up Database**
```bash
# Create PostgreSQL database
createdb ecg_db

# Run migrations (Alembic)
alembic upgrade head
```

### **3. Configure Environment**
```bash
# .env file
DATABASE_URL=postgresql://user:pass@localhost/ecg_db
AWS_ACCESS_KEY_ID=your_key
AWS_SECRET_ACCESS_KEY=your_secret
AWS_S3_BUCKET=ecg-reports
SECRET_KEY=your-secret-key
```

### **4. Start Backend**
```bash
uvicorn app.main:app --reload --port 8000
```

### **5. Update ECG App**
```python
# Replace CloudUploader with BackendAPI
from utils.backend_api import BackendAPI

backend = BackendAPI()
backend.login(username, password)
backend.upload_report(file_path, metadata)
```

---

## 📊 Database Schema (Simplified)

```sql
-- Users
users (user_id, username, email, password_hash, role, ...)

-- Reports
ecg_reports (report_id, user_id, filename, s3_path, heart_rate, ...)

-- Metrics
ecg_metrics (metric_id, user_id, heart_rate, recorded_at, ...)
```

---

## 🔐 Security Features

- **JWT Tokens**: Secure authentication
- **Password Hashing**: bcrypt
- **Role-Based Access**: Admin vs User
- **CORS Protection**: Configured origins
- **Rate Limiting**: Prevent abuse

---

## 📈 Performance for 5000 Users

- **Database**: PostgreSQL handles easily
- **Caching**: Redis for frequently accessed data
- **Pagination**: All list endpoints paginated
- **Connection Pooling**: Optimized for concurrency

---

## 💰 Estimated Costs

**AWS (Production):**
- RDS PostgreSQL: ~$60/month
- Redis Cache: ~$15/month
- S3 Storage: ~$2/month
- EC2 Backend: ~$30/month
- **Total: ~$120/month**

**Self-Hosted (VPS):**
- VPS (4GB RAM): ~$20/month
- S3 Storage: ~$2/month
- **Total: ~$22/month**

---

## ✅ Benefits Over Current System

| Current (JSON Files) | New (Backend + Database) |
|---------------------|-------------------------|
| ❌ No relationships | ✅ Relational data |
| ❌ Slow queries | ✅ Fast indexed queries |
| ❌ No authentication | ✅ JWT authentication |
| ❌ Manual sync | ✅ Automatic sync |
| ❌ Limited to 1 device | ✅ Multi-device ready |
| ❌ No admin features | ✅ Full admin portal |

---

## 🎯 Next Steps

1. **Review** `BACKEND_ARCHITECTURE_5000_USERS.md` for full details
2. **Set up** development environment
3. **Create** database schema
4. **Build** FastAPI backend
5. **Integrate** with ECG app
6. **Deploy** to production

---

## 📚 Key Files to Create

```
backend/
├── app/
│   ├── main.py              # FastAPI app
│   ├── models/             # Database models
│   ├── api/v1/             # API endpoints
│   ├── services/           # Business logic
│   └── core/               # Config, security, database
```

---

## 🆘 Common Questions

**Q: Do I need to migrate existing data?**
A: Yes, but backend can read from current JSON files and migrate automatically.

**Q: Can I keep using S3 directly?**
A: Yes, but backend provides better organization, relationships, and security.

**Q: Will this work with CPAP/BiPAP/Oxygen apps?**
A: Yes! Same backend, different device types.

**Q: How long to implement?**
A: 6-8 weeks for full implementation (see migration plan in main doc).

---

**Ready to start?** See `BACKEND_ARCHITECTURE_5000_USERS.md` for complete details! 🚀


