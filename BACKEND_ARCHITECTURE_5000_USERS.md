# Backend Architecture for 5000 Users with Cloud Integration

## 🎯 Overview

This document outlines a scalable backend architecture to support **5000 users** with:
- ✅ Cloud storage integration (AWS S3, Azure, GCS)
- ✅ Data accessible in both **ECG App** and **Admin Portal**
- ✅ Real-time synchronization
- ✅ High performance and reliability

---

## 📊 System Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                         CLIENT LAYER                            │
├─────────────────────┬───────────────────────────────────────────┤
│   ECG Desktop App   │         Admin Web Portal                 │
│   (PyQt5)           │         (React/Vue + PyQt5)              │
│                     │                                           │
│  • User Login       │  • Admin Dashboard                       │
│  • ECG Recording    │  • User Management                       │
│  • Report Gen       │  • Report Viewing                        │
│  • Data Upload      │  • Analytics                            │
└──────────┬──────────┴──────────────┬──────────────────────────┘
           │                         │
           │  REST API / WebSocket   │
           │  (HTTPS)                │
           ▼                         ▼
┌─────────────────────────────────────────────────────────────────┐
│                      BACKEND API LAYER                          │
├─────────────────────────────────────────────────────────────────┤
│  FastAPI Backend (Python)                                       │
│  • Authentication & Authorization                                │
│  • Data Validation & Processing                                 │
│  • Business Logic                                               │
│  • Rate Limiting & Caching                                      │
└──────────┬──────────────────────────────────────────────────────┘
           │
           ├──────────────────┬──────────────────┬───────────────┐
           ▼                  ▼                  ▼               ▼
