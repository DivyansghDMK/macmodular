# Bug Report: Dashboard vs Report BPM Mismatch

**Date:** November 6, 2025  
**Reported by:** Divyansh  
**Severity:** MEDIUM  
**Status:** 🔴 IDENTIFIED - NEEDS FIX

---

## 🐛 Issue

**Problem:** Dashboard shows different BPM (heart rate) than the report PDF.

**Root Cause:** Data key mismatch between dashboard and report generator.

---

## 🔍 Analysis

### Dashboard receives: `Heart_Rate`

**Location:** `src/dashboard/dashboard.py` line 1829

```python
def update_ecg_metrics(self, intervals):
    if 'Heart_Rate' in intervals and intervals['Heart_Rate'] is not None:
        self.metric_labels['heart_rate'].setText(
            f"{int(round(intervals['Heart_Rate']))} bpm"
        )
```

### Report generator expects: `HR_avg`

**Location:** `src/ecg/ecg_report_generator.py` lines 902, 1082

```python
# Get real ECG data from dashboard
HR = data.get('HR_avg', 70)  # ❌ WRONG KEY! Dashboard doesn't send 'HR_avg'
PR = data.get('PR', 192) 
QRS = data.get('QRS', 93)
```

**Fallback value:** When `HR_avg` is not found, it defaults to **70 bpm**

This is why the report always shows a different (likely 70) BPM than the dashboard!

---

## 📊 Data Flow Problem

```
ECG Test Page
    ↓ (sends 'Heart_Rate')
Dashboard.update_ecg_metrics()
    ↓ (stores as 'Heart_Rate')
Report Generator
    ↓ (looks for 'HR_avg' - NOT FOUND!)
Uses fallback: 70 bpm ❌ WRONG!
```

---

## ✅ Solution

### Option 1: Make dashboard send `HR_avg` (Recommended)

**Location:** Where report is generated from dashboard

Find where the dashboard calls `generate_ecg_report()` and ensure it passes:

```python
data = {
    'HR_avg': current_heart_rate,  # Add this
    'Heart_Rate': current_heart_rate,  # Keep for compatibility
    'PR': pr_interval,
    'QRS': qrs_duration,
    ...
}
```

### Option 2: Change report generator to use `Heart_Rate`

**Location:** `src/ecg/ecg_report_generator.py` lines 902, 1082

```python
# OLD (WRONG):
HR = data.get('HR_avg', 70)

# NEW (CORRECT):
HR = data.get('Heart_Rate', data.get('HR_avg', 70))
# Try 'Heart_Rate' first, fallback to 'HR_avg', then 70
```

---

## 🔧 Recommended Fix (Option 2 - Safer)

Change the report generator to check both keys:

**File:** `src/ecg/ecg_report_generator.py`

**Line 902:**
```python
# OLD:
HR = data.get('HR_avg', 70)

# NEW:
HR = data.get('Heart_Rate', data.get('HR_avg', 70))
```

**Line 1082:**
```python
# OLD:
HR = data.get('HR_avg', 70)

# NEW:
HR = data.get('Heart_Rate', data.get('HR_avg', 70))
```

**Also check line 464** (default data):
```python
# Add both keys for compatibility:
data = {
    "HR": 0,
    "HR_avg": 0,  # ✅ Already exists
    "Heart_Rate": 0,  # ✅ Add this
    ...
}
```

---

## 🧪 Testing

After fix:
1. Run app
2. Start Demo Mode or connect hardware
3. Note the BPM on dashboard (e.g., 72 bpm)
4. Generate report
5. **Verify:** Report PDF should show **same 72 bpm**, not 70!

---

## 📝 Additional Keys to Check

The report also uses these keys - verify they match what dashboard sends:

| Report Key | Dashboard Key | Match? | Fix Needed? |
|-----------|---------------|--------|-------------|
| `HR_avg` | `Heart_Rate` | ❌ NO | ✅ YES |
| `PR` | `PR` | ✅ YES | ❌ NO |
| `QRS` | `QRS` | ✅ YES | ❌ NO |
| `QT` | `QT` | ✅ YES | ❌ NO |
| `QTc` | `QTc` | ✅ YES | ❌ NO |
| `ST` | `ST` | ✅ YES | ❌ NO |
| `QRS_axis` | `QRS_axis` | ✅ YES | ❌ NO |

**Only `HR_avg` vs `Heart_Rate` is mismatched!**

---

## 🎯 Priority

**Priority:** MEDIUM (not critical, but confusing for users)

**Estimated Fix Time:** 5 minutes

**Who Should Fix:** Divyansh (Backend Lead)

**When to Fix:** Week 1 (Nov 6-7) - Along with other bug fixes

---

## 💡 Prevention

Add a utility function to normalize data keys before passing to report generator:

```python
def normalize_ecg_data(data):
    """Normalize ECG data keys for report generation"""
    # Ensure both Heart_Rate and HR_avg exist
    if 'Heart_Rate' in data and 'HR_avg' not in data:
        data['HR_avg'] = data['Heart_Rate']
    elif 'HR_avg' in data and 'Heart_Rate' not in data:
        data['Heart_Rate'] = data['HR_avg']
    
    return data
```

---

**Prepared by:** AI Code Review  
**Date:** November 6, 2025  
**Assign to:** Divyansh (Backend Lead)  
**Deadline:** November 7, 2025

---

**Status:** 🔴 OPEN - Awaiting implementation

