# Troubleshooting Guide

**Version:** 1.1  
**Last Updated:** January 27, 2025

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

### 2. "Invoice already exists in IFCAP"

**Error Example:**
```json
{
  "message": "Invoice already exists in IFCAP (JournalID: 5001). Use /api/invoices/resubmit endpoint to replace it."
}
```

**Cause:** Invoice was previously sent to IFCAP

**Solution:**

**Option A - Invoice was edited in AutoCount:**
```
POST /api/invoices/resubmit?docKey=12345
```
This will replace the old invoice with updated data.

**Option B - No changes made:**
1. Check IFCAP using JournalID 5001
2. If data is correct, no action needed
3. If duplicate submission, no action needed

**How to decide:**
- Did you edit the invoice in AutoCount after submission? → Use resubmit
- Was this an accidental duplicate submission? → No action needed
- Not sure? → Check invoice in both AutoCount and IFCAP

---

### 3. 🆕 "Failed to delete old invoice from IFCAP"

**Error Example:**
```json
{
  "message": "Failed to delete old invoice (JournalID: 5001) from IFCAP"
}
```

**Cause:** During resubmission, the API couldn't delete the old invoice

**Possible Reasons:**
- Invoice is locked in IFCAP
- Database constraints preventing deletion
- Invoice already processed/approved in IFCAP
- Concurrent access issue

**Solution:**
1. **Stop immediately** - Don't retry
2. Contact IT Support with:
   - DocKey
   - Old JournalID (from error message)
   - Invoice DocNo
   - Screenshot of error
3. IT will manually investigate the IFCAP database
4. Do NOT edit the invoice further in AutoCount until resolved

---

### 4. "Cannot submit cancelled invoice"

**Cause:** Invoice is cancelled in AutoCount

**Solution:**
1. Open invoice in AutoCount
2. Verify it's not cancelled
3. If incorrectly cancelled, un-cancel it
4. Resubmit

---

### 5. "No line items found"

**Cause:** Invoice has no detail lines

**Solution:**
1. Open invoice in AutoCount
2. Add line items
3. Save invoice
4. Resubmit

---

### 6. "ToDate must be greater than FromDate"

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

### 7. "Invoice not found or not in pending status (Y)"

**Cause:** Trying to approve/reject an invoice that:
- Doesn't exist
- Hasn't been submitted yet (status N)
- Was already approved (status A)
- Was already rejected (status R)

**Solution:**
1. Check invoice status first:
```
GET /api/invoices/status/12345
```
2. Verify the status before attempting approve/reject

---

## 🆕 Resubmit-Specific Issues

### Issue: Resubmit doesn't update IFCAP data

**Symptoms:**
- Resubmit returns success
- But IFCAP still shows old data

**Solution:**
1. Check the response - verify new JournalID was created
2. Search IFCAP using the NEW JournalID (not the old one)
3. If still showing old data, contact IT

---

### Issue: Should I use submit or resubmit?

**Decision Tree:**

```
Is this the first submission of this invoice?
├── Yes → Use /submit
└── No → Was the invoice edited in AutoCount after submission?
    ├── Yes → Use /resubmit
    └── No → No action needed (already in IFCAP)
```

**Examples:**

✅ **Use `/submit`:**
- New invoice, never submitted before
- Invoice status is "N" in AutoCount

✅ **Use `/resubmit`:**
- Invoice amount was corrected after submission
- Line items were added/removed after submission
- Supplier code was changed after submission
- Any edit made in AutoCount after IFCAP submission

❌ **Don't resubmit if:**
- No changes were made
- Just checking status
- Invoice is already correct in IFCAP

---

### Issue: Multiple invoices need resubmission

**Scenario:** You edited 20 invoices in AutoCount after they were submitted.

**Solution:**

**Option A - Manual (Recommended for < 10 invoices):**
```
POST /api/invoices/resubmit?docKey=12340
POST /api/invoices/resubmit?docKey=12341
POST /api/invoices/resubmit?docKey=12342
```

**Option B - Track and batch (For 10+ invoices):**
1. Create a list of DocKeys that need resubmission
2. Use Postman or PowerShell script
3. Add 1-2 second delay between each resubmit
4. Log successes and failures

**Sample PowerShell Script:**
```powershell
$apiUrl = "http://your-server:5000/api/invoices/resubmit"
$docKeys = @(12340, 12341, 12342, 12343, 12344)

foreach ($docKey in $docKeys) {
    Write-Host "Resubmitting DocKey: $docKey"
    try {
        $response = Invoke-RestMethod -Uri "$apiUrl?docKey=$docKey" -Method Post
        Write-Host "✓ Success: $($response.message)" -ForegroundColor Green
    }
    catch {
        Write-Host "✗ Failed: $($_.Exception.Message)" -ForegroundColor Red
    }
    Start-Sleep -Seconds 2
}
```

---

### Issue: Don't know if invoice was edited

**Solution:**

**Check AutoCount:**
1. Open invoice in AutoCount
2. Check the modification date/time
3. Check your IFCAP submission logs
4. Compare timestamps