┌──────────────┐   ┌──────────────┐   ┌──────────────┐  ┌──────────────┐
│  PostgreSQL  │   │   Redis      │   │   AWS S3     │  │   CloudWatch │
│  Database    │   │   Cache      │   │   Storage    │  │   Logging    │
│              │   │              │   │              │  │              │
│  • Users     │   │  • Sessions │   │  • Reports   │  │  • Logs      │
│  • Reports   │   │  • Cache    │   │  • Metrics   │  │  • Metrics   │
│  • Devices   │   │  • Queues   │   │  • Files     │  │  • Alerts    │
│  • Metrics   │   │              │   │              │  │              │
└──────────────┘   └──────────────┘   └──────────────┘  └──────────────┘
```

---

## 🗄️ Database Schema Design

### **1. Users Table**
```sql
CREATE TABLE users (
    user_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(255) UNIQUE,
    password_hash VARCHAR(255) NOT NULL,
    full_name VARCHAR(255),
    age INTEGER,
    gender VARCHAR(10),
    phone VARCHAR(20),
    address TEXT,
    serial_number VARCHAR(100) UNIQUE,
    role VARCHAR(20) DEFAULT 'user', -- 'user', 'admin', 'doctor'
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    last_login TIMESTAMP,
    
    -- Indexes for performance
    INDEX idx_username (username),
    INDEX idx_email (email),
    INDEX idx_serial_number (serial_number),
    INDEX idx_role (role)
);
```

### **2. Devices Table**
```sql
CREATE TABLE devices (
    device_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(user_id) ON DELETE CASCADE,
    device_type VARCHAR(20) NOT NULL, -- 'ecg', 'cpap', 'bipap', 'oxygen'
    device_serial VARCHAR(100) UNIQUE,
    device_name VARCHAR(255),
    manufacturer VARCHAR(100),
    model VARCHAR(100),
    firmware_version VARCHAR(50),
    is_active BOOLEAN DEFAULT TRUE,
    registered_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    last_seen TIMESTAMP,
    
    INDEX idx_user_id (user_id),
    INDEX idx_device_type (device_type),
    INDEX idx_device_serial (device_serial)
);
```

### **3. ECG Reports Table**
```sql
CREATE TABLE ecg_reports (
    report_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(user_id) ON DELETE CASCADE,
    device_id UUID REFERENCES devices(device_id) ON DELETE SET NULL,
    
    -- Report metadata
    filename VARCHAR(255) NOT NULL,
    file_path_s3 VARCHAR(500), -- S3 key/path
    file_size BIGINT,
    file_type VARCHAR(10) DEFAULT 'pdf',
    
    -- ECG metrics
    heart_rate INTEGER,
    pr_interval INTEGER,
    qrs_duration INTEGER,
    qtc_interval INTEGER,
    st_segment VARCHAR(50),
    qrs_axis VARCHAR(20),
    
    -- Clinical data
    patient_name VARCHAR(255),
    patient_age INTEGER,
    report_date TIMESTAMP,
    machine_serial VARCHAR(100),
    
    -- Status
    upload_status VARCHAR(20) DEFAULT 'pending', -- 'pending', 'uploaded', 'failed'
    cloud_service VARCHAR(20), -- 's3', 'azure', 'gcs'
    cloud_url TEXT,
    
    -- Timestamps
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    uploaded_at TIMESTAMP,
    
    INDEX idx_user_id (user_id),
    INDEX idx_device_id (device_id),
    INDEX idx_report_date (report_date),
    INDEX idx_upload_status (upload_status),
    INDEX idx_created_at (created_at)
);
```

### **4. ECG Metrics Table** (For real-time data)
```sql
CREATE TABLE ecg_metrics (
    metric_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(user_id) ON DELETE CASCADE,
    device_id UUID REFERENCES devices(device_id) ON DELETE SET NULL,
    
    -- Live metrics
    heart_rate INTEGER,
    pr_interval INTEGER,
    qrs_duration INTEGER,
    qtc_interval INTEGER,
    st_segment VARCHAR(50),
    qrs_axis VARCHAR(20),
    
    -- Arrhythmia detection
    arrhythmia_type VARCHAR(100), -- 'Normal Sinus Rhythm', 'Atrial Fibrillation', etc.
    arrhythmia_confidence DECIMAL(5,2), -- 0.00 to 100.00
    
    -- Raw data reference (optional, for detailed analysis)
    raw_data_s3_path VARCHAR(500),
    
    -- Timestamp
    recorded_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    INDEX idx_user_id (user_id),
    INDEX idx_device_id (device_id),
    INDEX idx_recorded_at (recorded_at),
    INDEX idx_arrhythmia_type (arrhythmia_type)
);
```

### **5. Upload Log Table** (Replaces JSON file)
```sql
CREATE TABLE upload_logs (
    log_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(user_id) ON DELETE CASCADE,
    file_path VARCHAR(500) NOT NULL,
    filename VARCHAR(255) NOT NULL,
    file_type VARCHAR(20), -- 'report', 'metric', 'user_signup'
    cloud_service VARCHAR(20),
    cloud_url TEXT,
    upload_status VARCHAR(20), -- 'success', 'failed', 'duplicate'
    error_message TEXT,
    uploaded_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    -- Prevent duplicates
    UNIQUE(user_id, filename, uploaded_at),
    INDEX idx_user_id (user_id),
    INDEX idx_filename (filename),
    INDEX idx_uploaded_at (uploaded_at)
);
```

### **6. User Sessions Table** (For authentication)
```sql
CREATE TABLE user_sessions (
    session_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(user_id) ON DELETE CASCADE,
    token_hash VARCHAR(255) NOT NULL,
    device_info VARCHAR(255),
    ip_address VARCHAR(45),
    expires_at TIMESTAMP NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    INDEX idx_user_id (user_id),
    INDEX idx_token_hash (token_hash),
    INDEX idx_expires_at (expires_at)
);
```

---

## 🔌 API Endpoints Design

### **Authentication Endpoints**

```python
POST   /api/v1/auth/register
POST   /api/v1/auth/login
POST   /api/v1/auth/logout
POST   /api/v1/auth/refresh-token
GET    /api/v1/auth/me
PUT    /api/v1/auth/update-profile
POST   /api/v1/auth/change-password
```

### **User Management Endpoints**

```python
GET    /api/v1/users                    # List users (admin only)
GET    /api/v1/users/{user_id}          # Get user details
PUT    /api/v1/users/{user_id}          # Update user (admin or self)
DELETE /api/v1/users/{user_id}          # Delete user (admin only)
GET    /api/v1/users/{user_id}/devices   # Get user's devices
GET    /api/v1/users/{user_id}/reports   # Get user's reports
GET    /api/v1/users/{user_id}/metrics   # Get user's metrics
```

### **Device Management Endpoints**

```python
POST   /api/v1/devices                  # Register device
GET    /api/v1/devices                  # List devices
GET    /api/v1/devices/{device_id}      # Get device details
PUT    /api/v1/devices/{device_id}      # Update device
DELETE /api/v1/devices/{device_id}      # Delete device
GET    /api/v1/devices/{device_id}/reports
```

### **ECG Data Endpoints**

```python
# Report Upload
POST   /api/v1/ecg/reports/upload       # Upload ECG report (multipart/form-data)
GET    /api/v1/ecg/reports               # List reports (with pagination)
GET    /api/v1/ecg/reports/{report_id}  # Get report details
GET    /api/v1/ecg/reports/{report_id}/download  # Download report PDF
DELETE /api/v1/ecg/reports/{report_id} # Delete report

