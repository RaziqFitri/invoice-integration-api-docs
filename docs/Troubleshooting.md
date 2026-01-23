# Troubleshooting Guide

Common issues and solutions for the Invoice Integration API.

---

## 🔍 Quick Diagnosis

### Is the API running?

Test: `http://your-server:5000/swagger`

- ✅ **Loads:** API is running
- ❌ **Doesn't load:** API is down - contact IT

---

## ❌ Common Errors

### 1. "Creditor mapping not found for: XXXX"

**Error Example:**
```json
{
  "message": "Creditor mapping not found for: 4000/Z999"
}
```

**Cause:** Supplier code not in configuration

**Solution:**
1. Note the supplier code (e.g., 4000/Z999)
2. Contact IT with supplier name and code
3. Wait for IT to add mapping
4. Resubmit invoice

---

### 2. "Invoice already submitted"

**Error Example:**
```json
{
  "message": "Invoice already submitted. IFCAP JournalID: 5001"
}
```

**Cause:** Invoice was previously sent to IFCAP

**Solution:**
1. Check IFCAP using JournalID 5001
2. If duplicate, no action needed
3. If genuine, contact IT

---

### 3. "Cannot submit cancelled invoice"

**Cause:** Invoice is cancelled in AutoCount

**Solution:**
1. Open invoice in AutoCount
2. Verify it's not cancelled
3. If incorrectly cancelled, uncancelled it
4. Resubmit

---

### 4. "No line items found"

**Cause:** Invoice has no detail lines

**Solution:**
1. Open invoice in AutoCount
2. Add line items
3. Save invoice
4. Resubmit

---

### 5. "ToDate must be greater than FromDate"

**Cause:** Invalid date range

**Example of error:**
```
?fromDate=2025-01-31&toDate=2025-01-01
```

**Solution:**
```
?fromDate=2025-01-01&toDate=2025-01-31
```

---

## 🔧 PowerShell Script Issues

### "Script cannot be loaded - execution policy"

**Solution:**
```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

---

### "Cannot connect to API"

**Solutions:**
1. Verify API URL in script (line 8)
2. Check API is running
3. Test URL in browser
4. Check firewall settings

---

## ⏰ Task Scheduler Issues

### Task shows "Running" forever

**Solution:**
1. Task Manager → End `powershell.exe`
2. Check API status
3. Run task again

---

### Task doesn't run at scheduled time

**Solutions:**
1. Computer must be on at 6 PM
2. Uncheck power conditions
3. Enable "Wake computer"
4. Verify Task Scheduler service running

---

## 📊 Performance Issues

### Batch taking too long

**Solution:**
- Break into smaller batches
- Submit 50-100 invoices at a time
- Use off-peak hours

---

## 📞 When to Contact Support

Contact IT if:
- ❌ API not responding
- ❌ Database connection errors
- ❌ Recurring creditor mapping errors
- ❌ Task Scheduler not working
- ❌ Unexplained failures

---

**Always include:**
1. Error message
2. Invoice DocKey/DocNo
3. Screenshot
4. Date/time of issue
```

4. Commit with message: `Add troubleshooting guide`

---

### **STEP 6: Add Images (Optional)**

1. Click **"Add file"** → **"Upload files"**

2. Create folder by typing: `images/` in the upload area

3. Upload screenshots:
   - Task Scheduler screenshots
   - Swagger UI screenshots
   - Example responses

4. Commit with message: `Add documentation images`

---

### **STEP 7: Enable GitHub Pages (Optional - Makes it a Website)**

1. Go to repository **Settings**

2. Scroll to **"Pages"** section (left sidebar)

3. Under **"Source"**, select:
   - Branch: `main`
   - Folder: `/ (root)`

4. Click **"Save"**

5. Wait 1-2 minutes

6. Your docs will be available at:
```
   https://your-username.github.io/invoice-integration-api-docs/
