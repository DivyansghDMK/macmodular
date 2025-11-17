# Algorithms Used in ECG Software

## Overview
This document describes the core algorithms and signal processing techniques implemented in the ECG monitoring software.

---

## 1. **Pan-Tompkins QRS Detection Algorithm**

**Purpose**: Detect R-peaks (QRS complexes) in ECG signals for heart rate calculation.

**Implementation**: `src/ecg/pan_tompkins.py`

**Algorithm Steps**:
1. **Bandpass Filtering** (5-15 Hz): Removes noise and baseline wander using Butterworth filter
2. **Differentiation**: Enhances QRS slopes using first derivative
3. **Squaring**: Emphasizes large slopes and makes all values positive
4. **Moving Window Integration** (150ms window): Smooths the signal to create an envelope
5. **Adaptive Peak Detection**: Uses adaptive threshold (mean + 0.5 × std) to find peaks
6. **Minimum Distance Filter**: Ensures peaks are at least 200ms apart (prevents false positives)

**Usage**: Primary R-peak detection for accurate heart rate measurement.

---

## 2. **Multi-Strategy Adaptive Peak Detection**

**Purpose**: Accurately detect R-peaks across wide heart rate range (40-300 BPM).

**Implementation**: `src/dashboard/dashboard.py` (lines 1421-1487), `src/ecg/twelve_lead_test.py`

**Algorithm Strategy**:
- **Conservative Detection** (40-120 BPM): 500ms minimum distance between peaks
- **Normal Detection** (100-180 BPM): 300ms minimum distance between peaks  
- **Tight Detection** (160-300 BPM): 200ms minimum distance between peaks

**Selection Logic**:
1. Run all three detection strategies simultaneously
2. Calculate BPM and standard deviation for each strategy
3. Select the strategy with lowest standard deviation (most consistent)
4. Filter R-R intervals to physiologically reasonable range (200-2000ms)

**Benefits**: Adapts automatically to patient's heart rate without manual configuration.

---

## 3. **Median Filter Smoothing (Anti-Flickering)**

**Purpose**: Stabilize BPM readings and prevent UI flickering.

**Implementation**: `src/dashboard/dashboard.py` (lines 1508-1518), `src/ecg/twelve_lead_test.py`

**Algorithm**:
- Maintains a rolling buffer of last 5 BPM readings
- Uses **median** (not mean) to reject outliers
- Only updates display if change ≥ 2 BPM (update threshold)

**Benefits**: 
- Eliminates rapid fluctuations in displayed heart rate
- Provides stable, clinically useful readings
- Reduces false alarms from transient noise

---

## 4. **Butterworth Bandpass Filtering**

**Purpose**: Remove noise and enhance ECG signal quality.

**Implementation**: Multiple files (dashboard.py, expanded_lead_view.py, ecg_report_generator.py)

**Filter Specifications**:
- **Frequency Range**: 0.5-40 Hz (standard ECG bandwidth)
- **Filter Type**: 4th-order Butterworth bandpass
- **Method**: Forward-backward filtering (`filtfilt`) for zero-phase distortion

**Why This Range**:
- **0.5 Hz**: Removes baseline wander and DC offset
- **40 Hz**: Removes high-frequency noise (muscle artifacts, 50/60Hz power line interference)

---

## 5. **Arrhythmia Detection Algorithms**

**Purpose**: Automatically identify cardiac rhythm abnormalities.

**Implementation**: `src/ecg/expanded_lead_view.py` (ArrhythmiaDetector class, lines 283-364)

### 5.1 **Atrial Fibrillation Detection**
- **Method**: Coefficient of Variation (CV) of R-R intervals
- **Formula**: `CV = std(RR) / mean(RR)`
- **Threshold**: CV > 0.15 indicates irregular rhythm
- **Requirement**: Minimum 10 R-peaks for reliable detection

### 5.2 **Ventricular Tachycardia Detection**
- **Method**: Heart rate and R-R interval regularity
- **Criteria**: 
  - Mean HR > 120 BPM
  - Low R-R interval variation (std < 40ms) - indicates regular fast rhythm

### 5.3 **Premature Ventricular Contractions (PVCs)**
- **Method**: Premature beat detection with compensatory pause
- **Logic**:
  - R-R interval < 80% of mean R-R interval (premature beat)
  - Followed by R-R interval > 120% of mean (compensatory pause)
- **Requirement**: Minimum 5 R-peaks

### 5.4 **Sinus Bradycardia**
- **Method**: Heart rate threshold
- **Criteria**: Mean HR < 60 BPM

### 5.5 **Sinus Tachycardia**
- **Method**: Heart rate threshold
- **Criteria**: Mean HR > 100 BPM

### 5.6 **Normal Sinus Rhythm (NSR)**
- **Method**: Heart rate and R-R interval stability
- **Criteria**:
  - Mean HR between 60-100 BPM
  - R-R interval standard deviation < 120ms (regular rhythm)

