# ECG Monitor Application

A comprehensive ECG monitoring application with 12-lead ECG analysis, real-time metrics calculation, and dashboard visualization.

## Features

- **12-Lead ECG Analysis**: Real-time ECG signal processing and visualization
- **Medical-Grade Signal Filtering**: Advanced filtering system for smooth, clean ECG waves like professional medical devices
- **Expanded Lead View**: Detailed analysis page for individual ECG leads with PQRST labeling
- **Live Metrics Calculation**: Heart Rate, PR Interval, QRS Duration, QTc Interval, QRS Axis, ST Segment
- **Arrhythmia Detection**: Automatic detection of various cardiac arrhythmias
- **Dashboard Interface**: Clean, modern dashboard with live metric updates
- **User Authentication**: Sign-in/sign-out functionality
- **PDF Report Generation**: Generate comprehensive ECG reports
- **Recent Reports Panel**: In-app list of the last 10 generated PDF reports with Open action
- **Dual Save Reports**: When generating a report, it saves to your chosen location (e.g., Downloads) and a managed copy is stored in `reports/` for history
- **Crash Logger & Email Reporting**: Hidden diagnostic system accessible via triple-click on heart rate metric
- **Background GIF Support**: Animated background on sign-in screen
- **Real-time Data Processing**: Live ECG data acquisition and processing from hardware

## Project Structure

```
modularecg/
├── src/                    # Main application source code
│   ├── main.py            # Application entry point
│   ├── auth/              # Authentication modules
│   ├── dashboard/         # Dashboard and UI components
│   ├── ecg/               # ECG processing and analysis
│   ├── utils/             # Utility functions and helpers
│   └── nav_*.py           # Navigation components
├── assets/                # Images, GIFs, and other resources
├── requirements.txt       # Python dependencies
├── launch_app.bat        # Windows batch launcher
├── launch_app.ps1        # PowerShell launcher
├── users.json            # User data storage
└── clutter/              # Archived files and backups
```

## Installation

1. **Clone the repository**
2. **Install dependencies**:
   ```bash
   pip install -r requirements.txt
   ```
3. **Configure Email Reporting (Optional)**:
   - Copy `email_config_template.txt` to `.env`
   - Edit `.env` with your Gmail credentials
   - Generate a Gmail App Password for security
   - See `email_config_template.txt` for detailed instructions

## Crash Logger & Email Reporting

The application includes a hidden diagnostic system for troubleshooting and crash reporting:

### Accessing the Crash Logger
- **Triple-click** the heart rate metric on the dashboard
- A diagnostic dialog will open showing:
  - Session statistics (duration, errors, crashes, memory usage)
  - Email configuration status
  - Crash logs and error reports
  - Options to send reports via email or clear logs

### Email Configuration
To enable email reporting on other computers:

1. **Copy the template**: `cp email_config_template.txt .env`
2. **Edit credentials**: Open `.env` and replace placeholder values
3. **Gmail Setup**:
   - Enable 2-Factor Authentication on your Google account
   - Go to Google Account → Security → App passwords
   - Generate a password for "Mail"
   - Use this app password (not your regular Gmail password)
4. **Restart** the application

### Features
- **Automatic crash detection** and logging
- **System information** collection
- **Session statistics** tracking
- **Email reporting** with detailed diagnostics
- **Log management** (view, clear, export)

## Running the Application

### Option 1: Using Batch File (Windows)
```bash
launch_app.bat
```

### Option 2: Using PowerShell Script
```bash
.\launch_app.ps1
```

### Option 3: Direct Python Execution
```bash
cd src
python main.py
```

## Usage

1. **Launch the application** using one of the methods above
2. **Sign in** with your credentials
3. **Navigate to the dashboard** to view live ECG metrics
4. **Click "ECG Lead Test 12"** to open the 12-lead ECG analysis page
5. **View live metrics** including Heart Rate, PR Interval, QRS Duration, QTc Interval, QRS Axis, and ST Segment
6. **Click on any ECG lead** to open the expanded lead view for detailed analysis
7. **View PQRST labeling** and detailed metrics for individual leads
8. **Monitor arrhythmia detection** in real-time

### Generating Reports
- Click "Generate Report" on the dashboard
- Choose a filename and location (e.g., Downloads)
- The app also stores a managed copy in `reports/` and updates `reports/index.json`
- The new report appears instantly in the dashboard "Recent Reports" panel; click Open to view

## ECG Metrics

The application calculates and displays the following metrics in real-time:

