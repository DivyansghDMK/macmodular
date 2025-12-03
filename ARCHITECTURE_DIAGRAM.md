# ECG Monitor Application - End-to-End Architecture

## System Architecture Diagram

```mermaid
graph TB
    %% Entry Point
    START[User Launches Application] --> MAIN[main.py<br/>Application Entry Point]
    
    %% Initialization Layer
    MAIN --> INIT{Initialize Core Systems}
    INIT --> LOGGER[Core Logging System<br/>core/logging_config.py]
    INIT --> CONFIG[Configuration Manager<br/>config/settings.py]
    INIT --> CRASH[Crash Logger<br/>utils/crash_logger.py]
    INIT --> CONSTANTS[Constants & Messages<br/>core/constants.py]
    
    %% Authentication Flow
    MAIN --> SPLASH[Splash Screen<br/>splash_screen.py]
    SPLASH --> LOGIN[Login/Register Dialog<br/>main.py]
    LOGIN --> AUTH{Authentication}
    AUTH --> SIGNIN[Sign In Handler<br/>auth/sign_in.py]
    AUTH --> REGISTER[User Registration]
    
    %% User Data Storage
    SIGNIN --> USERDB[(users.json<br/>User Database)]
    REGISTER --> USERDB
    REGISTER --> CLOUD_UPLOAD[Cloud Upload<br/>utils/cloud_uploader.py]
    
    %% Admin Flow
    AUTH --> ADMIN_CHECK{Admin User?}
    ADMIN_CHECK -->|Yes| ADMIN[Admin Dashboard<br/>dashboard/admin_reports.py]
    ADMIN --> S3_REPORTS[(AWS S3<br/>Cloud Reports)]
    ADMIN --> S3_USERS[(AWS S3<br/>User Data)]
    
    %% Main Dashboard Flow
    ADMIN_CHECK -->|No| DASHBOARD[Main Dashboard<br/>dashboard/dashboard.py]
    
    %% Dashboard Components
    DASHBOARD --> SESSION[Session Recorder<br/>utils/session_recorder.py]
    DASHBOARD --> AUTOSYNC[Auto Sync Service<br/>utils/auto_sync_service.py]
    DASHBOARD --> SETTINGS[Settings Manager<br/>utils/settings_manager.py]
    
    %% Dashboard UI Components
    DASHBOARD --> DASH_UI{Dashboard UI Components}
    DASH_UI --> METRICS[Live ECG Metrics Display]
    DASH_UI --> CALENDAR[Calendar & Reports Filter]
    DASH_UI --> REPORTS_LIST[Recent Reports Panel]
    DASH_UI --> CONCLUSION[Conclusion Analysis]
    DASH_UI --> HEART_VIZ[3D Heart Visualization<br/>utils/heartbeat_widget.py]
    DASH_UI --> CHATBOT[AI Chatbot<br/>dashboard/chatbot_dialog.py]
    
    %% ECG Test Flow
    DASHBOARD --> ECG_BTN[ECG Lead Test 12 Button]
    ECG_BTN --> ECG_TEST[ECG Test Page<br/>ecg/twelve_lead_test.py]
    
    %% ECG Data Sources
    ECG_TEST --> DATA_SOURCE{Data Source}
    DATA_SOURCE -->|Hardware Mode| SERIAL[Serial Port Reader<br/>Hardware ECG Device]
    DATA_SOURCE -->|Demo Mode| DEMO[Demo Manager<br/>ecg/demo_manager.py]
    
    %% ECG Processing
    SERIAL --> ECG_PROC[ECG Signal Processing]
    DEMO --> ECG_PROC
    ECG_PROC --> FILTER[Signal Filtering<br/>8-Stage Medical Grade]
    FILTER --> LEAD_CALC[12-Lead Calculation<br/>From 8 Hardware Channels]
    LEAD_CALC --> METRICS_CALC[Metrics Calculation]
    
    %% Metrics Calculation
    METRICS_CALC --> HR[Heart Rate<br/>Pan-Tompkins Algorithm<br/>ecg/pan_tompkins.py]
    METRICS_CALC --> PR[PR Interval]
    METRICS_CALC --> QRS[QRS Duration & Axis]
    METRICS_CALC --> QT[QT/QTc Interval]
    METRICS_CALC --> ST[ST Segment]
    
    %% ECG Visualization
    ECG_TEST --> VIZ[12-Lead Real-Time Visualization<br/>PyQtGraph]
    VIZ --> LEAD_CLICK[Click on Lead]
    LEAD_CLICK --> EXPANDED[Expanded Lead View<br/>ecg/expanded_lead_view.py]
    
    %% Expanded Lead Analysis
    EXPANDED --> PQRST[PQRST Wave Detection<br/>PQRST Analyzer]
    EXPANDED --> ARRHYTHMIA[Arrhythmia Detection<br/>Arrhythmia Detector]
    EXPANDED --> ZOOM[Amplification Controls]
    EXPANDED --> HISTORY[History Slider]
    
    %% Report Generation
    ECG_TEST --> GENERATE[Generate Report Button]
    GENERATE --> REPORT_GEN[Report Generator<br/>ecg/ecg_report_generator.py]
    REPORT_GEN --> PDF[PDF Report<br/>ReportLab]
    REPORT_GEN --> LEAD_IMG[10-Second Lead Images<br/>Matplotlib]
    
    %% Report Storage
    PDF --> LOCAL_REPORTS[(Local Storage<br/>reports/ folder)]
    PDF --> REPORT_INDEX[(reports/index.json<br/>Recent Reports Index)]
    REPORT_INDEX --> REPORTS_LIST
    
    %% Cloud Sync
    AUTOSYNC --> OFFLINE_QUEUE[Offline Queue<br/>utils/offline_queue.py]
    OFFLINE_QUEUE --> CLOUD_CHECK{Internet<br/>Available?}
    CLOUD_CHECK -->|Yes| CLOUD_UPLOAD
    CLOUD_CHECK -->|No| OFFLINE_QUEUE
    CLOUD_UPLOAD --> S3_REPORTS
    
    %% Settings & Recording
    SETTINGS --> ECG_SETTINGS[(ecg_settings.json<br/>Wave Speed, Gain, etc)]
    SESSION --> SESSION_LOG[(Session Logs<br/>reports/sessions/)]
    
    %% Crash Handling
    CRASH --> CRASH_LOG[(logs/crash_logs.json)]
    CRASH --> EMAIL[Email Notification<br/>Gmail SMTP]
    
    %% AI Features
    CHATBOT --> AI_INSIGHTS[Dashboard Insights<br/>ai/dashboard_insights.py]
    CHATBOT --> AI_ENHANCE[Report Enhancer<br/>ai/report_enhancer.py]
    
    %% Configuration Files
    CONFIG --> ENV["ENV File<br/>Cloud Credentials"]
    CONFIG --> APP_CONFIG[(ecg_app.log<br/>Application Logs)]
    
    %% Sign Out
    DASHBOARD --> SIGNOUT[Sign Out Button<br/>auth/sign_out.py]
    SIGNOUT --> SESSION_END[End Session]
    SESSION_END --> LOGIN
    
    %% Styling
    classDef entryPoint fill:#ff6600,stroke:#e65c00,color:#fff
    classDef auth fill:#4CAF50,stroke:#45a049,color:#fff
    classDef dashboard fill:#2196F3,stroke:#1976D2,color:#fff
    classDef ecg fill:#9C27B0,stroke:#7B1FA2,color:#fff
    classDef storage fill:#FFC107,stroke:#FFA000,color:#000
    classDef cloud fill:#00BCD4,stroke:#0097A7,color:#fff
    classDef ai fill:#E91E63,stroke:#C2185B,color:#fff
    classDef util fill:#607D8B,stroke:#455A64,color:#fff
    
    class START,MAIN entryPoint
    class LOGIN,AUTH,SIGNIN,REGISTER,ADMIN_CHECK,SIGNOUT auth
    class DASHBOARD,DASH_UI,METRICS,CALENDAR,REPORTS_LIST,CONCLUSION,HEART_VIZ dashboard
    class ECG_TEST,ECG_PROC,FILTER,LEAD_CALC,METRICS_CALC,HR,PR,QRS,QT,ST,VIZ,EXPANDED,PQRST,ARRHYTHMIA,DEMO ecg
    class USERDB,LOCAL_REPORTS,REPORT_INDEX,ECG_SETTINGS,SESSION_LOG,CRASH_LOG,APP_CONFIG storage
    class CLOUD_UPLOAD,AUTOSYNC,OFFLINE_QUEUE,S3_REPORTS,S3_USERS cloud
    class CHATBOT,AI_INSIGHTS,AI_ENHANCE ai
    class LOGGER,CONFIG,CRASH,CONSTANTS,SESSION,SETTINGS,HEART_VIZ,REPORT_GEN util
```

