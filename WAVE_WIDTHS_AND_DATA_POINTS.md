# ECG Wave Widths, Rolling Windows, and Data Points

## Overview
This document details the wave widths, rolling window sizes, and data points for the 12-lead ECG display at different wave speeds (12.5mm/s, 25mm/s, 50mm/s).

---

## 1. 12-Lead Grid Display (12 Boxes)

### Display Configuration
- **Layout**: 4×3 grid (4 rows × 3 columns)
- **Total Leads**: 12 leads displayed simultaneously
- **Buffer Size**: `HISTORY_LENGTH = 10,000` samples
- **Buffer Duration**: At 500 Hz = 20 seconds of data stored

### Wave Width Calculation
The visible window width is calculated based on wave speed:
- **Baseline**: 25 mm/s → 3 seconds visible
- **Formula**: `seconds_to_show = 3.0 × (25.0 / wave_speed)`

### Wave Speeds and Data Points (at 500 Hz sampling rate)

#### **12.5 mm/s (Slow Speed)**
- **Time Window**: 6.0 seconds
- **Data Points**: 6.0 × 500 = **3,000 samples**
- **Visible Duration**: Longest view (6 seconds)
- **Use Case**: Detailed analysis, slow heart rates

#### **25.0 mm/s (Standard Speed)**
- **Time Window**: 3.0 seconds  
- **Data Points**: 3.0 × 500 = **1,500 samples**
- **Visible Duration**: Standard clinical view (3 seconds)
- **Use Case**: Standard ECG interpretation (most common)

#### **50.0 mm/s (Fast Speed)**
- **Time Window**: 1.5 seconds
- **Data Points**: 1.5 × 500 = **750 samples**
- **Visible Duration**: Shortest view (1.5 seconds)
- **Use Case**: Fast heart rates, rhythm analysis

### Lead Colors (12-Lead Grid)
Each lead has a unique color for identification:

| Lead | Color Code | Color Name |
|------|------------|------------|
| I | `#00ff99` | Cyan-Green |
| II | `#ff0055` | Magenta-Pink |
| III | `#0099ff` | Blue |
| aVR | `#ff9900` | Orange |
| aVL | `#cc00ff` | Purple-Magenta |
| aVF | `#00ccff` | Cyan-Blue |
| V1 | `#ffcc00` | Gold-Yellow |
| V2 | `#00ffcc` | Turquoise |
| V3 | `#ff6600` | Orange-Red |
| V4 | `#6600ff` | Purple |
| V5 | `#00b894` | Teal-Green |
| V6 | `#ff0066` | Pink-Red |

---

## 2. Expanded Lead View

### Window Size
- **Initial Size**: 80% of screen size (responsive)
- **Minimum Size**: 960 × 600 pixels
- **Default Size**: 1280 × 720 pixels (if screen info unavailable)
- **Maximum**: Can be maximized to full screen

### Display Features
- **Single Lead Focus**: One lead displayed at a time
- **Detailed Analysis**: PQRST wave detection and analysis
- **Zoom Controls**: Amplification from 0.1x to 10x
- **History View**: Sliding window with 10-second default view
- **Color**: Uses lead-specific color from `LEAD_COLORS`

### Rolling Window
- **Default View Duration**: 10 seconds
- **At 500 Hz**: 10 × 500 = **5,000 samples** visible
- **Sliding**: User can scroll through history
- **Buffer**: Uses full `HISTORY_LENGTH` (10,000 samples = 20 seconds)

---

## 3. Rolling Window Size Summary

### Buffer Configuration
- **Total Buffer**: `HISTORY_LENGTH = 10,000` samples
- **At 500 Hz**: 20 seconds of continuous data
- **Memory**: ~400 KB per lead (10,000 × 4 bytes float32)
- **Total Memory**: ~4.8 MB for all 12 leads

### Visible Window by Wave Speed

| Wave Speed | Time Window | Data Points (500 Hz) | Data Points (250 Hz) |
|------------|-------------|---------------------|---------------------|
| **12.5 mm/s** | 6.0 seconds | **3,000** | **1,500** |
| **25.0 mm/s** | 3.0 seconds | **1,500** | **750** |
| **50.0 mm/s** | 1.5 seconds | **750** | **375** |