- **Heart Rate**: Beats per minute (BPM)
- **PR Interval**: Time from P-wave to QRS complex (ms)
- **QRS Duration**: Duration of QRS complex (ms)
- **QTc Interval**: Corrected QT interval using Bazett's formula (ms)
- **QRS Axis**: Electrical axis of the heart (degrees)
- **ST Segment**: ST elevation/depression (mV)

## Technical Details

- **Framework**: PyQt5 for GUI
- **Plotting**: PyQtGraph for real-time ECG visualization
- **Signal Processing**: NumPy and SciPy for ECG analysis
- **Medical-Grade Filtering**: Advanced filtering pipeline including Wiener filter, Gaussian smoothing, adaptive median filtering
- **Report Generation**: Matplotlib for static plots and PDF generation
- **Real-time Processing**: Live data acquisition and processing from ECG hardware
- **Arrhythmia Detection**: Pan-Tompkins algorithm for R-peak detection and cardiac rhythm analysis

## File Organization

- **Modular Architecture**: Clean separation of concerns with dedicated modules for core functionality, configuration, and utilities
- **Core Modules**: Centralized error handling, logging, validation, and configuration management
- **Clutter Folder**: All unused files, test scripts, and deprecated code moved to `clutter/` directory
- **Clean Structure**: Organized directory structure with proper Python package hierarchy
- **Documentation**: Comprehensive documentation including technical specs and project structure

## Recent Updates

### Codebase Refactoring and Modularization
- **Modular Architecture**: Complete restructuring with dedicated modules for core functionality, configuration, and utilities
- **Error Handling**: Comprehensive error handling with custom exception classes and graceful fallbacks
- **Logging System**: Centralized logging with rotation, performance monitoring, and debug information
- **Configuration Management**: Centralized configuration system with JSON file support and runtime updates
- **Data Validation**: Input validation for ECG signals, range checking for metrics, and signal quality assessment
- **Clean Organization**: Unused files moved to clutter directory, proper Python package structure

### Medical-Grade ECG Filtering System
- **Advanced Filtering Pipeline**: Implemented 8-stage filtering system for professional medical device-quality signals
- **Wiener Filter**: Statistical noise reduction optimized for ECG signals
- **Gaussian Smoothing**: Multi-stage smoothing for clean waveform appearance
- **Adaptive Median Filtering**: Dynamic noise removal based on signal characteristics
- **Real-time Smoothing**: Individual data point smoothing for live data processing

### Expanded Lead View
- **Detailed Analysis**: Click any ECG lead to open expanded analysis view
- **PQRST Labeling**: Automatic detection and labeling of cardiac waveform components
- **Enhanced Metrics**: Comprehensive metrics display with improved visibility
- **Arrhythmia Detection**: Real-time detection of various cardiac arrhythmias
- **Responsive UI**: Optimized layout and sizing for better user experience

### Signal Quality Improvements
### Dashboard Recent Reports & Report Management
- Added "Recent Reports" panel with scrollbar and app-themed styling
- Reports are saved both to the selected path and to `reports/`
- `reports/index.json` maintains metadata for the last 10 reports for quick access
- One-click Open action from the dashboard

- **Smooth Waveforms**: Medical-grade signal processing for clean, professional appearance
- **Stable Baseline**: Reduced drift and improved signal stability
- **Sharp R-peaks**: Enhanced peak detection for accurate heart rate calculation
- **Noise Reduction**: Comprehensive noise filtering for clear signal visualization

## Documentation

### 📚 Complete Documentation Library

- **[TECHNICAL_DOCUMENTATION.md](TECHNICAL_DOCUMENTATION.md)** - Comprehensive technical guide
  - System architecture and technology stack
  - Core modules and API reference
  - ECG signal processing algorithms
  - Cloud integration (AWS S3)
  - Performance optimization
  - Deployment and troubleshooting

- **[PROJECT_STATUS_UPDATE_NOV2025.md](PROJECT_STATUS_UPDATE_NOV2025.md)** - Latest project status
  - Completed features (52+)
  - Pending features (40+)
  - Recent achievements
  - Development timeline
  - Cost analysis

- **[PROJECT_STRUCTURE.md](PROJECT_STRUCTURE.md)** - File organization guide
  - Directory structure
  - Module descriptions
  - Code organization

- **[AWS_S3_STEP_BY_STEP_GUIDE.md](AWS_S3_STEP_BY_STEP_GUIDE.md)** - Cloud setup guide
  - AWS account creation
  - S3 bucket configuration
  - IAM user setup
  - Step-by-step instructions

