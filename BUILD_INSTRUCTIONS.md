# Building ECG Monitor Executable

This guide explains how to create a standalone executable (.exe) file for the ECG Monitor application.

## Prerequisites

1. **Python 3.8+** installed
2. **All dependencies** installed:
   ```bash
   pip install -r requirements.txt
   pip install pyinstaller
   ```

## Quick Build (Windows)

### Option 1: Using Batch Script (Easiest)
```bash
build_exe.bat
```

### Option 2: Using Python Script
```bash
python build_exe.py
```

### Option 3: Using PyInstaller Directly
```bash
pyinstaller ECG_Monitor.spec
```

## Build Methods

### Method 1: One-File Executable (Recommended)
Creates a single `.exe` file that contains everything.

**Pros:**
- Single file to distribute
- Easy to share

**Cons:**
- Slower startup time
- Larger file size (~200-300 MB)

**Build Command:**
```bash
pyinstaller ECG_Monitor.spec
```

### Method 2: One-Directory Executable
Creates a folder with the executable and dependencies.

**Pros:**
- Faster startup
- Smaller individual files

**Cons:**
- Multiple files to distribute
- Need to keep folder structure

**Build Command:**
Modify `ECG_Monitor.spec` and change:
```python
exe = EXE(
    ...
    onefile=False,  # Change this
    ...
)
```

## Output Location

After building, the executable will be in:
- **Windows**: `dist/ECG_Monitor.exe`
- **macOS**: `dist/ECG_Monitor.app`
- **Linux**: `dist/ECG_Monitor`

## What's Included

The executable includes:
- ✅ All Python dependencies (PyQt5, NumPy, SciPy, etc.)
- ✅ All assets (images, sounds, GIFs)
- ✅ All application modules
- ✅ Configuration templates

## Troubleshooting

### Issue: "Module not found" errors
**Solution:** Add the missing module to `hiddenimports` in `ECG_Monitor.spec`

### Issue: Assets not found
**Solution:** Check that `assets` folder is included in `datas` in the spec file

### Issue: Large file size
**Solution:** This is normal - PyQt5 and scientific libraries are large. Consider using UPX compression (already enabled in spec file).

### Issue: Antivirus false positives
**Solution:** This is common with PyInstaller executables. You may need to:
1. Add exception in antivirus
2. Code sign the executable (requires certificate)
3. Submit to antivirus vendors for whitelisting

## Testing the Executable

1. Navigate to `dist/` folder
2. Run `ECG_Monitor.exe`
3. Test all features:
   - Sign in/out
   - ECG test page
   - Dashboard
   - Report generation
   - Hardware connection (if available)

## Distribution

To distribute the application:
1. Copy `dist/ECG_Monitor.exe` to your distribution location
2. Include a README with:
   - System requirements
   - Installation instructions
   - Known issues
   - Support contact

## File Size Optimization

The executable will be large (~200-300 MB) because it includes:
- Python interpreter
- PyQt5 framework
- NumPy, SciPy, Matplotlib
- All dependencies

This is normal and expected for PyInstaller executables.

## Recent Changes Included

This build includes:
- ✅ BPM calculation fixes (Windows/macOS compatibility)
- ✅ Packet loss detection for 500Hz hardware
- ✅ Enhanced buffer overflow protection
- ✅ Cross-platform sampling rate detection
- ✅ Windows-specific logging and validation

## Support

If you encounter build issues:
1. Check that all dependencies are installed
2. Verify Python version (3.8+)
3. Check PyInstaller version (latest recommended)
4. Review build logs in `build/` folder