### Calculation Formula
```python
baseline_seconds = 3.0  # At 25 mm/s
seconds_to_show = baseline_seconds × (25.0 / wave_speed)
window_samples = seconds_to_show × sampling_rate
```

---

## 4. Overlay Views (12:1 and 6:2)

### 12:1 Overlay (All 12 Leads Stacked)
- **Layout**: Single column, all 12 leads vertically stacked
- **Color**: `#00ff00` (Green) for all leads
- **Buffer Size**: Same as main grid (calculated from wave speed)
- **Update Rate**: 100ms (10 FPS)

### 6:2 Overlay (Two Columns)
- **Left Column**: Limb leads (I, II, III, aVR, aVL, aVF)
- **Right Column**: Chest leads (V1, V2, V3, V4, V5, V6)
- **Color**: `#00ff00` (Green) for all leads
- **Buffer Size**: Same as main grid (calculated from wave speed)
- **Update Rate**: 100ms (10 FPS)

---

## 5. Data Points Calculation Examples

### Example 1: 12.5 mm/s at 500 Hz
```
Time Window = 3.0 × (25.0 / 12.5) = 6.0 seconds
Data Points = 6.0 × 500 = 3,000 samples
```

### Example 2: 25.0 mm/s at 500 Hz
```
Time Window = 3.0 × (25.0 / 25.0) = 3.0 seconds
Data Points = 3.0 × 500 = 1,500 samples
```

### Example 3: 50.0 mm/s at 500 Hz
```
Time Window = 3.0 × (25.0 / 50.0) = 1.5 seconds
Data Points = 1.5 × 500 = 750 samples
```

### Example 4: Different Sampling Rates
If hardware sends at 250 Hz instead of 500 Hz:

| Wave Speed | Time Window | Data Points (250 Hz) |
|------------|-------------|---------------------|
| 12.5 mm/s | 6.0 seconds | 1,500 |
| 25.0 mm/s | 3.0 seconds | 750 |
| 50.0 mm/s | 1.5 seconds | 375 |

---

## 6. Visual Width Reference

### Report Generation (PDF)
- **Graph Width**: 33 boxes × 5mm = **165mm**
- **Time Calculation**: `Time = 165mm / wave_speed`

| Wave Speed | Time Window (Report) |
|------------|---------------------|
| 12.5 mm/s | 165 / 12.5 = **13.2 seconds** |
| 25.0 mm/s | 165 / 25.0 = **6.6 seconds** |
| 50.0 mm/s | 165 / 50.0 = **3.3 seconds** |

**Note**: Report uses different calculation (165mm width) vs. live display (3-second baseline).

---

## 7. Summary Table

| Feature | 12.5 mm/s | 25.0 mm/s | 50.0 mm/s |
|---------|-----------|-----------|-----------|
| **Time Window** | 6.0 sec | 3.0 sec | 1.5 sec |
| **Data Points (500 Hz)** | 3,000 | 1,500 | 750 |
| **Data Points (250 Hz)** | 1,500 | 750 | 375 |
| **Use Case** | Detailed analysis | Standard view | Fast rhythms |
| **Report Width** | 13.2 sec | 6.6 sec | 3.3 sec |

---

## 8. Technical Notes

### Sampling Rate Detection
- **Hardware Default**: 500 Hz
- **Fallback**: 250 Hz (if detection fails)
- **Detection**: Automatic via `SamplingRateCalculator`
- **Update Interval**: Every 5 seconds

### Buffer Management
- **Circular Buffer**: `np.roll()` for efficient updates
- **Memory Efficient**: Float32 (4 bytes per sample)
- **Real-time**: Updates every 30-33ms (~30 FPS)

### Color Coding
- **12-Lead Grid**: Each lead has unique color
- **Overlay Views**: All leads use green (`#00ff00`)
- **Expanded View**: Uses lead-specific color

---

**Last Updated**: Based on code analysis of `twelve_lead_test.py` and `expanded_lead_view.py`

