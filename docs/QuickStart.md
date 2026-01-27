# Invoice Integration API - Quick Start Guide

**⏱️ Reading Time:** 6 minutes  
**👤 For:** End Users (Finance, Accounting)  
**Version:** 1.1

---

## 🎯 What You'll Learn

By the end of this guide, you'll know how to:
1. ✅ Submit a single invoice
2. ✅ Resubmit an edited invoice (NEW!)
3. ✅ Submit yesterday's invoices
4. ✅ Search for pending invoices
5. ✅ Check invoice status

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

---

## 🔄 Step 2B: 🆕 Resubmit an Edited Invoice

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

**⚠️ Important:** The old invoice is permanently deleted. The new invoice will need approval again.

---

## 🔍 Step 3: Check Invoice Status

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

## 📊 Step 4: Search Pending Invoices

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

## 🚀 Step 5: Submit Yesterday's Invoices (Batch)

### Using Date Range

1. Find **POST /api/invoices/batch/submit-by-date**
2. Click **"Try it out"**
3. Enter:
   - **fromDate**: `2025-01-21` (yesterday)
   - **toDate**: `2025-01-21` (yesterday)
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

## 💡 Common Tasks

### Task 1: Submit All Pending Invoices

**⚠️ Warning:** This submits ALL invoices with status "N"

```
POST /api/invoices/batch/submit-by-status?status=N
```

**Use when:**
- Month-end closing
- Catching up on backlog
- All invoices are ready for IFCAP

---

### Task 2: Submit Specific Supplier's Invoices

**Example:** Submit all January invoices from T & T CAHAYA MURNI

```
POST /api/invoices/batch/submit-by-date
  ?fromDate=2025-01-01
  &toDate=2025-01-31
  &creditorCode=4000/T001
```

**Use when:**
- Supplier audit requests
- Processing by supplier priority
- Testing new supplier integration

---

### Task 3: 🆕 Fix Invoices That Were Edited After Submission

**Scenario:** You submitted 10 invoices yesterday, but today you realized 3 of them had wrong amounts. You fixed them in AutoCount.

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

**Result:** IFCAP now has the corrected data!

---

### Task 4: Check This Month's Submissions

```
GET /api/invoices/search
  ?fromDate=2025-01-01
  &toDate=2025-01-31
  &status=Y
```

Shows all invoices submitted in January that are pending approval.

---

### Task 5: Approve an Invoice

```
POST /api/invoices/approve?docKey=12345
```

Changes status from Y → A.

---

### Task 6: Reject an Invoice

```
POST /api/invoices/reject?docKey=12345&reason=Incorrect amount
```

Changes status from Y → R and stores the rejection reason.

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
- `2025-01-23`
- `2025-12-31`

❌ **Wrong:**
- `23/01/2025`
- `01-23-2025`
- `23-Jan-2025`

---

## 🔧 Using Browser (Alternative to Swagger)

You can also use the browser address bar for GET requests:

### Search Pending Invoices
```
http://your-server:5000/api/invoices/search?status=N&pageSize=10
```

### Check Status
```
http://your-server:5000/api/invoices/status/12345
```

**Note:** POST requests (submit, resubmit, approve, reject) cannot be done via browser address bar. Use Swagger or tools like Postman.

---

## ⚠️ Common Mistakes

### 1. Wrong Date Format
❌ `fromDate=23/01/2025`  
✅ `fromDate=2025-01-23`

### 2. Trying to Submit Cancelled Invoice
**Error:** "Cannot submit cancelled invoice"  
**Fix:** Check invoice in AutoCount - ensure it's not cancelled

### 3. Missing Creditor Mapping
**Error:** "Creditor mapping not found for: 4000/XXXX"  
**Fix:** Contact IT to add the supplier mapping

### 4. 🆕 Using Submit Instead of Resubmit
**Error:** "Invoice already exists in IFCAP"  
**Fix:** If invoice was edited, use `/resubmit` instead of `/submit`

### 5. 🆕 Forgetting Invoice Was Edited
**Problem:** Old data still in IFCAP after you edited in AutoCount  
**Fix:** Use `/resubmit` to replace with updated data

---

## ✅ Daily Checklist

### Morning Routine (5-7 minutes)

1. **Check pending count**
   ```
   GET /search?status=N&pageSize=1
   ```
   (Check `totalRecords` in response)

2. **Submit yesterday's invoices**
   ```
   POST /batch/submit-by-date?fromDate=2025-01-26&toDate=2025-01-26
   ```

3. **Review results**
   - Note any failures
   - Note "already exists" errors

4. **Handle edited invoices** (NEW!)
   - For "already exists" errors, check if invoice was edited
   - Use `/resubmit` for edited invoices

5. **Report issues to IT**
   - Creditor mapping errors
   - Technical failures

6. **Done!** ✅

---

## 📞 Getting Help

### Error Messages You Can Fix:

✅ **"ToDate must be greater than FromDate"**
- Fix: Check your dates

✅ **"Invoice not found"**
- Fix: Verify DocKey in AutoCount

✅ **"Invoice already exists" + invoice was edited**
- Fix: Use `/resubmit` instead of `/submit`

### Errors That Need IT Support:

❌ **"Creditor mapping not found"**
- Action: Email IT with supplier code

❌ **"Failed to delete old invoice from IFCAP"**
- Action: Contact IT immediately with DocKey and JournalID

❌ **"Cannot connect to API"**
- Action: Contact IT support

❌ **"Database connection error"**
- Action: Contact IT support immediately

---

## 🎯 Next Steps

After mastering the basics:

1. ✅ Set up **automated daily submissions** (Task Scheduler)
2. ✅ Learn to use **Postman** for advanced testing
3. ✅ Understand **status workflow** (N → Y → A/R)
4. ✅ Master the **resubmit workflow** for edited invoices
5. ✅ Read full **API Documentation** for all features

---

## 📚 Additional Resources

- **Full API Documentation:** [Link to main doc]
- **Task Scheduler Setup Guide:** [Link to scheduler doc]
- **Troubleshooting Guide:** [Link to troubleshooting]
- **Video Tutorials:** [If available]

---

## ✨ Pro Tips

💡 **Tip 1:** Use search before batch submit to know how many invoices you'll process

💡 **Tip 2:** Always review batch results, even if all succeeded

💡 **Tip 3:** Keep a log of invoices that need resubmission

💡 **Tip 4:** Check "already exists" errors - they might indicate edited invoices

💡 **Tip 5:** Test with 1-2 invoices before large batch submissions

💡 **Tip 6 (NEW):** If you edit an invoice in AutoCount after submission, immediately resubmit it to keep IFCAP in sync

💡 **Tip 7 (NEW):** Use browser bookmarks for common searches and frequent resubmits

---

## 🆕 What's New in Version 1.1

✨ **Resubmit Feature** - Replace edited invoices in IFCAP  
✨ **Better Error Messages** - Clear guidance on when to use resubmit  
✨ **Improved Workflow** - Handle edited invoices seamlessly

---

**🎉 Congratulations!** You now know the basics of the Invoice Integration API including the new resubmit feature!

**Questions?** Contact IT Support or refer to the full documentation.

---

**Document Version:** 1.1  
**Last Updated:** January 27, 2025  
**Changes:** Added resubmit feature documentation  
**For:** End Users
