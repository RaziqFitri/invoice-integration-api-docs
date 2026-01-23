# Task Scheduler Setup Guide

**⏱️ Setup Time:** 15 minutes  
**👤 For:** System Administrators, IT Support

---

## 📖 Overview

This guide shows you how to set up Windows Task Scheduler to automatically submit invoices every day at 6 PM.

---

## 🎯 What You'll Achieve

- ✅ Automatic daily invoice submission at 6 PM
- ✅ Submits yesterday's invoices automatically
- ✅ Detailed logs for monitoring
- ✅ Email notifications on failures (optional)

---

## 📋 Prerequisites

- Windows Server or Windows 10/11
- Administrator access
- Invoice Integration API running
- PowerShell execution policy configured

---

## 🚀 Step-by-Step Setup

### **STEP 1: Prepare Folders**

1. Open **File Explorer**
2. Navigate to `C:\`
3. Create folder: `InvoiceAPI`
4. Inside `C:\InvoiceAPI`, create:
   - `Scripts`
   - `Logs`

**Final structure:**
```
C:\
└── InvoiceAPI\
    ├── Scripts\
    └── Logs\
```

---

### **STEP 2: Create PowerShell Script**

1. Open **Notepad**

2. Copy this script:

```powershell
# ============================================
# Daily Invoice Submission Script
# Submits yesterday's invoices to IFCAP
# ============================================

# Configuration
$apiUrl = "http://localhost:5000"  # ⚠️ CHANGE THIS
$logPath = "C:\InvoiceAPI\Logs\DailySubmit_$(Get-Date -Format 'yyyyMMdd').log"

# Create log directory if it doesn't exist
$logDir = Split-Path $logPath
if (!(Test-Path $logDir)) {
    New-Item -ItemType Directory -Path $logDir -Force | Out-Null
}

# Function to write log
function Write-Log {
    param($Message)
    $timestamp = Get-Date -Format "yyyy-MM-dd HH:mm:ss"
    $logMessage = "[$timestamp] $Message"
    Write-Host $logMessage
    Add-Content -Path $logPath -Value $logMessage
}

# Start logging
Write-Log "========================================="
Write-Log "Starting Daily Invoice Submission"
Write-Log "========================================="

try {
    # Calculate yesterday's date
    $yesterday = (Get-Date).AddDays(-1)
    $fromDate = $yesterday.ToString("yyyy-MM-dd")
    $toDate = $yesterday.ToString("yyyy-MM-dd")
    
    Write-Log "Submitting invoices from: $fromDate to $toDate"
    
    # Build API URL
    $endpoint = "$apiUrl/api/invoices/batch/submit-by-date?fromDate=$fromDate&toDate=$toDate"
    
    Write-Log "Calling API: $endpoint"
    
    # Call the API
    $response = Invoke-RestMethod -Uri $endpoint -Method POST -ContentType "application/json"
    
    # Log results
    Write-Log "API Response Received"
    Write-Log "Total Invoices: $($response.totalInvoices)"
    Write-Log "Success Count: $($response.successCount)"
    Write-Log "Failed Count: $($response.failedCount)"
    Write-Log "Summary: $($response.summary)"
    
    # Log each result
    if ($response.results -and $response.results.Count -gt 0) {
        Write-Log ""
        Write-Log "Detailed Results:"
        foreach ($result in $response.results) {
            if ($result.success) {
                Write-Log "  ✓ SUCCESS - DocKey: $($result.docKey) | DocNo: $($result.docNo) | JournalID: $($result.journalID)"
            } else {
                Write-Log "  ✗ FAILED  - DocKey: $($result.docKey) | DocNo: $($result.docNo) | Error: $($result.message)"
            }
        }
    }
    
    Write-Log ""
    Write-Log "Daily submission completed successfully"
    Write-Log "========================================="
    
} catch {
    Write-Log "ERROR: $($_.Exception.Message)"
    Write-Log "Stack Trace: $($_.ScriptStackTrace)"
    Write-Log "========================================="
    exit 1
}

exit 0
```

3. **Save as:**
   - Location: `C:\InvoiceAPI\Scripts\`
   - Filename: `SubmitDailyInvoices.ps1`
   - Save as type: **All Files (*.*)**

4. **⚠️ IMPORTANT:** Edit line 8 to your API URL

---

### **STEP 3: Test the Script**

1. Open **PowerShell as Administrator**

2. Run:
```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

3. Navigate to script:
```powershell
cd C:\InvoiceAPI\Scripts
```

4. Run the script:
```powershell
.\SubmitDailyInvoices.ps1
```

5. **Verify:**
   - Check console output
   - Check log file: `C:\InvoiceAPI\Logs\DailySubmit_YYYYMMDD.log`

---

### **STEP 4: Create Scheduled Task**

#### **4A: Open Task Scheduler**

1. Press `Win + R`
2. Type: `taskschd.msc`
3. Press Enter

