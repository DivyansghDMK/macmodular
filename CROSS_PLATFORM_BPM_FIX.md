# Cross-Platform BPM Calculation Fix (Windows/macOS/Linux)

## Problem
BPM calculations were working correctly on macOS but showing incorrect values on Windows.

## Root Cause
1. **Inconsistent Sampling Rate Fallbacks**: 
   - `_calculate_heart_rate_legacy()` used `186.5 Hz` fallback
   - `calculate_live_ecg_metrics()` used `250.0 Hz` fallback
   - Windows sampling rate detection might fail silently, causing wrong fallback

2. **Platform-Specific Timer Precision**: 
   - Windows `time.monotonic()` might have different precision than macOS
   - Sampling rate calculator depends on timer accuracy

3. **Silent Failures**: 
   - No Windows-specific error logging when sampling rate detection fails

## Fixes Applied

### 1. Unified Sampling Rate Fallback
- **Both functions now use `250.0 Hz` as standard fallback** (consistent across all platforms)
- Matches ECG test page default for consistency

### 2. Enhanced Windows Detection & Logging
- Added Windows-specific error messages when sampling rate detection fails
- Debug output shows platform tag (`[Windows]` vs `[macOS/Linux]`)
- First 5 calculations print debug info (increased from 3 for better Windows debugging)

### 3. Cross-Platform Validation
- Added explicit validation for invalid sampling rates
- Windows-specific warnings when fallback is used
- Ensures `fs > 0` and `np.isfinite(fs)` before calculations

## Code Changes

### `_calculate_heart_rate_legacy()` (Line ~1578)
```python
# OLD: fs = 186.5  # Different fallback
# NEW: fs = 250.0  # Standard fallback for all platforms
```

### `calculate_live_ecg_metrics()` (Line ~1690)
- Enhanced Windows debugging
- Platform tag in debug output
- Consistent fallback logic

## Testing on Windows

### What to Check:
1. **Console Output**: Look for these messages:
   ```
   🔍 [Windows] BPM Calculation - Sampling rate: 250.0 Hz, Signal length: X samples
   ```
   If you see warnings like:
   ```
   ⚠️ Windows: Sampling rate detection failed: ..., using fallback 250.0 Hz
   ```
   This means the hardware sampling rate isn't being detected correctly.

2. **BPM Accuracy**: 
   - Test at known rates: 72, 100, 150, 200, 250, 300 BPM
   - Compare with machine setting
   - Should match within ±2 BPM

3. **Sampling Rate Detection**:
   - Check if `ecg_test_page.sampler.sampling_rate` is being set correctly
   - Should be > 10 Hz (typically 80-500 Hz depending on hardware)

## Troubleshooting Windows Issues

### If BPM is still incorrect:

1. **Check Hardware Sampling Rate**:
   ```python
   # In ECG test page, check:
   print(f"Sampling rate: {self.sampler.sampling_rate}")
   ```

2. **Verify Timer Precision**:
   - Windows `time.monotonic()` should work, but check if `QTimer` intervals are consistent
   - ECG update timer should be ~50ms (20 Hz)

3. **Check NumPy/SciPy Versions**:
   ```bash
   pip list | grep -E "numpy|scipy"
   ```
   Ensure versions match between macOS and Windows:
   - numpy >= 1.20.0
   - scipy >= 1.7.0

4. **Verify Signal Length**:
   - Need at least 200 samples for BPM calculation
   - At 250 Hz, that's ~0.8 seconds of data
   - Check console output: `Signal length: X samples`

## Expected Behavior

### macOS (Working):
- Sampling rate detected correctly (usually 250-500 Hz)
- BPM matches machine setting
- No warnings in console

### Windows (Should Now Work):
- Sampling rate detected OR uses 250 Hz fallback
- BPM matches machine setting (same as macOS)
- May show warnings if hardware detection fails, but calculations still correct

## Additional Notes

- **Demo Mode**: Uses fixed sampling rate, should work identically on both platforms
- **Hardware Mode**: Depends on serial port reading rate, may vary between platforms
- **Timer Intervals**: QTimer should work identically, but Windows may have slightly different precision

## Files Modified
- `src/dashboard/dashboard.py`: 
  - `_calculate_heart_rate_legacy()`: Unified fallback to 250 Hz
  - `calculate_live_ecg_metrics()`: Enhanced Windows debugging

## Next Steps if Issues Persist

1. Add more detailed Windows logging in `SamplingRateCalculator`
2. Check if serial port reading rate differs on Windows
3. Verify PyQt5 timer precision on Windows
4. Consider platform-specific sampling rate calibration

