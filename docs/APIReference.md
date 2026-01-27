# Invoice Integration API - User Documentation

**Version:** 1.1  
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
✅ **Resubmit Edited Invoices** - Replace existing invoices with updated data  
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

**Description:** Submits a single invoice from AutoCount to IFCAP. Will fail if the invoice already exists in IFCAP.

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
  "message": "Invoice already exists in IFCAP (JournalID: 5001). Use /api/invoices/resubmit endpoint to replace it."
}
```

---

### 2. 🆕 Resubmit Invoice (Replace Existing)

**Endpoint:** `POST /api/invoices/resubmit`

**Description:** Resubmits an invoice that was edited in AutoCount after initial submission. This endpoint will:
- Delete the old invoice from IFCAP365 (including all line items)
- Create a new invoice with the updated data from AutoCount
- Update the AutoCount status

**When to use this:**
- Invoice was edited in AutoCount after being submitted to IFCAP
- Need to correct mistakes in a submitted invoice
- Line items were added/removed/modified

**Parameters:**
| Name | Type | Required | Description |
|------|------|----------|-------------|
| docKey | integer | Yes | AutoCount invoice DocKey |

**Example Request:**
```
POST /api/invoices/resubmit?docKey=12345
```

**Success Response (200 OK) - Old invoice replaced:**
```json
{
  "message": "Invoice resubmitted successfully. Old JournalID 5001 deleted, new JournalID: 5050",
  "journalID": 5050
}
```

**Success Response (200 OK) - No old invoice found:**
```json
{
  "message": "Invoice submitted successfully (no existing version found). JournalID: 5050",
  "journalID": 5050
}
```

**Error Response (400 Bad Request):**
```json
{
  "message": "Failed to delete old invoice (JournalID: 5001) from IFCAP"
}
```

⚠️ **Important Notes:**
- This performs a **hard delete** - the old invoice is permanently removed from IFCAP
- All line items associated with the old invoice are also deleted
- The new invoice gets a new JournalID
- AutoCount status is updated to "Y" (Submitted) with the new JournalID

---

### 3. Batch Submit by Date Range

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
      "message": "Invoice already exists in IFCAP (JournalID: 5002). Use /api/invoices/resubmit endpoint to replace it.",
      "journalID": null
    }
  ]
}
```

---

### 4. Batch Submit by Status

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

### 5. Search Invoices

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

### 6. Get Invoice Status

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

### 7. Approve Invoice

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

### 8. Reject Invoice

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

### Example 2: 🆕 Resubmit an Edited Invoice

**Scenario:** An invoice was submitted to IFCAP, but then the amount was corrected in AutoCount.

**Step 1 - Check current status:**
```
GET /api/invoices/status/12345
```

**Response:**
```json
{
  "docKey": 12345,
  "docNo": "INV-001",
  "docStatus": "Y",
  "ifcapJournalID": 5001,
  "statusDescription": "Submitted/Pending in IFCAP"
}
```

**Step 2 - Resubmit with updated data:**
```
POST /api/invoices/resubmit?docKey=12345
```

**Response:**
```json
{
  "message": "Invoice resubmitted successfully. Old JournalID 5001 deleted, new JournalID: 5050",
  "journalID": 5050
}
```

**What happened:**
- Old invoice (JournalID 5001) deleted from IFCAP
- New invoice (JournalID 5050) created with correct data
- AutoCount status updated with new JournalID

**Use Case:** 
- Invoice amounts corrected after submission
- Line items added/removed after submission
- Supplier code changed after submission

---

### Example 3: Month-End Closing

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

### Example 4: Submit Specific Supplier Invoices

**Scenario:** Submit all invoices from T & T CAHAYA MURNI SDN BHD for January.

**Request:**
```
POST /api/invoices/batch/submit-by-date?fromDate=2025-01-01&toDate=2025-01-31&creditorCode=4000/T001
```

**Use Case:** Supplier-specific processing or audit requests.