---

#### **4B: Create Task**

1. Click **"Create Task..."** (not "Create Basic Task")

---

#### **4C: General Tab**

- **Name:** `Invoice Daily Submission`
- **Description:** `Automatically submits yesterday's invoices to IFCAP at 6 PM daily`
- **Security options:**
  - ☑ Run whether user is logged on or not
  - ☑ Run with highest privileges
- **Configure for:** Windows 10 (or your OS version)

---

#### **4D: Triggers Tab**

1. Click **"New..."**
2. **Settings:**
   - Begin the task: `On a schedule`
   - Daily
   - Start: `Today at 6:00:00 PM`
   - Recur every: `1 days`
3. ☑ **Enabled**
4. Click **OK**

---

#### **4E: Actions Tab**

1. Click **"New..."**
2. **Settings:**
   - Action: `Start a program`
   - Program/script: `powershell.exe`
   - Add arguments:
     ```
     -ExecutionPolicy Bypass -NoProfile -File "C:\InvoiceAPI\Scripts\SubmitDailyInvoices.ps1"
     ```
   - Start in: `C:\InvoiceAPI\Scripts`
3. Click **OK**

---

#### **4F: Conditions Tab**

- ☐ **Uncheck** "Start only if on AC power"
- ☐ **Uncheck** "Stop if switches to battery"

---

#### **4G: Settings Tab**

- ☑ Allow task to be run on demand
- ☑ Run task as soon as possible after scheduled start is missed
- ☑ If task fails, restart every: `10 minutes`
- Attempt to restart up to: `3 times`

---

#### **4H: Save**

1. Click **OK**
2. Enter Windows credentials when prompted
3. Click **OK**

---

### **STEP 5: Test the Scheduled Task**

1. Find your task in the list
2. **Right-click** → **"Run"**
3. Check:
   - Last Run Result: `The operation completed successfully (0x0)`
   - Status: `Ready`
4. **Verify log:** `C:\InvoiceAPI\Logs\DailySubmit_YYYYMMDD.log`

---

## 📊 Monitoring

### Check Logs Daily

**Log Location:** `C:\InvoiceAPI\Logs\`

**Log Files:**
- `DailySubmit_20250123.log`
- `DailySubmit_20250124.log`
- etc.

### Log Retention

Logs are created daily. Consider:
- Archiving logs older than 30 days
- Setting up log rotation
- Regular review for patterns

---

## 🔧 Troubleshooting

### Issue: Task shows as "Running" forever

**Solution:**
1. Open Task Manager
2. End `powershell.exe` process
3. Check if API is responding
4. Run task again

---

### Issue: "Last Run Result: 0x1"

**Solutions:**
1. Check log file first
2. Ensure API is running
3. Test script manually
4. Check Windows Event Viewer

---

### Issue: Task doesn't run at 6 PM

**Solutions:**
1. Ensure computer is on at 6 PM
2. Check Conditions tab (uncheck power settings)
3. Enable "Wake computer to run this task"
4. Verify Task Scheduler service is running

---

## 📧 Optional: Email Notifications

To receive emails on failures, uncomment these lines in the script:

```powershell
# Around line 56
if ($response.failedCount -gt 0) {
    Send-MailMessage `
        -To "your-email@company.com" `
        -From "invoiceapi@company.com" `
        -Subject "Invoice Submission Failures - $fromDate" `
        -Body "Failed to submit $($response.failedCount) invoices. Check log: $logPath" `
        -SmtpServer "smtp.office365.com" `
        -Port 587 `
        -UseSsl `
        -Credential (Get-Credential)
}
```

**Update:**
- Email addresses
- SMTP server settings
- Port number

---

## ✅ Verification Checklist

- [ ] Folders created (`C:\InvoiceAPI\Scripts` and `Logs`)
- [ ] PowerShell script saved
- [ ] API URL updated in script
- [ ] Execution policy set
- [ ] Script tested manually
- [ ] Log file generated
- [ ] Task created in Task Scheduler
- [ ] All tabs configured
- [ ] Task tested (Right-click → Run)
- [ ] Next Run Time shows 6:00 PM

---

## 📅 Daily Workflow

```
6:00 PM → Task Scheduler triggers
       ↓
       PowerShell script runs
       ↓
       Submits yesterday's invoices
       ↓
       Logs results
       ↓
       Task completes
       ↓
       (Email sent if failures)
```

---

## 🔄 Maintenance

### Weekly
- Review log files for patterns
- Check for recurring errors

### Monthly
- Archive old logs
- Review success rate
- Update script if needed

### Quarterly
- Verify task is still scheduled
- Test manually
- Review with stakeholders

---

## 📞 Support

**Issues with Task Scheduler:**
- Contact IT Support

**Issues with Script:**
- Check logs first
- Verify API is running
- Contact IT Support with log file

---

**Document Version:** 1.0  
**Last Updated:** January 23, 2025