---

## Component Layer Diagram

```mermaid
graph LR
    subgraph "Presentation Layer"
        UI[User Interface<br/>PyQt5 Widgets]
        SPLASH_UI[Splash Screen]
        LOGIN_UI[Login/Register]
        DASH_UI[Dashboard]
        ECG_UI[ECG Test Page]
        EXPAND_UI[Expanded Lead View]
        ADMIN_UI[Admin Panel]
        CHAT_UI[Chatbot Dialog]
    end
    
    subgraph "Application Layer"
        AUTH[Authentication<br/>auth/]
        DASH_CTRL[Dashboard Controller<br/>dashboard/]
        ECG_CTRL[ECG Controller<br/>ecg/]
        AI[AI Services<br/>ai/]
    end
    
    subgraph "Business Logic Layer"
        SIGNAL[Signal Processing]
        ANALYSIS[ECG Analysis]
        METRICS[Metrics Calculation]
        ARRHYTHMIA[Arrhythmia Detection]
        REPORT[Report Generation]
    end
    
    subgraph "Data Layer"
        LOCAL_DB[(Local JSON Files)]
        CLOUD_DB[(AWS S3 Cloud)]
        CONFIG_DB[(Config Files)]
        LOGS_DB[(Log Files)]
    end
    
    subgraph "Infrastructure Layer"
        LOGGING[Logging System]
        ERROR[Error Handling]
        SETTINGS[Settings Manager]
        SYNC[Cloud Sync]
        QUEUE[Offline Queue]
    end
    
    subgraph "Hardware Layer"
        SERIAL_HW[Serial Port<br/>ECG Hardware]
        DEMO_HW[Demo Data Generator]
    end
    
    UI --> AUTH
    UI --> DASH_CTRL
    UI --> ECG_CTRL
    UI --> AI
    
    AUTH --> LOCAL_DB
    DASH_CTRL --> METRICS
    ECG_CTRL --> SIGNAL
    SIGNAL --> ANALYSIS
    ANALYSIS --> METRICS
    METRICS --> ARRHYTHMIA
    REPORT --> LOCAL_DB
    
    ECG_CTRL --> SERIAL_HW
    ECG_CTRL --> DEMO_HW
    
    DASH_CTRL --> SYNC
    SYNC --> QUEUE
    QUEUE --> CLOUD_DB
    
    AUTH --> CLOUD_DB
    REPORT --> CLOUD_DB
    
    LOGGING --> LOGS_DB
    ERROR --> LOGS_DB
    SETTINGS --> CONFIG_DB
    
    classDef presentation fill:#E3F2FD,stroke:#2196F3
    classDef application fill:#F3E5F5,stroke:#9C27B0
    classDef business fill:#E8F5E9,stroke:#4CAF50
    classDef data fill:#FFF3E0,stroke:#FF9800
    classDef infrastructure fill:#EFEBE9,stroke:#795548
    classDef hardware fill:#FCE4EC,stroke:#E91E63
    
    class UI,SPLASH_UI,LOGIN_UI,DASH_UI,ECG_UI,EXPAND_UI,ADMIN_UI,CHAT_UI presentation
    class AUTH,DASH_CTRL,ECG_CTRL,AI application
    class SIGNAL,ANALYSIS,METRICS,ARRHYTHMIA,REPORT business
    class LOCAL_DB,CLOUD_DB,CONFIG_DB,LOGS_DB data
    class LOGGING,ERROR,SETTINGS,SYNC,QUEUE infrastructure
    class SERIAL_HW,DEMO_HW hardware
```