# Real-time Metrics
POST   /api/v1/ecg/metrics              # Upload live metrics
GET    /api/v1/ecg/metrics               # Get metrics (with filters)
GET    /api/v1/ecg/metrics/latest       # Get latest metrics for user
GET    /api/v1/ecg/metrics/{metric_id}   # Get specific metric

# WebSocket for real-time updates
WS     /ws/ecg/{user_id}                # Real-time ECG data stream
```

### **Admin Portal Endpoints**

```python
# Dashboard Statistics
GET    /api/v1/admin/stats              # Overall statistics
GET    /api/v1/admin/stats/users        # User statistics
GET    /api/v1/admin/stats/reports      # Report statistics
GET    /api/v1/admin/stats/devices      # Device statistics

# User Management
GET    /api/v1/admin/users               # List all users (paginated)
GET    /api/v1/admin/users/{user_id}    # Get user with all data
PUT    /api/v1/admin/users/{user_id}     # Update user
DELETE /api/v1/admin/users/{user_id}    # Delete user

# Report Management
GET    /api/v1/admin/reports             # List all reports (paginated)
GET    /api/v1/admin/reports/{report_id}
DELETE /api/v1/admin/reports/{report_id}

# Analytics
GET    /api/v1/admin/analytics/overview
GET    /api/v1/admin/analytics/users
GET    /api/v1/admin/analytics/reports
GET    /api/v1/admin/analytics/devices
```

### **Cloud Storage Endpoints**

```python
POST   /api/v1/cloud/upload              # Direct file upload to cloud
GET    /api/v1/cloud/files               # List files in cloud
GET    /api/v1/cloud/files/{file_id}/url # Get presigned URL
DELETE /api/v1/cloud/files/{file_id}    # Delete file from cloud
```

---

## 🏗️ Backend Implementation (FastAPI)

### **Project Structure**

```
backend/
├── app/
│   ├── __init__.py
│   ├── main.py                 # FastAPI app entry point
│   ├── config.py               # Configuration settings
│   │
│   ├── models/                 # SQLAlchemy models
│   │   ├── __init__.py
│   │   ├── user.py
│   │   ├── device.py
│   │   ├── report.py
│   │   └── metric.py
│   │
│   ├── schemas/                # Pydantic schemas
│   │   ├── __init__.py
│   │   ├── user.py
│   │   ├── device.py
│   │   ├── report.py
│   │   └── metric.py
│   │
│   ├── api/                    # API routes
│   │   ├── __init__.py
│   │   ├── v1/
│   │   │   ├── __init__.py
│   │   │   ├── auth.py
│   │   │   ├── users.py
│   │   │   ├── devices.py
│   │   │   ├── ecg.py
│   │   │   ├── admin.py
│   │   │   └── cloud.py
│   │
│   ├── core/                   # Core functionality
│   │   ├── __init__.py
│   │   ├── security.py         # JWT, password hashing
│   │   ├── database.py         # Database connection
│   │   ├── cloud_storage.py    # S3/Azure/GCS integration
│   │   └── cache.py            # Redis caching
│   │
│   ├── services/               # Business logic
│   │   ├── __init__.py
│   │   ├── user_service.py
│   │   ├── device_service.py
│   │   ├── report_service.py
│   │   ├── metric_service.py
│   │   └── cloud_service.py
│   │
│   └── utils/                  # Utilities
│       ├── __init__.py
│       ├── validators.py
│       └── helpers.py
│
├── alembic/                    # Database migrations
│   ├── versions/
│   └── env.py
│
├── tests/                      # Unit tests
│   ├── test_auth.py
│   ├── test_users.py
│   └── test_reports.py
│
├── requirements.txt
├── .env
└── docker-compose.yml
```

### **Main Application File**

```python
# app/main.py
from fastapi import FastAPI, Depends, HTTPException
from fastapi.middleware.cors import CORSMiddleware
from fastapi.security import HTTPBearer
from app.core.database import engine, SessionLocal
from app.core.config import settings
from app.api.v1 import auth, users, devices, ecg, admin, cloud