---

## 6. **PQRST Wave Detection**

**Purpose**: Identify all components of ECG waveform (P, Q, R, S, T waves).

**Implementation**: `src/ecg/expanded_lead_view.py` (PQRSTAnalyzer class)

**Algorithm**:
1. **R-Peak Detection**: Uses Pan-Tompkins algorithm
2. **Q-Point Detection**: Minimum value before R-peak (within 20ms window)
3. **S-Point Detection**: Minimum value after R-peak (within 20ms window)
4. **P-Wave Detection**: Maximum value before Q-point (within 100-200ms window)
5. **T-Wave Detection**: Maximum value after S-point (within 150-350ms window)

**Usage**: Detailed waveform analysis for clinical interpretation.

---

## 7. **QRS Axis Calculation**

**Purpose**: Determine electrical axis of heart (important for diagnosis).

**Implementation**: `src/dashboard/dashboard.py`, `src/ecg/twelve_lead_test.py`

**Algorithm**:
- Uses **Lead I** (horizontal) and **Lead aVF** (vertical) vectors
- **Formula**: `Axis = arctan2(Lead aVF, Lead I) × 180/π`
- Result in degrees: -180° to +180°

**Clinical Significance**:
- Normal: -30° to +90°
- Left Axis Deviation: < -30°
- Right Axis Deviation: > +90°

---

## 8. **ECG Metrics Calculation**

**Purpose**: Calculate clinically relevant parameters from ECG signal.

**Implementation**: `src/dashboard/dashboard.py` (calculate_live_ecg_metrics)

### 8.1 **PR Interval**
- **Method**: Time from P-wave peak to R-peak
- **Range**: 120-200ms (normal)
- **Calculation**: Average of first 3 detected intervals

### 8.2 **QRS Duration**
- **Method**: Time from Q-point to S-point
- **Range**: 40-200ms (normal)
- **Calculation**: Average of first 5 detected complexes

### 8.3 **ST Interval**
- **Method**: Time from S-point to T-wave end
- **Calculation**: Detects T-wave end by finding return to baseline

---

## 9. **SciPy Peak Detection**

**Purpose**: General-purpose peak detection using scientific Python library.

**Implementation**: Used throughout codebase via `scipy.signal.find_peaks`

**Parameters Used**:
- **height**: Minimum peak height (adaptive threshold)
- **distance**: Minimum distance between peaks (varies by BPM range)
- **prominence**: Minimum peak prominence (reduces false positives)

**Advantages**: 
- Fast, optimized C implementation
- Highly configurable
- Robust to noise

---

## 10. **Signal Processing Libraries**

### **NumPy**
- Array operations, mathematical computations
- Used for all signal processing calculations

### **SciPy Signal Processing**
- `butter()`: Butterworth filter design
- `filtfilt()`: Zero-phase filtering
- `find_peaks()`: Peak detection
- `lfilter()`: Linear filtering

### **PyQtGraph**
- Real-time plotting and visualization
- High-performance graphics rendering

---

## Summary: How to Describe as Features

### **For Technical Audiences**:
"Our ECG software implements the **Pan-Tompkins QRS detection algorithm** with **multi-strategy adaptive peak detection** to accurately measure heart rates from 40-300 BPM. We use **4th-order Butterworth bandpass filtering** (0.5-40 Hz) for noise reduction, and **median filter smoothing** to prevent display flickering. Our **arrhythmia detection system** uses statistical analysis of R-R intervals to identify 5 types of cardiac rhythm abnormalities."

### **For Clinical/Non-Technical Audiences**:
"Our software uses advanced signal processing algorithms to automatically detect heartbeats and calculate heart rate with high accuracy. The system adapts to different heart rates (from very slow to very fast) and filters out noise to provide clear, stable readings. It can automatically identify irregular heart rhythms and provide clinical insights."

### **Key Algorithm Highlights**:
1. ✅ **Pan-Tompkins Algorithm** - Industry-standard QRS detection
2. ✅ **Adaptive Multi-Strategy Detection** - Works across 40-300 BPM range
3. ✅ **Median Filter Smoothing** - Stable, flicker-free readings
4. ✅ **Butterworth Bandpass Filtering** - Professional-grade noise reduction
5. ✅ **Statistical Arrhythmia Detection** - Automated rhythm analysis
6. ✅ **PQRST Wave Analysis** - Complete waveform component detection

---

## References

- **Pan-Tompkins Algorithm**: Pan, J., & Tompkins, W. J. (1985). A real-time QRS detection algorithm. IEEE Transactions on Biomedical Engineering, 32(3), 230-236.
- **Butterworth Filter**: Standard digital signal processing technique for biomedical applications
- **SciPy Signal Processing**: Scientific Python library for signal analysis