---

## Data Flow Diagram

```mermaid
sequenceDiagram
    participant User
    participant Main
    participant Auth
    participant Dashboard
    participant ECG
    participant Hardware
    participant Analysis
    participant Report
    participant Cloud
    
    User->>Main: Launch Application
    Main->>Main: Initialize Core Systems
    Main->>Auth: Show Login
    User->>Auth: Enter Credentials
    Auth->>Auth: Validate User
    Auth-->>User: Authentication Success
    
    Auth->>Dashboard: Open Dashboard
    Dashboard->>Dashboard: Initialize Session
    Dashboard->>Cloud: Start Auto-Sync
    Dashboard-->>User: Show Dashboard
    
    User->>Dashboard: Click "ECG Lead Test 12"
    Dashboard->>ECG: Open ECG Test Page
    
    User->>ECG: Start Acquisition
    ECG->>Hardware: Read Serial Data
    Hardware-->>ECG: Raw ECG Data (8 channels)
    
    ECG->>Analysis: Process Signal
    Analysis->>Analysis: Filter (8-stage)
    Analysis->>Analysis: Calculate 12 Leads
    Analysis->>Analysis: Detect R-peaks (Pan-Tompkins)
    Analysis->>Analysis: Calculate Metrics
    Analysis-->>ECG: Processed Data & Metrics
    
    ECG-->>Dashboard: Update Dashboard Metrics
    ECG-->>User: Display 12-Lead ECG
    
    User->>ECG: Click on Lead
    ECG->>ECG: Open Expanded View
    ECG->>Analysis: PQRST Analysis
    Analysis-->>ECG: Wave Components
    ECG->>Analysis: Arrhythmia Detection
    Analysis-->>ECG: Arrhythmia Results
    ECG-->>User: Show Detailed Analysis
    
    User->>ECG: Generate Report
    ECG->>Report: Create PDF Report
    Report->>Report: Capture 10s Lead Images
    Report->>Report: Generate PDF
    Report->>Report: Save Local Copy
    Report-->>User: Report Saved
    
    Report->>Cloud: Upload Report (Background)
    Cloud-->>Report: Upload Success
    
    User->>Dashboard: Sign Out
    Dashboard->>Dashboard: End Session
    Dashboard->>Auth: Return to Login
```