app = FastAPI(
    title="ECG Monitoring API",
    version="1.0.0",
    description="Backend API for ECG monitoring system (5000 users)"
)

# CORS middleware
app.add_middleware(
    CORSMiddleware,
    allow_origins=settings.CORS_ORIGINS,
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# Include routers
app.include_router(auth.router, prefix="/api/v1/auth", tags=["Authentication"])
app.include_router(users.router, prefix="/api/v1/users", tags=["Users"])
app.include_router(devices.router, prefix="/api/v1/devices", tags=["Devices"])
app.include_router(ecg.router, prefix="/api/v1/ecg", tags=["ECG"])
app.include_router(admin.router, prefix="/api/v1/admin", tags=["Admin"])
app.include_router(cloud.router, prefix="/api/v1/cloud", tags=["Cloud"])

@app.get("/")
async def root():
    return {"message": "ECG Monitoring API", "version": "1.0.0"}

@app.get("/health")
async def health_check():
    return {"status": "healthy"}
```

### **Database Configuration**

```python
# app/core/database.py
from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker
from app.core.config import settings

engine = create_engine(
    settings.DATABASE_URL,
    pool_size=20,
    max_overflow=40,
    pool_pre_ping=True,
    echo=False
)

SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)
Base = declarative_base()

def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()
```

### **Cloud Storage Service**

```python
# app/services/cloud_service.py
import boto3
from botocore.exceptions import ClientError
from app.core.config import settings
from typing import Optional

class CloudStorageService:
    def __init__(self):
        self.s3_client = boto3.client(
            's3',
            aws_access_key_id=settings.AWS_ACCESS_KEY_ID,
            aws_secret_access_key=settings.AWS_SECRET_ACCESS_KEY,
            region_name=settings.AWS_REGION
        )
        self.bucket_name = settings.AWS_S3_BUCKET
    
    def upload_file(self, file_path: str, s3_key: str, metadata: dict = None) -> dict:
        """Upload file to S3"""
        try:
            extra_args = {}
            if metadata:
                extra_args['Metadata'] = metadata
            
            self.s3_client.upload_file(
                file_path,
                self.bucket_name,
                s3_key,
                ExtraArgs=extra_args
            )
            
            url = f"https://{self.bucket_name}.s3.{settings.AWS_REGION}.amazonaws.com/{s3_key}"
            
            return {
                "status": "success",
                "url": url,
                "s3_key": s3_key
            }
        except ClientError as e:
            return {
                "status": "error",
                "message": str(e)
            }
    
    def get_presigned_url(self, s3_key: str, expiration: int = 3600) -> str:
        """Generate presigned URL for file access"""
        try:
            url = self.s3_client.generate_presigned_url(
                'get_object',
                Params={'Bucket': self.bucket_name, 'Key': s3_key},
                ExpiresIn=expiration
            )
            return url
        except ClientError as e:
            raise Exception(f"Error generating presigned URL: {e}")
    
    def delete_file(self, s3_key: str) -> bool:
        """Delete file from S3"""
        try:
            self.s3_client.delete_object(Bucket=self.bucket_name, Key=s3_key)
            return True
        except ClientError:
            return False
