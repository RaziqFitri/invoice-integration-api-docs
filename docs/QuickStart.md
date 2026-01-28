# Invoice Integration API - Quick Start Guide

**⏱️ Reading Time:** 8 minutes  
**👤 For:** End Users (Finance, Accounting)  
**Version:** 1.2

---

## 🎯 What You'll Learn

By the end of this guide, you'll know how to:
1. ✅ Submit a single invoice
2. ✅ Resubmit an edited invoice
3. ✅ Submit yesterday's invoices
4. ✅ Search for pending invoices
5. ✅ Check invoice status
6. ✅ 🆕 View audit logs and track activity

---

## 📍 Step 1: Access the API

### Open Swagger UI

1. Open your web browser
2. Go to: `http://your-server-name:5000/swagger`
3. You should see the API documentation page

---

## 📤 Step 2: Submit Your First Invoice

### Find an Invoice DocKey

1. Open **AutoCount**
2. Go to **Accounts Payable → Invoice**
3. Open any invoice
4. Note the **DocKey** (usually shown in the URL or document properties)
   - Example: `12345`

### Submit via Swagger

1. In Swagger UI, find **POST /api/invoices/submit**
2. Click **"Try it out"**
3. Enter your **DocKey**: `12345`
4. Click **"Execute"**

### ✅ Success Response

You should see:
```json
{
  "message": "Invoice submitted to IFCAP successfully",
  "journalID": 5001
}
```

**What happened:**
- ✅ Invoice copied from AutoCount to IFCAP
- ✅ IFCAP JournalID: 5001 created
- ✅ AutoCount status changed to "Y" (Submitted)
- ✅ **Action logged in audit trail**

---

## 🔄 Step 3: Resubmit an Edited Invoice

### When to Use Resubmit

**Use this when:**
- You already submitted an invoice to IFCAP
- Then made changes in AutoCount (amount, line items, etc.)
- IFCAP now has outdated data

### Example Scenario

You submitted invoice DocKey 12345, but then realized the amount was wrong and corrected it in AutoCount.

### Resubmit via Swagger

1. In Swagger UI, find **POST /api/invoices/resubmit**
2. Click **"Try it out"**
3. Enter **DocKey**: `12345`
4. Click **"Execute"**

### ✅ Success Response

```json
{
  "message": "Invoice resubmitted successfully. Old JournalID 5001 deleted, new JournalID: 5050",
  "journalID": 5050
}
```

**What happened:**
- 🗑️ Old invoice (JournalID 5001) deleted from IFCAP
- ✅ New invoice (JournalID 5050) created with updated data
- ✅ AutoCount status updated with new JournalID
- ✅ **Two audit logs created:** Delete + Resubmit

⚠️ **Important:** The old invoice is permanently deleted. The new invoice will need approval again.

---

## 🔍 Step 4: Check Invoice Status

### Using Swagger

1. Find **GET /api/invoices/status/{docKey}**
2. Click **"Try it out"**
3. Enter **DocKey**: `12345`
4. Click **"Execute"**

### Response

```json
{
  "docKey": 12345,
  "docNo": "INV-001",
  "docStatus": "Y",
  "ifcapJournalID": 5050,
  "statusDescription": "Submitted/Pending in IFCAP"
}
```

**Status Meanings:**
- **N** = Not submitted yet
- **Y** = Submitted, pending in IFCAP
- **A** = Approved
- **R** = Rejected

---

## 📊 Step 5: Search Pending Invoices

### Find All Unsubmitted Invoices

1. Find **GET /api/invoices/search**
2. Click **"Try it out"**
3. Enter:
   - **status**: `N`
   - **pageSize**: `10`
4. Click **"Execute"**

### Response

```json
{
  "data": [
    {
      "docKey": 12340,
      "docNo": "INV-020",
      "creditorCode": "4000/T001",
      "docDate": "2025-01-20",
      "total": 2500.00,
      "udf_APIstatus": "N",
      "statusDescription": "Not Submitted"
    }
  ],
  "totalRecords": 45,
  "totalPages": 5
}
```

**This tells you:**
- 45 invoices waiting to be submitted
- 10 shown per page (5 pages total)

---

## 🚀 Step 6: Submit Yesterday's Invoices (Batch)

### Using Date Range

1. Find **POST /api/invoices/batch/submit-by-date**
2. Click **"Try it out"**
3. Enter:
   - **fromDate**: `2025-01-27` (yesterday)
   - **toDate**: `2025-01-27` (yesterday)
4. Click **"Execute"**

### Response

