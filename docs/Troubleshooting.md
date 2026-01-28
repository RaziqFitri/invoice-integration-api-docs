# Troubleshooting Guide

**Version:** 1.2  
**Last Updated:** January 28, 2025

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

**Check Audit Log:**
```
GET /api/audit/logs?action=Submit&docKey=12345
```
Verify the error was logged.

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
```
GET /api/audit/invoice/12345
```
Check if there's a "Resubmit" action → Was edited  
No "Resubmit" action → Duplicate submission

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
- Database constraints
- Already processed/approved
- Concurrent access

**Solution:**
1. **Stop immediately** - Don't retry
2. Contact IT Support with:
   - DocKey
   - Old JournalID (from error)
   - Screenshot
3. IT will manually investigate
4. Do NOT edit invoice further until resolved

**Check Audit:**
```
GET /api/audit/invoice/12345
```
See if delete action failed.

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

## 🆕 Audit & Tracking Issues

### Issue: Cannot see audit logs

**Check Database:**
```sql
SELECT TOP 10 * FROM InvoiceAuditLog 
ORDER BY LogDate DESC;
```

**If empty:**
- Audit logging might not be working
- Contact IT

**If has data but API returns empty:**
- Check date filters
- Try without filters first

---

### Issue: Audit shows different result than expected

**Example:** API says success but audit shows failure

**Solution:**
```
GET /api/audit/invoice/{docKey}
```
Check the complete history. The audit log is the source of truth.

---

### Issue: Need to find who did something

**Solution:**
```
GET /api/audit/invoice/12345
```
Look at `userName` and `ipAddress` fields.

**Query Database:**
```sql
SELECT LogDate, Action, UserName, IPAddress, Details
FROM InvoiceAuditLog
WHERE DocKey = 12345
ORDER BY LogDate;
```

---

### Issue: Too many audit logs

**Solution:** Use filters to narrow down:
```
GET /api/audit/logs?fromDate=2025-01-28&action=Submit
```

Filter by:
- Date range
- Action type
- Specific invoice (docKey)

---

## 🔄 Resubmit-Specific Issues

### Issue: Resubmit doesn't update IFCAP data

**Symptoms:**
- Resubmit returns success
- But IFCAP still shows old data

**Solution:**
1. Check response - verify NEW JournalID
2. Search IFCAP using NEW JournalID (not old one)
3. If still showing old data, contact IT

**Verify in Audit:**
```
GET /api/audit/invoice/12345
```
Check if both Delete and Resubmit actions succeeded.

---

### Issue: Should I use submit or resubmit?

**Decision Tree:**
```
First submission of this invoice?
├── Yes → Use /submit
└── No → Was edited after submission?
    ├── Yes → Use /resubmit
    └── No → No action needed
```

**Check Audit to Verify:**
```
GET /api/audit/invoice/12345
```
- If has "Submit" action → Already submitted
- If has "Resubmit" action → Already resubmitted

---

### Issue: Multiple invoices need resubmission

**Scenario:** You edited 20 invoices after submission

**Solution:**

**Option A - Manual (< 10 invoices):**
```
POST /api/invoices/resubmit?docKey=12340
POST /api/invoices/resubmit?docKey=12341
POST /api/invoices/resubmit?docKey=12342
```

**Option B - Track and verify:**
1. Create list of DocKeys
2. Resubmit each
3. Use audit to verify:
```
GET /api/audit/resubmits?fromDate=2025-01-28&toDate=2025-01-28
```

---

### Issue: Don't know if invoice was edited

**Solution:**

**Check Audit History:**
```
GET /api/audit/invoice/12345
```
- Look for "Resubmit" action
- If found → Was edited
- If not found → Not edited

**Compare Data:**
1. Check AutoCount data
2. Check IFCAP data (using JournalID from status)
3. If different → Use resubmit

---

## 📊 Performance Issues

### Issue: Batch taking too long

**Solution:**
- Break into smaller batches (50-100 invoices)
- Use off-peak hours
- Submit by date range (week by week)

**Example - Instead of:**
```
?fromDate=2025-01-01&toDate=2025-12-31
```

**Do:**
```
?fromDate=2025-01-01&toDate=2025-01-07
?fromDate=2025-01-08&toDate=2025-01-14
```

---

### Issue: Resubmit is slow