---

## File System Structure

```
modularecg/
├── src/                                    # Main source code
│   ├── main.py                             # ⭐ Application Entry Point
│   ├── splash_screen.py                    # Splash screen UI
│   │
│   ├── auth/                               # 🔐 Authentication Module
│   │   ├── sign_in.py                      # User sign-in logic
│   │   └── sign_out.py                     # User sign-out logic
│   │
│   ├── dashboard/                          # 📊 Dashboard Module
│   │   ├── dashboard.py                    # Main dashboard UI & controller
│   │   ├── admin_reports.py                # Admin panel for cloud reports
│   │   └── chatbot_dialog.py               # AI chatbot interface
│   │
│   ├── ecg/                                # 💓 ECG Processing Module
│   │   ├── twelve_lead_test.py             # 12-lead ECG visualization
│   │   ├── expanded_lead_view.py           # Detailed lead analysis
│   │   ├── demo_manager.py                 # Demo mode data generator
│   │   ├── recording.py                    # Recording panel UI
│   │   ├── pan_tompkins.py                 # R-peak detection algorithm
│   │   └── ecg_report_generator.py         # PDF report generation
│   │
│   ├── ai/                                 # 🤖 AI Services Module
│   │   ├── dashboard_insights.py           # Dashboard AI insights
│   │   └── report_enhancer.py              # Report enhancement AI
│   │
│   ├── utils/                              # 🔧 Utility Module
│   │   ├── crash_logger.py                 # Crash logging & email alerts
│   │   ├── session_recorder.py             # Session recording
│   │   ├── cloud_uploader.py               # AWS S3 upload handler
│   │   ├── auto_sync_service.py            # Background sync service
│   │   ├── offline_queue.py                # Offline data queue
│   │   ├── settings_manager.py             # Settings persistence
│   │   ├── heartbeat_widget.py             # 3D heart visualization
│   │   ├── helpers.py                      # Helper functions
│   │   └── backend_api.py                  # Backend API client
│   │
│   ├── core/                               # ⚙️ Core System Module
│   │   ├── logging_config.py               # Logging configuration
│   │   ├── exceptions.py                   # Custom exceptions
│   │   ├── constants.py                    # Application constants
│   │   └── validation.py                   # Data validation
│   │
│   └── config/                             # 📝 Configuration Module
│       └── settings.py                     # Config loader & manager
│
├── reports/                                # 📄 Generated Reports
│   ├── *.pdf                               # PDF ECG reports
│   ├── index.json                          # Recent reports index
│   ├── metrics.json                        # Metrics data
│   ├── upload_log.json                     # Cloud upload log
│   └── sessions/                           # Session recordings
│       └── session_*.jsonl                 # Session data files
│
├── logs/                                   # 📝 Application Logs
│   ├── crash_logs.json                     # Crash reports
│   └── session_*.log                       # Session logs
│
├── assets/                                 # 🎨 Static Assets
│   ├── v.gif                               # Background animation
│   ├── v1.png                              # Login image
│   └── heart.png                           # Heart icon
│
├── users.json                              # 👥 User database
├── ecg_settings.json                       # ⚙️ ECG settings
├── ecg_app.log                             # 📊 Application log
├── .env                                    # 🔒 Environment variables (AWS)
└── requirements.txt                        # 📦 Python dependencies
```

