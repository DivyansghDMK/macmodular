# Windows Compatibility Fix for BPM, PR, and Wave Plotting

## Problem
BPM, PR interval, and wave plotting not working correctly on Windows (working fine on macOS).

## Root Causes Identified

### 1. **Timer Precision Differences**
- Windows QTimer has lower precision (~15ms) compared to macOS (~1ms)
- Timer interval of 33ms may not fire consistently on Windows
- **Fix**: Use 30ms interval on Windows (compensates for precision loss)

### 2. **Inconsistent Sampling Rate Fallbacks**
- Multiple places use `186.5 Hz` fallback (old hardware-specific value)
- Dashboard uses `250.0 Hz` standard fallback
- Mismatch causes incorrect BPM/PR calculations
- **Fix**: Standardize all fallbacks to `250.0 Hz`

### 3. **PR Interval Calculation**
- Depends on sampling rate accuracy
- If sampling rate detection fails, PR calculation uses wrong rate
- **Fix**: Use consistent 250 Hz fallback everywhere

### 4. **PyQtGraph Plotting Updates**
- Windows may need more frequent updates due to timer precision
- Plot widgets may not refresh properly with slower timers
- **Fix**: Faster timer interval (30ms) ensures smooth plotting

## Fixes Applied

### ✅ Timer Interval Optimization (Line ~5203)
```python
# CROSS-PLATFORM: Timer interval optimization
import sys
if sys.platform.startswith('win'):
    timer_interval = 30  # ~33 FPS for Windows
else:
    timer_interval = 33  # ~30 FPS for macOS/Linux
self.timer.start(timer_interval)
```

### ✅ Sampling Rate Fallback Standardization
Changed all instances of `186.5 Hz` → `250.0 Hz` for consistency:
- Line ~6984: Overlay plotting
- Line ~1948: PR interval calculation  
- Line ~2594: Wave amplitude calculation
- Line ~2774: QRS axis calculation
- Line ~3003: QRS axis from median
- Line ~6221: Overlay buffer calculation
- Line ~6292: 12:1 overlay filtering
- Line ~7320: Main plotting wave-speed scaling

### ✅ Fixed Sampling Rate Bug (Line ~6989)
```python
# OLD (BUG):
sampling_rate = float(self.sampler.sampling_rate)  # Wrong!

# NEW (FIXED):
sampling_rate = float(self.sampling_rate)  # Correct
```

## Testing on Windows

### What to Check:

1. **Console Output**:
   ```
   [Windows] ECGTestPage - Starting timer with 30ms interval (optimized for Windows)
   ```
   Should see Windows-specific timer message.

2. **BPM Accuracy**:
   - Test at: 72, 100, 150, 200, 250, 300 BPM
   - Should match machine setting within ±2 BPM
   - Should be identical to macOS results

3. **PR Interval**:
   - Should calculate correctly (typically 120-200 ms)
   - Should update in real-time
   - Should match macOS values

4. **Wave Plotting**:
   - Waves should update smoothly (~30 FPS)
   - No gaps or stuttering
   - Should match macOS smoothness

5. **Sampling Rate Detection**:
   - Check console for sampling rate messages
   - Should detect hardware rate OR use 250 Hz fallback
   - All calculations should use same rate

## Additional Windows-Specific Considerations

### PyQt5 Timer Precision
- Windows: ~15ms minimum precision
- macOS/Linux: ~1ms precision
- **Solution**: Use 30ms interval on Windows (2x minimum precision)

### Serial Port Reading
- Windows COM ports may have different timing
- Check if `SerialStreamReader` reads data at correct rate
- Verify `sampler.sampling_rate` is being calculated correctly

### PyQtGraph Performance
- Windows may need more frequent `update()` calls
- Ensure `plot_widget.update()` is called after data changes
- Check if `enableAutoRange()` is causing issues

## Files Modified

1. **`src/ecg/twelve_lead_test.py`**:
   - Timer interval: Windows-specific optimization
   - Sampling rate fallbacks: Standardized to 250 Hz
   - Fixed sampling rate bug in overlay plotting

2. **`src/dashboard/dashboard.py`** (already fixed):
   - Cross-platform BPM calculation
   - Unified 250 Hz fallback

## Expected Behavior After Fix

### Windows (Should Now Work):
- ✅ Timer fires consistently at ~30ms intervals
- ✅ BPM matches machine setting (same as macOS)
- ✅ PR interval calculates correctly
- ✅ Waves plot smoothly without gaps
- ✅ All metrics update in real-time

### macOS (Still Works):
- ✅ Timer fires at 33ms intervals (unchanged)
- ✅ BPM accurate (unchanged)
- ✅ PR interval accurate (unchanged)
- ✅ Waves plot smoothly (unchanged)

## Troubleshooting

### If BPM still incorrect:
1. Check console for sampling rate messages
2. Verify `sampler.sampling_rate` is being set
3. Check if hardware detection is working
4. Compare with macOS values at same machine setting

### If PR still incorrect:
1. Verify PR calculation uses correct sampling rate
2. Check if median beat is being built correctly
3. Ensure enough R-peaks detected (need ≥8 for median beat)

### If waves not plotting:
1. Check timer is active: `self.timer.isActive()`
2. Verify data is being read: `len(self.data[0]) > 0`
3. Check PyQtGraph widgets are created: `len(self.plot_widgets) == 12`
4. Verify update_plots() is being called

## Next Steps

1. Test on Windows with hardware device
2. Compare BPM/PR values with macOS
3. Verify wave plotting smoothness
4. Check console for any Windows-specific warnings
5. Report any remaining issues