**Cause:** Resubmit does two operations (delete + insert)

**Normal Speed:** ~2-3 seconds per resubmit

**Solution:**
- This is expected
- Don't resubmit simultaneously
- Add delays between resubmits

---

## 🔄 Workflow Issues

### Issue: Batch shows "already exists" errors

**Scenario:**
```json
{
  "failedCount": 5,
  "results": [
    {
      "success": false,
      "message": "Invoice already exists in IFCAP"
    }
  ]
}
```

**Solution:**

**Step 1 - Identify edited invoices:**
```
GET /api/audit/invoice/12340
GET /api/audit/invoice/12341
```
Check if each has "Resubmit" action.

**Step 2 - Check if recently edited:**
Look at AutoCount modification date vs. submission date.

**Step 3 - Resubmit only if edited:**
```
POST /api/invoices/resubmit?docKey=12340
```

**Step 4 - Ignore genuine duplicates:**
- If not edited → No action needed

---

### Issue: Lost track of resubmissions

**Solution:**

**Check Resubmit History:**
```
GET /api/audit/resubmits?fromDate=2025-01-01&toDate=2025-01-31
```

**Check AutoCount Note Field:**
```
Note: "IFCAP:5050 | "
```
The number after "IFCAP:" is current JournalID.

**Check IFCAP Creation Date:**
- Resubmitted invoices have recent creation dates
- Compare to audit log dates

---

### Issue: Need monthly audit report

**Solution:**

**Get All Activity:**
```
GET /api/audit/logs?fromDate=2025-01-01&toDate=2025-01-31
```

**Get Resubmits Only:**
```
GET /api/audit/resubmits?fromDate=2025-01-01&toDate=2025-01-31
```

**Query Database:**
```sql
SELECT 
    Action,
    COUNT(*) as Total,
    SUM(CASE WHEN Success = 1 THEN 1 ELSE 0 END) as Success,
    SUM(CASE WHEN Success = 0 THEN 1 ELSE 0 END) as Failed
FROM InvoiceAuditLog
WHERE LogDate >= '2025-01-01' AND LogDate < '2025-02-01'
GROUP BY Action;
```

---

## 📞 When to Contact Support

Contact IT immediately if:

- ❌ API not responding
- ❌ Database connection errors
- ❌ "Failed to delete old invoice" error
- ❌ Resubmit succeeds but data doesn't update
- ❌ Audit logs not recording
- ❌ Recurring failures
- ❌ Multiple resubmit failures
- ❌ Data corruption suspected

---

## 📋 Support Checklist

**When reporting issues, include:**

1. ✅ Endpoint used (`/submit` or `/resubmit`)
2. ✅ DocKey and DocNo
3. ✅ Full error message (JSON)
4. ✅ Old JournalID (if resubmit)
5. ✅ New JournalID (if resubmit)
6. ✅ Was invoice edited? (Yes/No)
7. ✅ Audit log for invoice:
   ```
   GET /api/audit/invoice/{docKey}
   ```
8. ✅ Screenshot
9. ✅ Date and time
10. ✅ AutoCount Note field content

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
✅ Verify in audit log after resubmit  

### 3. During Batch Operations
✅ Review results carefully  
✅ Distinguish duplicates from edited invoices  
✅ Use audit logs to verify  
✅ Don't blindly resubmit all failures  

### 4. Regular Monitoring
✅ Check daily summary every morning  
✅ Track resubmission frequency  
✅ Monitor for recurring patterns  
✅ Review audit logs weekly  

---

## 💡 Pro Tips

💡 **Tip 1:** Always check audit logs before troubleshooting

💡 **Tip 2:** "Already exists" error doesn't always mean problem

💡 **Tip 3:** Use audit logs to understand what happened

💡 **Tip 4:** Check today's summary for quick health check

💡 **Tip 5:** Document recurring issues and report to IT

💡 **Tip 6:** Test resubmit with 1-2 invoices before bulk

---

## 📚 Additional Resources

- **API Reference:** Complete endpoint documentation
- **Quick Start Guide:** Step-by-step tutorials
- **FAQ:** Common questions and answers
- **File Structure Guide:** For developers

---

**Document Version:** 1.2  
**Last Updated:** January 28, 2025  
**Changes:** Added audit logging troubleshooting  
**Maintained by:** IT Department