---

### Example 5: Check Pending Invoices

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

### Example 6: Handle Duplicate Submission Errors

**Scenario:** Batch submission shows "already submitted" errors.

**Step 1 - Review batch results:**
```json
{
  "results": [
    {
      "docKey": 12346,
      "docNo": "INV-002",
      "success": false,
      "message": "Invoice already exists in IFCAP (JournalID: 5002). Use /api/invoices/resubmit endpoint to replace it."
    }
  ]
}
```

**Step 2 - Check if invoice was edited:**
- Open invoice 12346 in AutoCount
- Verify if changes were made after submission

**Step 3 - If edited, use resubmit:**
```
POST /api/invoices/resubmit?docKey=12346
```

**Use Case:** Handling invoices that were edited after initial submission.

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

#### 2. "Invoice already exists in IFCAP"

**Cause:** This invoice was already sent to IFCAP previously.

**Full Error:**
```json
{
  "message": "Invoice already exists in IFCAP (JournalID: 5001). Use /api/invoices/resubmit endpoint to replace it."
}
```

**Solution:** 
- **If NOT edited:** Check IFCAP using the JournalID provided. No action needed if this is a duplicate.
- **If edited in AutoCount:** Use the `/api/invoices/resubmit` endpoint to replace the old invoice with updated data.

**Example - Resubmit:**
```
POST /api/invoices/resubmit?docKey=12345
```

---

#### 3. "Failed to delete old invoice from IFCAP"

**Cause:** During resubmission, the API couldn't delete the old invoice from IFCAP.

**Solution:** 
- Contact IT support immediately
- Provide the DocKey and JournalID
- There may be database constraints preventing deletion

---

#### 4. "Cannot submit cancelled invoice"

**Cause:** The invoice is marked as cancelled in AutoCount.

**Solution:** Cancelled invoices cannot be submitted. Verify the invoice status in AutoCount.

---

#### 5. "No line items found for invoice"

**Cause:** The invoice has no detail lines in AutoCount.

**Solution:** Add line items to the invoice in AutoCount before submitting.

---

#### 6. "Invoice not found or not in pending status (Y)"

**Cause:** Trying to approve/reject an invoice that:
- Doesn't exist
- Hasn't been submitted yet (status N)
- Was already approved (status A)
- Was already rejected (status R)

**Solution:** Check the invoice status first using the status endpoint.

---

#### 7. "ToDate must be greater than or equal to FromDate"

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

### Issue: Invoice was edited after submission

**Symptoms:**
- Invoice submitted to IFCAP
- Changes made in AutoCount
- IFCAP has old data

**Solution:**
```
POST /api/invoices/resubmit?docKey=12345
```

This will delete the old invoice and create a new one with current data.

---

### Issue: Batch submission shows "already exists" errors

**Cause:** Some invoices were previously submitted.

**Solution:**
1. Review the batch results
2. Identify invoices with "already exists" errors
3. For each:
   - Check if it was edited after submission
   - If yes, use `/resubmit`
   - If no, no action needed

---

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

### Q1: What's the difference between `/submit` and `/resubmit`?

**A:**
- **`/submit`**: First-time submission. Fails if invoice already exists in IFCAP.
- **`/resubmit`**: Replaces existing invoice. Deletes old version and creates new one with updated data.

**Use `/submit` for:** New invoices  
**Use `/resubmit` for:** Edited invoices

---

### Q2: Will resubmit affect the approval status?

**A:** Yes. The resubmitted invoice will have status "Y" (Pending) and will need to be approved again, even if the old invoice was already approved.

---

### Q3: Can I undo a resubmission?

**A:** No. The old invoice is permanently deleted from IFCAP. The resubmission creates a completely new invoice with a new JournalID.

---

### Q4: How do I know which invoices need to be submitted?

**A:** Use the search endpoint with status `N`:
```
GET /api/invoices/search?status=N
```