```json
{
  "totalInvoices": 8,
  "successCount": 7,
  "failedCount": 1,
  "summary": "Total: 8, Success: 7, Failed: 1",
  "results": [
    {
      "docKey": 12340,
      "docNo": "INV-020",
      "success": true,
      "message": "Invoice submitted to IFCAP successfully",
      "journalID": 5010
    },
    {
      "docKey": 12341,
      "docNo": "INV-021",
      "success": false,
      "message": "Invoice already exists in IFCAP (JournalID: 5002). Use /api/invoices/resubmit endpoint to replace it.",
      "journalID": null
    }
  ]
}
```

### 🆕 Handling "Already Exists" Errors

**If you see:** `"Invoice already exists in IFCAP"`

**Step 1 - Check if invoice was edited:**
1. Open the invoice in AutoCount (use the DocKey from error)
2. Check if you made changes after the original submission

**Step 2 - If edited, resubmit:**
```
POST /api/invoices/resubmit?docKey=12341
```

**Step 3 - If NOT edited:**
- No action needed
- Invoice is already in IFCAP correctly

---

## 🆕 Step 7: View Audit Logs

### Check Today's Activity Summary

1. Find **GET /api/audit/summary/today**
2. Click **"Execute"**

### Response

```json
{
  "date": "2025-01-28",
  "totalActions": 20,
  "successCount": 18,
  "failureCount": 2,
  "byAction": [
    {
      "action": "Submit",
      "count": 15,
      "success": 14,
      "failed": 1
    },
    {
      "action": "Resubmit",
      "count": 5,
      "success": 4,
      "failed": 1
    }
  ],
  "recentFailures": [
    {
      "action": "Submit",
      "docKey": 12350,
      "errorMessage": "Creditor mapping not found for: 4000/Z999"
    }
  ]
}
```

**What This Tells You:**
- Total operations today: 20
- Success rate: 90% (18/20)
- 2 failures to investigate
- Breakdown by action type

---

### Check Specific Invoice History

1. Find **GET /api/audit/invoice/{docKey}**
2. Enter **DocKey**: `12345`
3. Click **"Execute"**

### Response

```json
{
  "docKey": 12345,
  "totalActions": 3,
  "history": [
    {
      "logDate": "2025-01-28T10:00:00",
      "action": "Submit",
      "newJournalID": 5001,
      "success": true,
      "userName": "Anonymous",
      "ipAddress": "192.168.1.100"
    },
    {
      "logDate": "2025-01-28T14:30:00",
      "action": "Resubmit",
      "oldJournalID": 5001,
      "newJournalID": 5050,
      "success": true
    },
    {
      "logDate": "2025-01-28T16:00:00",
      "action": "Approve",
      "success": true
    }
  ]
}
```

**What This Tells You:**
- Complete history of the invoice
- When it was submitted (10:00 AM)
- When it was edited and resubmitted (2:30 PM)
- When it was approved (4:00 PM)
- Who did each action

**Use Cases:**
- Verify if invoice was edited
- Track approval timeline
- Audit trail for compliance
- Troubleshoot issues

---

## 💡 Common Tasks

### Task 1: Submit All Pending Invoices

⚠️ **Warning:** This submits ALL invoices with status "N"

```
POST /api/invoices/batch/submit-by-status?status=N
```

**Use when:**
- Month-end closing
- Catching up on backlog

---

### Task 2: Submit Specific Supplier's Invoices

**Example:** Submit all January invoices from T & T CAHAYA MURNI

```
POST /api/invoices/batch/submit-by-date
  ?fromDate=2025-01-01
  &toDate=2025-01-31
  &creditorCode=4000/T001
```

---

### Task 3: Fix Invoices That Were Edited After Submission

**Scenario:** You submitted 10 invoices yesterday, but today you realized 3 had wrong amounts. You fixed them in AutoCount.

**Step 1 - List the edited invoices:**
```
Edited: INV-020 (DocKey 12340)
Edited: INV-022 (DocKey 12342)
Edited: INV-025 (DocKey 12345)
```

**Step 2 - Resubmit each one:**
```
POST /api/invoices/resubmit?docKey=12340
POST /api/invoices/resubmit?docKey=12342
POST /api/invoices/resubmit?docKey=12345
```

**Result:** IFCAP now has the corrected data! ✓

---

### Task 4: Check Which Invoices Were Edited This Week

```
GET /api/audit/resubmits?fromDate=2025-01-20&toDate=2025-01-28
```

**Response shows:**
- All invoices that were resubmitted
- When they were resubmitted
- Old and new JournalIDs

**Use Cases:**
- Data quality monitoring
- Identify patterns (why are invoices being edited?)
- Compliance reporting

