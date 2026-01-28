# Invoice Integration API - API Reference

**Version:** 1.2  
**Last Updated:** January 28, 2025  
**Support Contact:** IT Department

---

## Table of Contents

1. [Overview](#overview)
2. [Getting Started](#getting-started)
3. [Invoice Endpoints](#invoice-endpoints)
4. [Audit Endpoints](#audit-endpoints)
5. [Status Codes](#status-codes)
6. [Error Handling](#error-handling)
7. [FAQ](#faq)

---

## Overview

### What is the Invoice Integration API?

The Invoice Integration API bridges AutoCount and IFCAP365, automating invoice transfers between systems with **complete audit tracking**.

### Key Features

✅ **Single Invoice Submission** - Submit one invoice at a time  
✅ **Resubmit Edited Invoices** - Replace existing invoices with updated data  
✅ **Batch Transfer by Date** - Submit all invoices in a date range  
✅ **Batch Transfer by Status** - Submit all pending invoices  
✅ **Search & Filter** - Find invoices by date and status  
✅ **Status Tracking** - Monitor invoice submission status  
✅ **Approve/Reject** - Manage invoice approvals  
✅ **🆕 Complete Audit Trail** - Every action is logged with full details

---

## Getting Started

### Base URL

```
http://your-server-name:5000/api
```

### Accessing Swagger UI

```
http://your-server-name:5000/swagger
```

### Date Format

**Always use:** `YYYY-MM-DD`

**Examples:**
- ✅ `2025-01-28`
- ✅ `2024-12-31`
- ❌ `28/01/2025`
- ❌ `01-28-2025`

---

## Invoice Endpoints

### 1. Submit Single Invoice

**Endpoint:** `POST /api/invoices/submit`

**Description:** Submits a single invoice from AutoCount to IFCAP.

**Parameters:**
| Name | Type | Required | Description |
|------|------|----------|-------------|
| docKey | integer | Yes | AutoCount invoice DocKey |

**Example:**
```http
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

**Audit Log:** Records submission with timestamp, user, IP, and details.

---

### 2. Resubmit Invoice

**Endpoint:** `POST /api/invoices/resubmit`

**Description:** Replaces an existing invoice in IFCAP with updated data from AutoCount.

**When to use:**
- Invoice was edited after initial submission
- Amount corrected
- Line items added/removed
- Any data change in AutoCount

**Parameters:**
| Name | Type | Required | Description |
|------|------|----------|-------------|
| docKey | integer | Yes | AutoCount invoice DocKey |

**Example:**
```http
POST /api/invoices/resubmit?docKey=12345
```

**Success Response (200 OK):**
```json
{
  "message": "Invoice resubmitted successfully. Old JournalID 5001 deleted, new JournalID: 5050",
  "journalID": 5050
}
```

**What Happens:**
1. Old invoice (JournalID 5001) deleted from IFCAP ✗
2. New invoice (JournalID 5050) created ✓
3. AutoCount status updated
4. **Two audit logs created:** Delete + Resubmit

---

### 3. Approve Invoice

**Endpoint:** `POST /api/invoices/approve`

**Description:** Approves an invoice (changes status from Y → A).

**Parameters:**
| Name | Type | Required | Description |
|------|------|----------|-------------|
| docKey | integer | Yes | AutoCount invoice DocKey |

**Example:**
```http
POST /api/invoices/approve?docKey=12345
```

**Success Response (200 OK):**
```json
{
  "message": "Invoice approved successfully"
}
```

**Audit Log:** Records approval action.

---

### 4. Reject Invoice

**Endpoint:** `POST /api/invoices/reject`

**Description:** Rejects an invoice with a reason (changes status from Y → R).

**Parameters:**
| Name | Type | Required | Description |
|------|------|----------|-------------|
| docKey | integer | Yes | AutoCount invoice DocKey |
| reason | string | Yes | Rejection reason |

**Example:**
```http
POST /api/invoices/reject?docKey=12345&reason=Incorrect amount
```

**Success Response (200 OK):**
```json
{
  "message": "Invoice rejected successfully"
}
```

**Audit Log:** Records rejection with reason.

---

### 5. Get Invoice Status

**Endpoint:** `GET /api/invoices/status/{docKey}`

**Description:** Get current status of a specific invoice.

**Example:**
```http
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

---

### 6. Search Invoices

**Endpoint:** `GET /api/invoices/search`

**Description:** Search invoices with filters and pagination.

**Parameters:**
| Name | Type | Required | Description |
|------|------|----------|-------------|
| fromDate | date | No | Start date (YYYY-MM-DD) |
| toDate | date | No | End date (YYYY-MM-DD) |
| status | string | No | Filter by status (N, Y, A, R) |
| pageSize | integer | No | Records per page (default: 50, max: 1000) |
| pageNumber | integer | No | Page number (default: 1) |

**Example:**
```http
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

### 7. Batch Submit by Date Range

**Endpoint:** `POST /api/invoices/batch/submit-by-date`

**Description:** Submits all pending invoices within a date range.

**Parameters:**
| Name | Type | Required | Description |
|------|------|----------|-------------|
| fromDate | date | Yes | Start date (YYYY-MM-DD) |
| toDate | date | Yes | End date (YYYY-MM-DD) |
| creditorCode | string | No | Filter by specific creditor |

**Example:**
```http
POST /api/invoices/batch/submit-by-date?fromDate=2025-01-01&toDate=2025-01-31
```

**With Creditor Filter:**
```http
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

**Audit Log:** Each invoice gets individual audit entry.

---

### 8. Batch Submit by Status

**Endpoint:** `POST /api/invoices/batch/submit-by-status`

**Description:** Submits all invoices with a specific status.

**Parameters:**
| Name | Type | Required | Description |
|------|------|----------|-------------|
| status | string | No | Status code (default: N) |

**Valid Status Values:** N, NULL, Y, A, R

**Example:**
```http
POST /api/invoices/batch/submit-by-status?status=N
```

**Response:** Same format as batch submit by date.

⚠️ **Warning:** This can process hundreds or thousands of invoices!

---

## Audit Endpoints

### 1. Get Audit Logs

**Endpoint:** `GET /api/audit/logs`

**Description:** Retrieve audit logs with optional filters.

**Parameters:**
| Name | Type | Required | Description |
|------|------|----------|-------------|
| fromDate | date | No | Start date (YYYY-MM-DD) |
| toDate | date | No | End date (YYYY-MM-DD) |
| action | string | No | Filter by action type |
| docKey | integer | No | Filter by specific invoice |

**Action Types:**
- `Submit` - Initial submission
- `Resubmit` - Edited invoice resubmission
- `Delete` - Deletion (during resubmit)
- `Approve` - Invoice approval
- `Reject` - Invoice rejection

**Example:**
```http
GET /api/audit/logs?fromDate=2025-01-01&toDate=2025-01-31&action=Resubmit
```

**Success Response (200 OK):**
```json
{
  "totalRecords": 15,
  "data": [
    {
      "logDate": "2025-01-28T14:30:00",
      "action": "Resubmit",
      "docKey": 12345,
      "docNo": "INV-001",
      "oldJournalID": 5001,
      "newJournalID": 5050,
      "status": "Y",
      "success": true,
      "userName": "Anonymous",
      "ipAddress": "192.168.1.100",
      "details": "Old JournalID: 5001, New JournalID: 5050, Amount: 1600.00"
    }
  ]
}
```

**Use Cases:**
- Generate monthly reports
- Track all activity
- Filter by action type
- Export for analysis

---

### 2. Get Invoice History

**Endpoint:** `GET /api/audit/invoice/{docKey}`

**Description:** Get complete history for a specific invoice.

**Example:**
```http
GET /api/audit/invoice/12345
```

**Success Response (200 OK):**
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
      "details": "Amount: 1500.00, Lines: 3"
    },
    {
      "logDate": "2025-01-28T14:30:00",
      "action": "Resubmit",
      "oldJournalID": 5001,
      "newJournalID": 5050,
      "success": true,
      "details": "Amount: 1600.00, Lines: 4"
    },
    {
      "logDate": "2025-01-28T16:00:00",
      "action": "Approve",
      "success": true
    }
  ]
}
```

**Use Cases:**
- Track invoice lifecycle
- Verify if invoice was edited
- Audit trail for compliance
- Troubleshoot issues
- See who did what

---

### 3. Get Today's Summary

**Endpoint:** `GET /api/audit/summary/today`

**Description:** Get summary of today's API activity.

**Example:**
```http
GET /api/audit/summary/today
```

**Success Response (200 OK):**
```json
{
  "date": "2025-01-28",
  "totalActions": 45,
  "successCount": 42,
  "failureCount": 3,
  "byAction": [
    {
      "action": "Submit",
      "count": 20,
      "success": 18,
      "failed": 2
    },
    {
      "action": "Resubmit",
      "count": 15,
      "success": 15,
      "failed": 0
    },
    {
      "action": "Approve",
      "count": 10,
      "success": 9,
      "failed": 1
    }
  ],
  "recentFailures": [
    {
      "logDate": "2025-01-28T14:25:00",
      "action": "Submit",
      "docKey": 12350,
      "docNo": "INV-050",
      "errorMessage": "Creditor mapping not found for: 4000/Z999"
    }
  ]
}
```

**Use Cases:**
- Daily monitoring
- Quick health check
- Identify issues early
- Track success rate
- Review failures

---

### 4. Get Resubmit History

**Endpoint:** `GET /api/audit/resubmits`

**Description:** Get all invoices that were resubmitted (edited after initial submission).

**Parameters:**
| Name | Type | Required | Description |
|------|------|----------|-------------|
| fromDate | date | No | Start date (default: 7 days ago) |
| toDate | date | No | End date |

**Example:**
```http
GET /api/audit/resubmits?fromDate=2025-01-20&toDate=2025-01-28
```

**Success Response (200 OK):**
```json
{
  "totalResubmits": 12,
  "successfulResubmits": 12,
  "failedResubmits": 0,
  "data": [
    {
      "logDate": "2025-01-28T14:30:00",
      "docKey": 12345,
      "docNo": "INV-001",
      "oldJournalID": 5001,
      "newJournalID": 5050,
      "success": true,
      "details": "Old JournalID: 5001, New JournalID: 5050, Amount: 1600.00"
    }
  ]
}
```

**Use Cases:**
- Track which invoices were edited
- Monitor data quality
- Identify patterns
- Compliance reporting

---

### 5. Get Deletion History

**Endpoint:** `GET /api/audit/deletions`

**Description:** Get all invoices deleted from IFCAP (during resubmit operations).

**Parameters:**
| Name | Type | Required | Description |
|------|------|----------|-------------|
| fromDate | date | No | Start date (default: 7 days ago) |
| toDate | date | No | End date |

**Example:**
```http
GET /api/audit/deletions?fromDate=2025-01-20&toDate=2025-01-28
```

**Success Response (200 OK):**
```json
{
  "totalDeletions": 12,
  "data": [
    {
      "logDate": "2025-01-28T14:30:01",
      "action": "Delete",
      "docKey": 12345,
      "docNo": "INV-001",
      "oldJournalID": 5001,
      "success": true,
      "details": "Deleted as part of resubmit operation"
    }
  ]
}
```

**Use Cases:**
- Track what was removed
- Audit trail for deletions
- Compliance requirements
- Link to resubmissions

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

**Cause:** Supplier code not in configuration.

**Solution:** Contact IT to add the creditor mapping.

**Example:**
```json
{
  "message": "Creditor mapping not found for: 4000/Z999"
}
```

---

#### 2. "Invoice already exists in IFCAP"

**Full Error:**
```json
{
  "message": "Invoice already exists in IFCAP (JournalID: 5001). Use /api/invoices/resubmit endpoint to replace it."
}
```

**Solution:**
- **If NOT edited:** No action needed
- **If edited in AutoCount:** Use `/api/invoices/resubmit`

---

#### 3. "Failed to delete old invoice from IFCAP"

**Cause:** During resubmission, couldn't delete old invoice.

**Solution:**
- Contact IT immediately
- Provide DocKey and JournalID
- Do not retry

---

#### 4. "Cannot submit cancelled invoice"

**Cause:** Invoice is marked as cancelled in AutoCount.

**Solution:** Verify invoice status in AutoCount.

---

#### 5. "No line items found for invoice"

**Cause:** Invoice has no detail lines.

**Solution:** Add line items to the invoice in AutoCount before submitting.

---

#### 6. "Invoice not found or not in pending status (Y)"

**Cause:** Trying to approve/reject an invoice that:
- Doesn't exist
- Hasn't been submitted yet
- Was already approved
- Was already rejected

**Solution:** Check invoice status first.

---

## FAQ

### Q1: How do I know if an invoice was edited after submission?

**A:** Check the audit log:
```http
GET /api/audit/invoice/{docKey}
```
Look for "Resubmit" action.

---

### Q2: Can I see who submitted an invoice?

**A:** Yes! Audit logs track username and IP:
```http
GET /api/audit/invoice/{docKey}
```

---

### Q3: How do I generate a monthly audit report?

**A:**
```http
GET /api/audit/logs?fromDate=2025-01-01&toDate=2025-01-31
```
Export JSON and process in Excel or use SQL queries.

---

### Q4: What gets logged when I resubmit?

**A:** Two entries:
1. **Delete** - Old invoice removal
2. **Resubmit** - New invoice creation

Both include Old/New JournalIDs.

---

### Q5: Can I see all invoices that were edited?

**A:** Yes!
```http
GET /api/audit/resubmits
```

---

### Q6: How do I track errors?

**A:**
```http
GET /api/audit/summary/today
```
Check the `recentFailures` section.

---

### Q7: How long are audit logs kept?

**A:** Currently indefinitely. Future: 2-year retention.

---

### Q8: What's the difference between /submit and /resubmit?

**A:**
- **`/submit`**: First-time submission. Fails if already exists.
- **`/resubmit`**: Replaces existing. Deletes old and creates new.

---

### Q9: Are deletions tracked?

**A:** Yes!
```http
GET /api/audit/deletions
```

---

### Q10: Can I filter audit logs by user?

**A:** Not directly via API, but you can query the database:
```sql
SELECT * FROM InvoiceAuditLog WHERE UserName = 'username';
```

---

## Support

**Technical Issues:**  
IT Support - support@company.com - Ext. 1234

**Business Issues:**  
Finance Department - finance@company.com

---

**Document Version:** 1.2  
**Last Updated:** January 28, 2025  
**Changes:** Added complete audit logging documentation