---

## Technology Stack

### Frontend / UI Layer
- **PyQt5**: Desktop GUI framework
- **PyQtGraph**: Real-time ECG plotting
- **Matplotlib**: Static plot generation for reports

### Backend / Processing Layer
- **NumPy**: Numerical computations
- **SciPy**: Signal processing (filtering, peak detection)
- **Pan-Tompkins Algorithm**: R-peak detection

### Data Storage
- **JSON**: Local data storage (users, settings, reports index)
- **JSONL**: Session recording
- **AWS S3**: Cloud storage for reports and user data

### Report Generation
- **ReportLab**: PDF generation
- **Matplotlib**: ECG waveform images

### Communication
- **PySerial**: Serial port communication with ECG hardware
- **SMTP (Gmail)**: Email crash reports
- **Boto3**: AWS S3 API client

### AI & Analytics
- **OpenAI API**: Chatbot and insights generation
- **Custom Algorithms**: Arrhythmia detection, PQRST analysis

---

## Key Design Patterns

### 1. **MVC Pattern**
- **Model**: Data classes, JSON files, cloud storage
- **View**: PyQt5 UI components
- **Controller**: Dashboard, ECG controller classes

### 2. **Observer Pattern**
- Real-time ECG data updates to dashboard
- Metrics updates trigger UI refresh

### 3. **Singleton Pattern**
- Crash logger instance
- Settings manager instance
- Cloud uploader instance

### 4. **Factory Pattern**
- ECG signal generator (Hardware/Demo)

### 5. **Queue Pattern**
- Offline queue for cloud sync
- Background upload service

---

## Security Features

1. **Authentication**: Username/password login with serial ID
2. **Admin Access**: Separate admin credentials
3. **Environment Variables**: AWS credentials in `.env`
4. **Session Management**: Session recording and timeout
5. **Data Encryption**: HTTPS for cloud uploads

---

## Scalability Features

1. **Modular Architecture**: Easy to add new ECG leads or analysis algorithms
2. **Cloud Storage**: AWS S3 for unlimited report storage
3. **Offline Support**: Queue system for offline uploads
4. **Background Sync**: Non-blocking cloud synchronization
5. **Responsive UI**: Adapts to different screen sizes

---

## Error Handling

1. **Crash Logger**: Automatic crash detection and logging
2. **Email Notifications**: Admin email alerts on crashes
3. **Graceful Degradation**: Fallback modes for missing modules
4. **User Feedback**: Clear error messages and recovery options
5. **Session Recovery**: Auto-save and recovery mechanisms

---

## Performance Optimizations

1. **Real-time Processing**: 20-60 FPS ECG rendering
2. **8-Stage Filtering**: Medical-grade signal quality
3. **Cached Reports**: Recent reports index for fast access
4. **Background Sync**: Non-blocking cloud uploads
5. **Lazy Loading**: Components loaded on-demand

---

## Future Enhancements

1. ✅ **Completed**: 12-lead ECG, real-time analysis, cloud sync
2. 🔄 **In Progress**: AI insights, advanced arrhythmia detection
3. 📅 **Planned**: Multi-user collaboration, telemedicine integration
4. 💡 **Ideas**: Mobile app, wearable device integration

---

**Last Updated**: December 3, 2025  
**Version**: 2.0  
**Maintainer**: Deckmount Team