```

---

## 🔄 Data Flow: App → Backend → Cloud → Admin Portal

### **1. User Registration Flow**

```
ECG App                    Backend API              Database              Cloud Storage
   │                           │                       │                       │
   │── POST /auth/register ───>│                       │                       │
   │                           │── Validate data ────>│                       │
   │                           │                       │── Create user ────────>│
   │                           │<── User created ─────│                       │
   │<── 201 Created ───────────│                       │                       │
   │                           │                       │                       │
   │── POST /users/{id}/signup │                       │                       │
   │                           │── Store signup ──────>│                       │
   │                           │── Upload to S3 ────────────────────────────>│
   │<── 200 OK ────────────────│                       │                       │
```

### **2. Report Upload Flow**

```
ECG App                    Backend API              Database              Cloud Storage
   │                           │                       │                       │
   │── POST /ecg/reports/upload│                       │                       │
   │   (multipart/form-data)   │                       │                       │
   │                           │── Validate file ─────>│                       │
   │                           │── Check duplicate ───>│                       │
   │                           │                       │                       │
   │                           │── Upload to S3 ────────────────────────────>│
   │                           │<── S3 URL ────────────│                       │
   │                           │                       │                       │
   │                           │── Store metadata ────>│                       │
   │                           │── Update upload_log ─>│                       │
   │<── 201 Created ───────────│                       │                       │
   │   {report_id, cloud_url}  │                       │                       │
```

### **3. Admin Portal Data Retrieval Flow**

```
Admin Portal               Backend API              Database              Cloud Storage
   │                           │                       │                       │
   │── GET /admin/users ───────>│                       │                       │
   │                           │── Query users ────────>│                       │
   │                           │<── User list ─────────│                       │
   │<── 200 OK ────────────────│                       │                       │
   │   [users...]              │                       │                       │
   │                           │                       │                       │
   │── GET /admin/reports ────>│                       │                       │
   │                           │── Query reports ─────>│                       │
   │                           │<── Report list ───────│                       │
   │                           │── Get S3 URLs ──────────────────────────────>│
   │                           │<── Presigned URLs ────│                       │
   │<── 200 OK ────────────────│                       │                       │
   │   [reports with URLs...]  │                       │                       │
```

---

## 🔐 Security & Authentication

### **JWT Token Authentication**

```python
# app/core/security.py
from datetime import datetime, timedelta
from jose import JWTError, jwt
from passlib.context import CryptContext
from app.core.config import settings

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

def verify_password(plain_password: str, hashed_password: str) -> bool:
    return pwd_context.verify(plain_password, hashed_password)

def get_password_hash(password: str) -> str:
    return pwd_context.hash(password)

def create_access_token(data: dict, expires_delta: timedelta = None):
    to_encode = data.copy()
    if expires_delta:
        expire = datetime.utcnow() + expires_delta
    else:
        expire = datetime.utcnow() + timedelta(hours=24)
    
    to_encode.update({"exp": expire})
    encoded_jwt = jwt.encode(to_encode, settings.SECRET_KEY, algorithm="HS256")
    return encoded_jwt

def verify_token(token: str):
    try:
        payload = jwt.decode(token, settings.SECRET_KEY, algorithms=["HS256"])
        return payload
    except JWTError:
        return None
```

### **Role-Based Access Control**

```python
# app/core/dependencies.py
from fastapi import Depends, HTTPException, status
from fastapi.security import HTTPBearer
from app.core.security import verify_token
from app.models.user import User

security = HTTPBearer()

async def get_current_user(token: str = Depends(security)) -> User:
    payload = verify_token(token.credentials)
    if not payload:
        raise HTTPException(status_code=401, detail="Invalid token")
    
    user_id = payload.get("sub")
    user = db.query(User).filter(User.user_id == user_id).first()
    if not user:
        raise HTTPException(status_code=404, detail="User not found")
    
    return user

async def get_admin_user(current_user: User = Depends(get_current_user)) -> User:
    if current_user.role != "admin":
        raise HTTPException(status_code=403, detail="Admin access required")
    return current_user
