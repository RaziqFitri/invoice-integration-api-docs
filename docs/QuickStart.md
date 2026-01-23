# Invoice Integration API - Quick Start Guide

**⏱️ Reading Time:** 5 minutes  
**👤 For:** End Users (Finance, Accounting)

---

## 🎯 What You'll Learn

By the end of this guide, you'll know how to:
1. ✅ Submit a single invoice
2. ✅ Submit yesterday's invoices
3. ✅ Search for pending invoices
4. ✅ Check invoice status

---

## 📍 Step 1: Access the API

### Open Swagger UI

1. Open your web browser
2. Go to: `http://your-server-name:5000/swagger`
3. You should see the API documentation page

![Swagger UI](placeholder)

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
  "ifcapJournalID": 5001,
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
    // ... more invoices
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
      "message": "Creditor mapping not found for: 4000/Z999",
      "journalID": null
    }
    // ... more results
  ]
}
```

**What to do with failures:**
1. Note the DocKey and error message
2. Common error: "Creditor mapping not found"
3. Contact IT to add the missing supplier
4. Resubmit the failed invoice later

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

### Task 3: Check This Month's Submissions

```
GET /api/invoices/search
  ?fromDate=2025-01-01
  &toDate=2025-01-31
  &status=Y
```

Shows all invoices submitted in January that are pending approval.

---

### Task 4: Approve an Invoice

```
POST /api/invoices/approve?docKey=12345
```

Changes status from Y → A.

---

### Task 5: Reject an Invoice

```
POST /api/invoices/reject?docKey=12345&reason=Incorrect amount
```

Changes status from Y → R and stores the rejection reason.

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

**Note:** POST requests (submit, approve, reject) cannot be done via browser address bar. Use Swagger or tools like Postman.

---

## 📱 Using Postman (Advanced)

### Setup

1. Download Postman (free)
2. Create new request
3. Set base URL: `http://your-server:5000`

### Example: Submit Invoice

- **Method:** POST
- **URL:** `{{baseURL}}/api/invoices/submit`
- **Params:**
  - Key: `docKey`
  - Value: `12345`
- Click **Send**

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

### 4. Submitting Already Submitted Invoice
**Error:** "Invoice already submitted. IFCAP JournalID: 5001"  
**Fix:** Check if this is intentional. Verify in IFCAP using JournalID.

---

## ✅ Daily Checklist

### Morning Routine (5 minutes)

1. **Check pending count**
   ```
   GET /search?status=N&pageSize=1
   ```
   (Check `totalRecords` in response)

2. **Submit yesterday's invoices**
   ```
   POST /batch/submit-by-date?fromDate=2025-01-21&toDate=2025-01-21
   ```

3. **Review results**
   - Note any failures
   - Report to IT if needed

4. **Done!** ✅

---

## 📞 Getting Help

### Error Messages You Can Fix:

✅ **"ToDate must be greater than FromDate"**
- Fix: Check your dates

✅ **"Invoice not found"**
- Fix: Verify DocKey in AutoCount

### Errors That Need IT Support:

❌ **"Creditor mapping not found"**
- Action: Email IT with supplier code

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
4. ✅ Read full **API Documentation** for all features

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

💡 **Tip 3:** Keep a log of creditor mapping errors to report in batches

💡 **Tip 4:** Use browser bookmarks for common searches

💡 **Tip 5:** Test with 1-2 invoices before large batch submissions

---

**🎉 Congratulations!** You now know the basics of the Invoice Integration API!

**Questions?** Contact IT Support or refer to the full documentation.

---

**Document Version:** 1.0  
**Last Updated:** January 23, 2025  
**For:** End Users