---

### Task 5: Find Today's Errors

```
GET /api/audit/summary/today
```

Look at the `recentFailures` section:
```json
{
  "recentFailures": [
    {
      "docKey": 12350,
      "errorMessage": "Creditor mapping not found for: 4000/Z999"
    }
  ]
}
```

**Action:** Email IT to add supplier mapping for 4000/Z999

---

## 🔄 Submit vs. Resubmit - Quick Reference

| Action | Use `/submit` | Use `/resubmit` |
|--------|---------------|-----------------|
| First time submitting | ✅ | ❌ |
| Invoice edited after submission | ❌ | ✅ |
| Invoice already in IFCAP | ❌ | ✅ |
| Need to replace old data | ❌ | ✅ |
| Fresh, new invoice | ✅ | ❌ |

**Rule of Thumb:**
- **First time?** Use `/submit`
- **Fixing/updating?** Use `/resubmit`

---

## 🎓 Date Format Guide

Always use **YYYY-MM-DD** format:

✅ **Correct:**
- `2025-01-28`
- `2025-12-31`

❌ **Wrong:**
- `28/01/2025`
- `01-28-2025`
- `28-Jan-2025`

---

## ⚠️ Common Mistakes

### 1. Wrong Date Format
❌ `fromDate=28/01/2025`  
✅ `fromDate=2025-01-28`

### 2. Using Submit Instead of Resubmit
**Error:** "Invoice already exists in IFCAP"  
**Fix:** If invoice was edited, use `/resubmit`

### 3. Not Checking Audit Logs
**Problem:** Don't know if invoice was edited  
**Fix:** Check `GET /api/audit/invoice/{docKey}`

### 4. Ignoring Today's Summary
**Problem:** Missing errors  
**Fix:** Check `GET /api/audit/summary/today` every morning

---

## ✅ Daily Checklist

### Morning Routine (5-7 minutes)

1. **Check today's summary**
   ```
   GET /api/audit/summary/today
   ```
   - Review success count
   - Check for failures

2. **Submit yesterday's invoices**
   ```
   POST /api/invoices/batch/submit-by-date?fromDate=2025-01-27&toDate=2025-01-27
   ```

3. **Review results**
   - Note any failures
   - Note "already exists" errors

4. **Handle edited invoices**
   - For "already exists" errors, check if edited
   - Use `/resubmit` for edited invoices

5. **Check pending count**
   ```
   GET /api/invoices/search?status=N&pageSize=1
   ```
   (Check `totalRecords`)

6. **Done!** ✅

---

## 📞 Getting Help

### Error Messages You Can Fix:

✅ **"Invoice already exists" + invoice was edited**
- Fix: Use `/resubmit`

✅ **"ToDate must be greater than FromDate"**
- Fix: Check your dates

### Errors That Need IT Support:

❌ **"Creditor mapping not found"**
- Action: Email IT with supplier code

❌ **"Failed to delete old invoice from IFCAP"**
- Action: Contact IT immediately

❌ **"Cannot connect to API"**
- Action: Contact IT support

---

## 🎯 Next Steps

After mastering the basics:

1. ✅ Set up **automated daily submissions** (Task Scheduler)
2. ✅ Learn **audit log queries** for reporting
3. ✅ Master the **resubmit workflow**
4. ✅ Use **audit logs** for compliance
5. ✅ Read full **API Documentation**

---

## ✨ Pro Tips

💡 **Tip 1:** Always check audit summary before batch submit

💡 **Tip 2:** Use audit logs to track who did what

💡 **Tip 3:** Monitor resubmit frequency (indicates data quality issues)

💡 **Tip 4:** If you edit an invoice, resubmit immediately

💡 **Tip 5:** Check "already exists" errors - they might indicate edited invoices

💡 **Tip 6:** Use browser bookmarks for common endpoints

💡 **Tip 7:** Generate monthly audit reports for compliance

---

## 🆕 What's New in Version 1.2

✨ **Complete Audit Trail** - Every action is now logged  
✨ **5 New Audit Endpoints** - View logs, history, reports  
✨ **Daily Summary** - Quick health check for activity  
✨ **Resubmit Tracking** - Know which invoices were edited  
✨ **Error Monitoring** - Track and fix failures quickly  

---

**🎉 Congratulations!** You now know how to use the Invoice Integration API including audit logging!

**Questions?** Contact IT Support or refer to the full documentation.

---

**Document Version:** 1.2  
**Last Updated:** January 28, 2025  
**Changes:** Added audit logging section  
**For:** End Users