```

---

## 📈 Performance Optimization for 5000 Users

### **1. Database Indexing**
- Index on `user_id`, `device_id`, `report_date`, `created_at`
- Composite indexes for common queries
- Regular `VACUUM` and `ANALYZE` operations

### **2. Caching Strategy (Redis)**
```python
# Cache frequently accessed data
- User sessions: 24 hours
- User profiles: 1 hour
- Report lists: 15 minutes
- Statistics: 5 minutes
```

### **3. Pagination**
```python
# All list endpoints support pagination
GET /api/v1/users?page=1&limit=50
GET /api/v1/ecg/reports?page=1&limit=20&user_id=xxx
```

### **4. Connection Pooling**
- PostgreSQL: Pool size 20, max overflow 40
- Redis: Connection pool size 10

### **5. Async Operations**
- File uploads: Background tasks
- Report generation: Queue system (Celery)
- Email notifications: Async

---

## 🔧 Configuration

### **Environment Variables (.env)**

```bash
# Database
DATABASE_URL=postgresql://user:password@localhost:5432/ecg_db
DATABASE_POOL_SIZE=20
DATABASE_MAX_OVERFLOW=40

# Redis
REDIS_URL=redis://localhost:6379/0
REDIS_CACHE_TTL=3600

# AWS S3
AWS_ACCESS_KEY_ID=your_access_key
AWS_SECRET_ACCESS_KEY=your_secret_key
AWS_REGION=us-east-1
AWS_S3_BUCKET=ecg-reports-bucket

# Security
SECRET_KEY=your-secret-key-here
ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_HOURS=24

# CORS
CORS_ORIGINS=["http://localhost:3000", "http://localhost:8000"]

# Application
API_V1_PREFIX=/api/v1
DEBUG=False
LOG_LEVEL=INFO
```

---

## 🚀 Deployment Architecture

### **Production Setup**

```
┌─────────────────────────────────────────────────────────┐
│                    Load Balancer                        │
│                    (AWS ALB/Nginx)                       │
└────────────────────┬──────────────────────────────────────┘
                     │
        ┌────────────┴────────────┐
        │                         │
        ▼                         ▼
┌──────────────┐          ┌──────────────┐
│  FastAPI     │          │  FastAPI     │
│  Instance 1  │          │  Instance 2  │
│  (Gunicorn)  │          │  (Gunicorn)  │
└──────┬───────┘          └──────┬───────┘
       │                         │
       └────────────┬────────────┘
                    │
        ┌───────────┴───────────┐
        │                       │
        ▼                       ▼
┌──────────────┐        ┌──────────────┐
│  PostgreSQL  │        │    Redis     │
│  (RDS)       │        │  (ElastiCache)│
└──────────────┘        └──────────────┘
```

### **Docker Compose (Development)**

```yaml
version: '3.8'

services:
  backend:
    build: ./backend
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=postgresql://postgres:password@db:5432/ecg_db
      - REDIS_URL=redis://redis:6379/0
    depends_on:
      - db
      - redis

  db:
    image: postgres:14
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=password
      - POSTGRES_DB=ecg_db
    volumes:
      - postgres_data:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data

volumes:
  postgres_data:
  redis_data:
```

---

## 📱 Client Integration (ECG App)

### **Update Cloud Uploader to Use Backend API**

```python
# src/utils/backend_api.py (New file)
import requests
from typing import Optional, Dict
import os

class BackendAPI:
    def __init__(self):
        self.base_url = os.getenv('BACKEND_API_URL', 'http://localhost:8000')
        self.token = None
    
    def login(self, username: str, password: str) -> bool:
        """Login and store token"""
        response = requests.post(
            f"{self.base_url}/api/v1/auth/login",
            json={"username": username, "password": password}
        )
        if response.status_code == 200:
            data = response.json()
            self.token = data.get("access_token")
            return True
        return False
    
    def upload_report(self, file_path: str, metadata: dict) -> dict:
        """Upload report via backend API"""
        headers = {"Authorization": f"Bearer {self.token}"}
        
        with open(file_path, 'rb') as f:
            files = {'file': f}
            data = metadata
            response = requests.post(
                f"{self.base_url}/api/v1/ecg/reports/upload",
                headers=headers,
                files=files,
                data=data
            )
        
        return response.json()
    
    def upload_metrics(self, metrics: dict) -> dict:
        """Upload live metrics"""
        headers = {"Authorization": f"Bearer {self.token}"}
        response = requests.post(
            f"{self.base_url}/api/v1/ecg/metrics",
            headers=headers,
            json=metrics
        )
        return response.json()