- **[DOCUMENTATION.md](DOCUMENTATION.md)** - Hardware specifications
  - Timer intervals and reading rates
  - Serial communication specs
  - Performance metrics

- **[CALCULATED_VS_PLACEHOLDER_VALUES.md](CALCULATED_VS_PLACEHOLDER_VALUES.md)** - Metrics reference
  - Which values are calculated
  - Which are placeholders
  - Implementation status

### 🎯 Quick Start Guides

**For Developers:**
1. Read [TECHNICAL_DOCUMENTATION.md](TECHNICAL_DOCUMENTATION.md) - Complete system overview
2. Check [PROJECT_STRUCTURE.md](PROJECT_STRUCTURE.md) - File organization
3. Review [PROJECT_STATUS_UPDATE_NOV2025.md](PROJECT_STATUS_UPDATE_NOV2025.md) - Current status

**For Admins:**
1. Read [AWS_S3_STEP_BY_STEP_GUIDE.md](AWS_S3_STEP_BY_STEP_GUIDE.md) - Cloud setup
2. Configure `.env` file with AWS credentials
3. Access Admin Panel with `admin`/`adminsd` credentials

**For Users:**
1. Install dependencies: `pip install -r requirements.txt`
2. Run: `python src/main.py`
3. Login or register to start monitoring

## Cloud Integration (AWS S3)

### Features
- ✅ Automatic report upload every 5 seconds
- ✅ User signup data backup
- ✅ Admin panel for report management
- ✅ Offline queue (uploads when online)
- ✅ Presigned URLs for secure downloads

### Setup
1. Create AWS account and S3 bucket
2. Create IAM user with S3 permissions
3. Copy `.env.example` to `.env`
4. Add AWS credentials to `.env`
5. See [AWS_S3_STEP_BY_STEP_GUIDE.md](AWS_S3_STEP_BY_STEP_GUIDE.md) for details

### Cost
- **100 reports:** ~$0.003/month
- **10,000 reports:** ~$0.28/month
- **100,000 reports:** ~$2.80/month

## Admin Panel

Access with credentials: `admin` / `adminsd`

### Features
- **Reports Tab:**
  - View all S3 reports (PDF + JSON)
  - Download reports
  - Copy presigned URLs
  - Search and filter
  - Summary metrics

- **Users Tab:**
  - View all registered users
  - Search users
  - Link users to reports
  - User details panel

## Performance

- **Real-time ECG:** 20-60 FPS
- **Metric Updates:** Sub-100ms latency
- **Report Generation:** < 5 seconds
- **Cloud Upload:** < 2 seconds
- **Admin Panel Load:** < 1 second (cached)

## Version History

- **v2.0** (Nov 5, 2025) - Admin panel overhaul, performance optimization
- **v1.3** (Nov 1, 2025) - AWS S3 integration, PDF reports
- **v1.2** (Oct 25, 2025) - 12-lead display, hardware support
- **v1.1** (Oct 15, 2025) - Dashboard, authentication
- **v1.0** (Oct 1, 2025) - Initial release

## Upcoming Features (v2.1)

- 🔄 Guest Mode (no login required)
- 🔄 Email/OTP authentication
- 🔄 Role-based permissions
- 🔄 Email report delivery
- 🔄 Two-factor authentication (2FA)

See [PROJECT_STATUS_UPDATE_NOV2025.md](PROJECT_STATUS_UPDATE_NOV2025.md) for complete roadmap.

## Support

### Documentation
- **Technical Issues:** See [TECHNICAL_DOCUMENTATION.md](TECHNICAL_DOCUMENTATION.md) → Troubleshooting
- **Cloud Setup:** See [AWS_S3_STEP_BY_STEP_GUIDE.md](AWS_S3_STEP_BY_STEP_GUIDE.md)
- **Feature Status:** See [PROJECT_STATUS_UPDATE_NOV2025.md](PROJECT_STATUS_UPDATE_NOV2025.md)

### Bug Reports
- GitHub Issues: https://github.com/YourUsername/modularecg/issues
- Email: support@example.com

### Community
- Discord: https://discord.gg/ecgmonitor
- Slack: #ecg-monitor

## License

MIT License - See LICENSE file for details

## Disclaimer

**Medical Use:** This software is for educational and research purposes only. NOT FDA-approved for clinical diagnosis. Always consult qualified healthcare professionals for medical advice.

---

**Last Updated:** November 5, 2025  
**Version:** 2.0  
**Status:** ✅ Production Ready