# Invoice Integration API - User Documentation

**Version:** 1.0  
**Last Updated:** January 2025  
**Support Contact:** IT Department

---

## Table of Contents

1. [Overview](#overview)
2. [Getting Started](#getting-started)
3. [API Endpoints](#api-endpoints)
4. [Usage Examples](#usage-examples)
5. [Status Codes](#status-codes)
6. [Error Handling](#error-handling)
7. [Troubleshooting](#troubleshooting)
8. [FAQ](#faq)

---

## Overview

### What is the Invoice Integration API?

The Invoice Integration API bridges AutoCount and IFCAP365, automating invoice transfers between systems.

### Key Features

✅ **Single Invoice Submission** - Submit one invoice at a time  
✅ **Batch Transfer by Date** - Submit all invoices in a date range  
✅ **Batch Transfer by Status** - Submit all pending invoices  
✅ **Search & Filter** - Find invoices by date and status  
✅ **Status Tracking** - Monitor invoice submission status  
✅ **Approve/Reject** - Manage invoice approvals

### System Requirements

- AutoCount database access
- IFCAP365 database access
- Network connectivity to API server
- Web browser or API testing tool (Postman, etc.)

---

## Getting Started

### Base URL

```
http://your-server-name:5000/api/invoices
```

Replace `your-server-name` with your actual server address.

### Accessing Swagger UI

For interactive documentation and testing:

```
http://your-server-name:5000/swagger
```

---

## API Endpoints

### 1. Submit Single Invoice

**Endpoint:** `POST /api/invoices/submit`

**Description:** Submits a single invoice from AutoCount to IFCAP.

**Parameters:**
| Name | Type | Required | Description |
|------|------|----------|-------------|
| docKey | integer | Yes | AutoCount invoice DocKey |

**Example Request:**
```
POST /api/invoices/submit?docKey=12345
```

**Success Response (200 OK):**
```json
{
  "message": "Invoice submitted to IFCAP successfully",
  "journalID": 5001
}
```

**Error Response (400 Bad Request):**
```json
{
  "message": "Creditor mapping not found for: 4000/Z999"
}
```

---

### 2. Batch Submit by Date Range

**Endpoint:** `POST /api/invoices/batch/submit-by-date`

**Description:** Submits all pending invoices within a date range.

**Parameters:**
| Name | Type | Required | Description |
|------|------|----------|-------------|
| fromDate | date | Yes | Start date (YYYY-MM-DD) |
| toDate | date | Yes | End date (YYYY-MM-DD) |
| creditorCode | string | No | Filter by specific creditor (e.g., 4000/T001) |

**Example Request:**
```
POST /api/invoices/batch/submit-by-date?fromDate=2025-01-01&toDate=2025-01-31
```

**With Creditor Filter:**
```
POST /api/invoices/batch/submit-by-date?fromDate=2025-01-01&toDate=2025-01-31&creditorCode=4000/T001
```

**Success Response (200 OK):**
```json
{
  "totalInvoices": 15,
  "successCount": 13,
  "failedCount": 2,
  "summary": "Total: 15, Success: 13, Failed: 2",
  "results": [
    {
      "docKey": 12345,
      "docNo": "INV-001",
      "success": true,
      "message": "Invoice submitted to IFCAP successfully",
      "journalID": 5001
    },
    {
      "docKey": 12346,
      "docNo": "INV-002",
      "success": false,
      "message": "Creditor mapping not found for: 4000/Z999",
      "journalID": null
    }
  ]
}
```

---

### 3. Batch Submit by Status

**Endpoint:** `POST /api/invoices/batch/submit-by-status`

**Description:** Submits all invoices with a specific status.

**Parameters:**
| Name | Type | Required | Description |
|------|------|----------|-------------|
| status | string | No | Status code (default: N) |

**Valid Status Values:**
- `N` or `NULL` - Not submitted (default)
- `Y` - Pending in IFCAP
- `A` - Approved
- `R` - Rejected

**Example Request:**
```
POST /api/invoices/batch/submit-by-status?status=N
```

**Response:** Same format as batch submit by date.

⚠️ **Warning:** This can process hundreds or thousands of invoices!

---

### 4. Search Invoices

**Endpoint:** `GET /api/invoices/search`

**Description:** Search and filter invoices with pagination.

**Parameters:**
| Name | Type | Required | Description |
|------|------|----------|-------------|
| fromDate | date | No | Start date (YYYY-MM-DD) |
| toDate | date | No | End date (YYYY-MM-DD) |
| status | string | No | Filter by status (N, Y, A, R) |
| pageSize | integer | No | Records per page (default: 50, max: 1000) |
| pageNumber | integer | No | Page number (default: 1) |

**Example Request:**
```
GET /api/invoices/search?fromDate=2025-01-01&toDate=2025-01-31&status=Y&pageSize=20&pageNumber=1
```

**Success Response (200 OK):**
```json
{
  "data": [
    {
      "docKey": 12345,
      "docNo": "INV-001",
      "creditorCode": "4000/T001",
      "docDate": "2025-01-15T00:00:00",
      "total": 1500.00,
      "udf_APIstatus": "Y",
      "note": "IFCAP:5001 | ",
      "ifcapJournalID": 5001,
      "statusDescription": "Submitted/Pending in IFCAP"
    }
  ],
  "pageNumber": 1,
  "pageSize": 20,
  "totalRecords": 45,
  "totalPages": 3,
  "hasNextPage": true,
  "hasPreviousPage": false
}
```

---

### 5. Get Invoice Status

**Endpoint:** `GET /api/invoices/status/{docKey}`

**Description:** Get the current status of a specific invoice.

**Parameters:**
| Name | Type | Required | Description |
|------|------|----------|-------------|
| docKey | integer | Yes | AutoCount invoice DocKey |

**Example Request:**
```
GET /api/invoices/status/12345
```

**Success Response (200 OK):**
```json
{
  "docKey": 12345,
  "docNo": "INV-001",
  "docStatus": "Y",
  "remark": "IFCAP:5001 | ",
  "ifcapJournalID": 5001,
  "statusDescription": "Submitted/Pending in IFCAP"
}
```

**Not Found Response (404):**
```json
{
  "message": "Invoice not found"
}
```

---

### 6. Approve Invoice

**Endpoint:** `POST /api/invoices/approve`

**Description:** Approve an invoice (changes status from Y to A).

**Parameters:**
| Name | Type | Required | Description |
|------|------|----------|-------------|
| docKey | integer | Yes | AutoCount invoice DocKey |

**Example Request:**
```
POST /api/invoices/approve?docKey=12345
```

**Success Response (200 OK):**
```json
{
  "message": "Invoice approved successfully"
}
```

**Error Response (400 Bad Request):**
```json
{
  "message": "Invoice not found or not in pending status (Y)"
}
```

---

### 7. Reject Invoice

**Endpoint:** `POST /api/invoices/reject`

**Description:** Reject an invoice with a reason (changes status from Y to R).

**Parameters:**
| Name | Type | Required | Description |
|------|------|----------|-------------|
| docKey | integer | Yes | AutoCount invoice DocKey |
| reason | string | Yes | Rejection reason |

**Example Request:**
```
POST /api/invoices/reject?docKey=12345&reason=Incorrect amount
```

**Success Response (200 OK):**
```json
{
  "message": "Invoice rejected successfully"
}
```

---

## Usage Examples

### Example 1: Submit Yesterday's Invoices

**Scenario:** Submit all invoices from yesterday at the end of the day.

**Request:**
```
POST /api/invoices/batch/submit-by-date?fromDate=2025-01-21&toDate=2025-01-21
```

**Use Case:** Daily routine submission.

---

### Example 2: Month-End Closing

**Scenario:** Submit all invoices from January 2025.

**Step 1 - Search to verify count:**
```
GET /api/invoices/search?fromDate=2025-01-01&toDate=2025-01-31&status=N
```

**Step 2 - Submit in batches:**
```
POST /api/invoices/batch/submit-by-date?fromDate=2025-01-01&toDate=2025-01-31
```

**Use Case:** Monthly closing process.

---

### Example 3: Submit Specific Supplier Invoices

**Scenario:** Submit all invoices from T & T CAHAYA MURNI SDN BHD for January.

**Request:**
```
POST /api/invoices/batch/submit-by-date?fromDate=2025-01-01&toDate=2025-01-31&creditorCode=4000/T001
```

**Use Case:** Supplier-specific processing or audit requests.

---

### Example 4: Check Pending Invoices

**Scenario:** See how many invoices are waiting to be submitted.

**Request:**
```
GET /api/invoices/search?status=N&pageSize=10&pageNumber=1
```

**Response shows:**
- Total pending invoices
- First 10 invoices
- How many pages total

**Use Case:** Daily monitoring and planning.

---

### Example 5: Review Failed Submissions

**Scenario:** After batch submission, some invoices failed. Review and fix them.

**Step 1 - Review batch results:**
Look at the `results` array in the batch response for `success: false` entries.

**Step 2 - Check specific invoice:**
```
GET /api/invoices/status/12346
```

**Step 3 - Fix issue (e.g., add missing creditor mapping)**

**Step 4 - Resubmit:**
```
POST /api/invoices/submit?docKey=12346
```

---

## Status Codes

### Invoice Status in AutoCount

| Code | Description | Meaning |
|------|-------------|---------|
| `N` or `NULL` | Not Submitted | Invoice created in AutoCount but not sent to IFCAP |
| `Y` | Submitted/Pending | Successfully sent to IFCAP, waiting for approval |
| `A` | Approved | Invoice has been approved |
| `R` | Rejected | Invoice was rejected with a reason |

### HTTP Status Codes

| Code | Meaning |
|------|---------|
| 200 | Success - Request completed successfully |
| 400 | Bad Request - Invalid parameters or business rule violation |
| 404 | Not Found - Invoice does not exist |
| 500 | Server Error - Contact IT support |

---

## Error Handling

### Common Error Messages

#### 1. "Creditor mapping not found for: XXXX"

**Cause:** The supplier code in AutoCount is not mapped to an IFCAP creditor.

**Solution:** Contact IT to add the creditor mapping to the configuration.

**Example:**
```json
{
  "message": "Creditor mapping not found for: 4000/Z999"
}
```

---

#### 2. "Invoice already submitted. IFCAP JournalID: XXXX"

**Cause:** This invoice was already sent to IFCAP previously.

**Solution:** 
- Check IFCAP using the JournalID provided
- If this is a duplicate, no action needed
- If re-submission is needed, contact IT support

**Example:**
```json
{
  "message": "Invoice already submitted. IFCAP JournalID: 5001"
}
```

---

#### 3. "Cannot submit cancelled invoice"

**Cause:** The invoice is marked as cancelled in AutoCount.

**Solution:** Cancelled invoices cannot be submitted. Verify the invoice status in AutoCount.

---

#### 4. "No line items found for invoice"

**Cause:** The invoice has no detail lines in AutoCount.

**Solution:** Add line items to the invoice in AutoCount before submitting.

---

#### 5. "Invoice not found or not in pending status (Y)"

**Cause:** Trying to approve/reject an invoice that:
- Doesn't exist
- Hasn't been submitted yet (status N)
- Was already approved (status A)
- Was already rejected (status R)

**Solution:** Check the invoice status first using the status endpoint.

---

#### 6. "ToDate must be greater than or equal to FromDate"

**Cause:** Date range is invalid.

**Solution:** Ensure `toDate` is the same as or after `fromDate`.

**Example of invalid request:**
```
?fromDate=2025-01-31&toDate=2025-01-01  ❌
```

**Correct:**
```
?fromDate=2025-01-01&toDate=2025-01-31  ✅
```

---

## Troubleshooting

### Issue: No invoices returned when searching

**Possible Causes:**
1. No invoices exist for the date range
2. Wrong status filter
3. All invoices already submitted

**Solution:**
```
# Try searching without filters first
GET /api/invoices/search?pageSize=10

# Then add filters one at a time
GET /api/invoices/search?status=N&pageSize=10
GET /api/invoices/search?fromDate=2025-01-01&toDate=2025-01-31&pageSize=10
```

---

### Issue: Batch submission takes too long

**Cause:** Large number of invoices being processed.

**Solution:**
- Break into smaller date ranges
- Submit in batches of 50-100 invoices
- Schedule during off-peak hours

**Example - Instead of:**
```
?fromDate=2025-01-01&toDate=2025-12-31  (Entire year)
```

**Do:**
```
?fromDate=2025-01-01&toDate=2025-01-31  (January)
?fromDate=2025-02-01&toDate=2025-02-28  (February)
```

---

### Issue: Some invoices fail in batch

**Cause:** Mixed - different reasons for different invoices.

**Solution:**
1. Review the `results` array in the response
2. Group failures by error message
3. Fix common issues (e.g., missing creditor mappings)
4. Resubmit failed invoices individually or in small batches

---

### Issue: Can't find IFCAP JournalID

**Cause:** Invoice submitted but JournalID not showing.

**Solution:**
Check the invoice's `Note` field in AutoCount - it contains:
```
IFCAP:5001 | 
```
The number after "IFCAP:" is the JournalID.

---

## FAQ

### Q1: How do I know which invoices need to be submitted?

**A:** Use the search endpoint with status `N`:
```
GET /api/invoices/search?status=N
```

This shows all invoices that haven't been submitted yet.

---

### Q2: Can I submit invoices from multiple months at once?

**A:** Yes, but it's not recommended for performance reasons. Better to submit in monthly batches:
```
# January
POST /api/invoices/batch/submit-by-date?fromDate=2025-01-01&toDate=2025-01-31

# February
POST /api/invoices/batch/submit-by-date?fromDate=2025-02-01&toDate=2025-02-28
```

---

### Q3: What happens if I submit the same invoice twice?

**A:** The API prevents duplicates. If an invoice was already submitted, you'll get an error:
```json
{
  "message": "Invoice already submitted. IFCAP JournalID: 5001"
}
```

---

### Q4: How long does batch submission take?

**A:** Approximately:
- 1-2 seconds per invoice
- 10 invoices = ~15 seconds
- 100 invoices = ~2-3 minutes

The API includes small delays between invoices to prevent database overload.

---

### Q5: Can I undo a submission?

**A:** No, submissions cannot be undone through the API. Once an invoice is in IFCAP, it must be managed there. However, you can reject it using:
```
POST /api/invoices/reject?docKey=12345&reason=Submitted by mistake
```

---

### Q6: What's the difference between status Y and A?

**A:**
- **Y (Submitted/Pending):** Invoice sent to IFCAP, waiting for approval
- **A (Approved):** Invoice has been reviewed and approved

Workflow: `N → Y → A` (or `R` if rejected)

---

### Q7: How do I find invoices submitted in the last week?

**A:**
```
GET /api/invoices/search?fromDate=2025-01-15&toDate=2025-01-22&status=Y
```

Change dates to your desired week.

---

### Q8: Can I schedule automatic submissions?

**A:** Yes! See the Task Scheduler documentation (separate document) for setting up automatic daily submissions at 6 PM.

---

### Q9: What if my supplier is not in the mapping?

**A:** Contact IT department with:
- Supplier Code (e.g., 4000/A999)
- Supplier Name
- IFCAP Creditor ID (if known)

IT will add the mapping and inform you when ready.

---

### Q10: How do I export search results?

**A:** The API returns JSON. You can:
1. Copy the response
2. Paste into Excel using "Data → From JSON"
3. Or use the pagination to get all pages:
```
page 1: ?pageSize=100&pageNumber=1
page 2: ?pageSize=100&pageNumber=2
page 3: ?pageSize=100&pageNumber=3
```

---

## Best Practices

### 1. Daily Routine
✅ Submit yesterday's invoices every morning
✅ Use automated scheduling (Task Scheduler)
✅ Review logs for failures

### 2. Month-End Process
✅ Search for pending invoices first
✅ Review the count and date range
✅ Submit in batches by week
✅ Verify all submissions before closing

### 3. Error Handling
✅ Always review batch results
✅ Fix common issues first (creditor mappings)
✅ Resubmit failed invoices promptly
✅ Keep IT informed of recurring issues

### 4. Performance
✅ Submit during off-peak hours
✅ Use smaller date ranges for large batches
✅ Don't submit more than 100 invoices at once

### 5. Monitoring
✅ Check logs regularly
✅ Monitor pending invoice count
✅ Track submission success rate
✅ Report patterns to IT

---

## Support

### Need Help?

**For Technical Issues:**
- API not responding
- Connection errors
- System errors

**Contact:** IT Support  
**Email:** support@company.com  
**Phone:** Extension 1234

**For Business Issues:**
- Missing creditor mappings
- Invoice approval workflow
- IFCAP access

**Contact:** Finance Department  
**Email:** finance@company.com

---

### Reporting Issues

When reporting issues, include:
1. ✅ Endpoint used
2. ✅ Parameters sent
3. ✅ Error message received
4. ✅ Invoice DocKey or DocNo
5. ✅ Screenshot (if applicable)
6. ✅ Date and time of issue

---

**Document Version:** 1.0  
**Last Updated:** January 23, 2025  
**Next Review:** February 2025