```

---

## 📊 Admin Portal Integration

### **Update Admin Portal to Fetch from Backend**

```python
# src/dashboard/admin_reports.py (Update)
class AdminReportsDialog(QDialog):
    def __init__(self, backend_api, parent=None):
        super().__init__(parent)
        self.backend_api = backend_api  # BackendAPI instance
        # ... rest of initialization
    
    def load_items(self):
        """Load reports from backend API instead of S3"""
        try:
            response = self.backend_api.get_reports()
            if response.status_code == 200:
                reports = response.json()
                self.populate_reports_table(reports)
        except Exception as e:
            QMessageBox.warning(self, "Error", f"Failed to load reports: {e}")
    
    def load_users(self):
        """Load users from backend API"""
        try:
            response = self.backend_api.get_users()
            if response.status_code == 200:
                users = response.json()
                self.populate_users_table(users)
        except Exception as e:
            QMessageBox.warning(self, "Error", f"Failed to load users: {e}")
```

---

## 📈 Scalability Considerations

### **For 5000 Users:**
- ✅ **Database**: PostgreSQL can handle 5000 users easily
- ✅ **API**: FastAPI with async can handle 1000+ concurrent requests
- ✅ **Caching**: Redis reduces database load by 70-80%
- ✅ **File Storage**: S3 scales automatically
- ✅ **Load Balancing**: Add more instances as needed

### **Future Scaling (10,000+ users):**
- Add read replicas for PostgreSQL
- Implement CDN for file downloads
- Use message queue (RabbitMQ/Kafka) for async tasks
- Microservices architecture for different domains

---

## 🧪 Testing Strategy

### **Unit Tests**
- Test all API endpoints
- Test authentication/authorization
- Test data validation

### **Integration Tests**
- Test database operations
- Test cloud storage integration
- Test end-to-end flows

### **Load Tests**
- Test with 5000 concurrent users
- Test file upload performance
- Test database query performance

---

## 📝 Migration Plan

### **Phase 1: Backend Setup (Week 1-2)**
1. Set up PostgreSQL database
2. Create database schema
3. Deploy FastAPI backend
4. Set up Redis cache
5. Configure S3 integration

### **Phase 2: API Development (Week 3-4)**
1. Implement authentication endpoints
2. Implement user management endpoints
3. Implement report upload endpoints
4. Implement admin endpoints
5. Add WebSocket support

### **Phase 3: Client Integration (Week 5-6)**
1. Update ECG app to use backend API
2. Update admin portal to use backend API
3. Test end-to-end flows
4. Migrate existing data from JSON files

### **Phase 4: Testing & Deployment (Week 7-8)**
1. Load testing
2. Security audit
3. Production deployment
4. Monitor and optimize

---

## 💰 Cost Estimation (Monthly)

### **AWS Services:**
- **RDS PostgreSQL** (db.t3.medium): ~$60/month
- **ElastiCache Redis** (cache.t3.micro): ~$15/month
- **S3 Storage** (100GB): ~$2.30/month
- **EC2 Instance** (t3.medium for backend): ~$30/month
- **Data Transfer**: ~$10/month
- **CloudWatch**: ~$5/month

**Total: ~$122/month**

### **Alternative (Self-hosted):**
- **VPS** (4GB RAM, 2 CPU): ~$20/month
- **PostgreSQL**: Included
- **Redis**: Included
- **S3 Storage**: ~$2.30/month

**Total: ~$22/month** (but requires more maintenance)

---

## ✅ Summary

This architecture provides:
- ✅ **Scalable backend** for 5000+ users
- ✅ **Unified data storage** in PostgreSQL
- ✅ **Cloud integration** (S3/Azure/GCS)
- ✅ **Real-time data** via WebSocket
- ✅ **Secure authentication** with JWT
- ✅ **Admin portal** with full access
- ✅ **Performance optimization** with caching
- ✅ **Easy migration** from current JSON-based system

**Next Steps:**
1. Review and approve this architecture
2. Set up development environment
3. Start with Phase 1 (Backend Setup)
4. Iterate and deploy