**If still unsure:**
1. Check status endpoint:
```
GET /api/invoices/status/12345
```
2. Compare data in AutoCount vs. IFCAP (using JournalID)
3. If data matches → No resubmit needed
4. If data differs → Use resubmit

---

## 📊 Performance Issues

### Issue: Batch taking too long

**Solution:**
- Break into smaller batches
- Submit 50-100 invoices at a time
- Use off-peak hours

---

### Issue: Resubmit is slow

**Cause:** Resubmit does two database operations (delete + insert)

**Normal Speed:**
- ~2-3 seconds per resubmit
- Slower than regular submit

**Solution:**
- This is expected behavior
- Don't resubmit multiple invoices simultaneously
- Add small delays between resubmits

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
1. Verify API URL in script
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
1. Computer must be on at scheduled time
2. Uncheck power conditions
3. Enable "Wake computer"
4. Verify Task Scheduler service running

---

## 🔄 Workflow Issues

### Issue: Batch submission shows mixed "already exists" errors

**Scenario:**
```json
{
  "totalInvoices": 20,
  "successCount": 15,
  "failedCount": 5,
  "results": [
    {
      "docKey": 12340,
      "success": false,
      "message": "Invoice already exists in IFCAP (JournalID: 5001). Use /api/invoices/resubmit endpoint to replace it."
    }
  ]
}
```

**Solution:**

**Step 1 - Identify which invoices were edited:**
1. Review the failed results
2. For each failed invoice, check AutoCount
3. Determine if it was edited after submission

**Step 2 - Resubmit only edited invoices:**
```
# Only resubmit if edited in AutoCount
POST /api/invoices/resubmit?docKey=12340
```

**Step 3 - Ignore genuine duplicates:**
- If not edited → No action needed
- Already correctly in IFCAP

---

### Issue: Lost track of which invoices were resubmitted

**Solution:**

**Check AutoCount Note field:**
```
Note: "IFCAP:5050 | "
```
The number after "IFCAP:" is the current JournalID.

**Check IFCAP creation date:**
- Resubmitted invoices will have recent creation dates
- Original submissions will have older dates

**Search by status:**
```
GET /api/invoices/search?status=Y
```
All resubmitted invoices will show status "Y" (Pending)

---

## 📞 When to Contact Support

Contact IT immediately if:

- ❌ API not responding
- ❌ Database connection errors
- ❌ "Failed to delete old invoice from IFCAP" error
- ❌ Resubmit succeeds but data doesn't update
- ❌ Recurring creditor mapping errors
- ❌ Task Scheduler not working
- ❌ Unexplained failures
- ❌ Multiple resubmit failures
- ❌ Data corruption suspected

---

## 📋 Support Checklist

**When reporting resubmit issues, include:**

1. ✅ **Endpoint used:** `/submit` or `/resubmit`
2. ✅ **DocKey and DocNo**
3. ✅ **Error message** (full JSON response)
4. ✅ **Old JournalID** (if applicable)
5. ✅ **New JournalID** (if applicable)
6. ✅ **Was invoice edited?** Yes/No
7. ✅ **What was edited?** (amount, line items, etc.)
8. ✅ **Screenshot** of error
9. ✅ **Date and time** of issue
10. ✅ **AutoCount Note field** content

---

## 🎯 Best Practices to Avoid Issues

### 1. Before Submitting
✅ Verify invoice data is correct  
✅ Check supplier mapping exists  
✅ Ensure invoice is not cancelled  
✅ Verify line items are complete

### 2. After Editing Invoices
✅ **Immediately resubmit** edited invoices  
✅ Don't wait until batch submission  
✅ Track which invoices were edited  
✅ Keep a log of resubmissions

### 3. During Batch Operations
✅ Review results carefully  
✅ Distinguish between duplicates and edited invoices  
✅ Handle "already exists" errors appropriately  
✅ Don't blindly resubmit all failures

### 4. Regular Monitoring
✅ Check daily submission logs  
✅ Track resubmission frequency  
✅ Monitor for recurring patterns  
✅ Report unusual behavior to IT

---

## 💡 Pro Tips

💡 **Tip 1:** If you edit an invoice, resubmit it immediately - don't wait

💡 **Tip 2:** Keep a simple log of edited invoices for your records

💡 **Tip 3:** Use status endpoint before deciding to resubmit

💡 **Tip 4:** "Already exists" error is not always a problem - check if edited first

💡 **Tip 5:** Document your workflow for handling edited invoices

💡 **Tip 6:** Test resubmit with 1-2 invoices before doing bulk resubmissions

---

## 📚 Additional Resources

- **API Reference:** Complete endpoint documentation
- **Quick Start Guide:** Step-by-step tutorials
- **FAQ:** Common questions and answers
- **Task Scheduler Setup:** Automation guide

---

**Document Version:** 1.1  
**Last Updated:** January 27, 2025  
**Changes:** Added resubmit troubleshooting section  
**Maintained by:** DT Department