This shows all invoices that haven't been submitted yet.

---

### Q5: Can I submit invoices from multiple months at once?

**A:** Yes, but it's not recommended for performance reasons. Better to submit in monthly batches:
```
# January
POST /api/invoices/batch/submit-by-date?fromDate=2025-01-01&toDate=2025-01-31

# February
POST /api/invoices/batch/submit-by-date?fromDate=2025-02-01&toDate=2025-02-28
```

---

### Q6: What happens if I submit the same invoice twice?

**A:** The `/submit` endpoint prevents duplicates. If an invoice was already submitted, you'll get an error:
```json
{
  "message": "Invoice already exists in IFCAP (JournalID: 5001). Use /api/invoices/resubmit endpoint to replace it."
}
```

---

### Q7: How long does batch submission take?

**A:** Approximately:
- 1-2 seconds per invoice
- 10 invoices = ~15 seconds
- 100 invoices = ~2-3 minutes

The API includes small delays between invoices to prevent database overload.

---

### Q8: What's the difference between status Y and A?

**A:**
- **Y (Submitted/Pending):** Invoice sent to IFCAP, waiting for approval
- **A (Approved):** Invoice has been reviewed and approved

Workflow: `N → Y → A` (or `R` if rejected)

---

### Q9: How do I find invoices submitted in the last week?

**A:**
```
GET /api/invoices/search?fromDate=2025-01-15&toDate=2025-01-22&status=Y
```

Change dates to your desired week.

---

### Q10: Can I schedule automatic submissions?

**A:** Yes! See the Task Scheduler documentation (separate document) for setting up automatic daily submissions at 6 PM.

---

## Best Practices

### 1. Daily Routine
✅ Submit yesterday's invoices every morning  
✅ Use automated scheduling (Task Scheduler)  
✅ Review logs for failures  
✅ Use `/resubmit` for any edited invoices

### 2. Month-End Process
✅ Search for pending invoices first  
✅ Review the count and date range  
✅ Submit in batches by week  
✅ Handle "already exists" errors with `/resubmit` if needed  
✅ Verify all submissions before closing

### 3. Error Handling
✅ Always review batch results  
✅ Fix common issues first (creditor mappings)  
✅ Use `/resubmit` for edited invoices  
✅ Resubmit failed invoices promptly  
✅ Keep IT informed of recurring issues

### 4. When to Use Resubmit
✅ Invoice amount was corrected after submission  
✅ Line items were added/removed after submission  
✅ Supplier code was changed after submission  
✅ Any data change in AutoCount after IFCAP submission

### 5. Performance
✅ Submit during off-peak hours  
✅ Use smaller date ranges for large batches  
✅ Don't submit more than 100 invoices at once  
✅ Monitor IFCAP database performance

### 6. Monitoring
✅ Check logs regularly  
✅ Monitor pending invoice count  
✅ Track submission success rate  
✅ Track resubmission frequency  
✅ Report patterns to IT

---

## Support

### Need Help?

**For Technical Issues:**
- API not responding
- Connection errors
- System errors
- Resubmit failures

**Contact:** IT Support  
**Email:** support@company.com  
**Phone:** Extension 1234

**For Business Issues:**
- Missing creditor mappings
- Invoice approval workflow
- IFCAP access
- When to resubmit vs. new submission

**Contact:** Finance Department  
**Email:** finance@company.com

---

### Reporting Issues

When reporting issues, include:
1. ✅ Endpoint used (`/submit` or `/resubmit`)
2. ✅ Parameters sent
3. ✅ Error message received
4. ✅ Invoice DocKey or DocNo
5. ✅ Whether invoice was edited after submission
6. ✅ Old and new JournalID (for resubmit issues)
7. ✅ Screenshot (if applicable)
8. ✅ Date and time of issue

---

**Document Version:** 1.1  
**Last Updated:** January 27, 2025  
**Changes:** Added resubmit endpoint documentation  
**Next Review:** February 2025
