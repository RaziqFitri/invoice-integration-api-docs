# Invoice Integration API - User Documentation

**Version:** 1.2  
**Last Updated:** January 28, 2025  
**Support Contact:** IT Department

---

## Table of Contents

1. [Overview](#overview)
2. [Getting Started](#getting-started)
3. [API Endpoints](#api-endpoints)
4. [Audit & Tracking](#audit--tracking)
5. [Usage Examples](#usage-examples)
6. [Status Codes](#status-codes)
7. [Error Handling](#error-handling)
8. [Troubleshooting](#troubleshooting)
9. [FAQ](#faq)

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

### Accessing Swagger UI

For interactive documentation and testing:

```
http://your-server-name:5000/swagger
```

---

## API Endpoints

### Invoice Operations

#### 1. Submit Single Invoice

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

**What gets logged:**
- ✅ Action: Submit
- ✅ DocKey: 12345
- ✅ New JournalID: 5001
- ✅ Status: Y (Submitted)
- ✅ Success: true
- ✅ User IP and timestamp

---

#### 2. Resubmit Invoice (Replace Existing)

**Endpoint:** `POST /api/invoices/resubmit`

**Description:** Resubmits an invoice that was edited in AutoCount after initial submission. This endpoint will:
- Delete the old invoice from IFCAP365 (logged as "Delete" action)
- Create a new invoice with the updated data (logged as "Resubmit" action)
- Update the AutoCount status

**Parameters:**
| Name | Type | Required | Description |
|------|------|----------|-------------|
| docKey | integer | Yes | AutoCount invoice DocKey |

**Example Request:**
```
POST /api/invoices/resubmit?docKey=12345
```

**Success Response (200 OK):**
```json
{
  "message": "Invoice resubmitted successfully. Old JournalID 5001 deleted, new JournalID: 5050",
  "journalID": 5050
}
```

**What gets logged:**
- ✅ Action: Delete (old invoice removal)
- ✅ Action: Resubmit (new invoice creation)
- ✅ Old JournalID: 5001
- ✅ New JournalID: 5050
- ✅ Full audit trail maintained

---

#### 3. Batch Submit by Date Range

**Endpoint:** `POST /api/invoices/batch/submit-by-date`

**Parameters:**
| Name | Type | Required | Description |
|------|------|----------|-------------|
| fromDate | date | Yes | Start date (YYYY-MM-DD) |
| toDate | date | Yes | End date (YYYY-MM-DD) |
| creditorCode | string | No | Filter by specific creditor |

**Example Request:**
```
POST /api/invoices/batch/submit-by-date?fromDate=2025-01-01&toDate=2025-01-31
```

**What gets logged:**
Each invoice in the batch gets its own audit entry with success/failure status.

---

#### 4. Batch Submit by Status

**Endpoint:** `POST /api/invoices/batch/submit-by-status`

**Parameters:**
| Name | Type | Required | Description |
|------|------|----------|-------------|
| status | string | No | Status code (default: N) |

**Valid Status Values:** N, NULL, Y, A, R

---

#### 5. Search Invoices

**Endpoint:** `GET /api/invoices/search`

**Parameters:**
| Name | Type | Required | Description |
|------|------|----------|-------------|
| fromDate | date | No | Start date (YYYY-MM-DD) |
| toDate | date | No | End date (YYYY-MM-DD) |
| status | string | No | Filter by status (N, Y, A, R) |
| pageSize | integer | No | Records per page (default: 50, max: 1000) |
| pageNumber | integer | No | Page number (default: 1) |

---

#### 6. Get Invoice Status

**Endpoint:** `GET /api/invoices/status/{docKey}`

**Example Request:**
```
GET /api/invoices/status/12345
```

---

#### 7. Approve Invoice

**Endpoint:** `POST /api/invoices/approve`

**Parameters:**
| Name | Type | Required | Description |
|------|------|----------|-------------|
| docKey | integer | Yes | AutoCount invoice DocKey |

**What gets logged:**
- ✅ Action: Approve
- ✅ Status changed: Y → A
- ✅ Timestamp and user

---

#### 8. Reject Invoice

**Endpoint:** `POST /api/invoices/reject`

**Parameters:**
| Name | Type | Required | Description |
|------|------|----------|-------------|
| docKey | integer | Yes | AutoCount invoice DocKey |
| reason | string | Yes | Rejection reason |

**What gets logged:**
- ✅ Action: Reject
- ✅ Status changed: Y → R
- ✅ Rejection reason stored
- ✅ Timestamp and user

---

## 🆕 Audit & Tracking

### Overview

Every action in the API is automatically logged to provide:
- ✅ Complete audit trail for compliance
- ✅ Error tracking and debugging
- ✅ User activity monitoring
- ✅ Performance metrics
- ✅ Resubmission history

### Audit Endpoints

#### 1. Get All Audit Logs

**Endpoint:** `GET /api/audit/logs`

**Description:** Retrieve filtered audit logs

**Parameters:**
| Name | Type | Required | Description |
|------|------|----------|-------------|
| fromDate | date | No | Start date (YYYY-MM-DD) |
| toDate | date | No | End date (YYYY-MM-DD) |
| action | string | No | Filter by action type |
| docKey | integer | No | Filter by specific invoice |

**Action Types:**
- `Submit` - Initial invoice submission
- `Resubmit` - Invoice resubmission (edited)
- `Delete` - Invoice deletion (during resubmit)
- `Approve` - Invoice approval
- `Reject` - Invoice rejection

**Example Request:**
```
GET /api/audit/logs?fromDate=2025-01-01&toDate=2025-01-31&action=Resubmit
```

**Response:**
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

---

#### 2. Get Invoice History

**Endpoint:** `GET /api/audit/invoice/{docKey}`

**Description:** Get complete history for a specific invoice

**Example Request:**
```
GET /api/audit/invoice/12345
```

**Response:**
```json
{
  "docKey": 12345,
  "totalActions": 3,
  "history": [
    {
      "logDate": "2025-01-28T10:00:00",
      "action": "Submit",
      "newJournalID": 5001,
      "success": true
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

**Use Cases:**
- Track invoice lifecycle from creation to approval
- Identify which invoices were edited (resubmitted)
- Audit trail for compliance
- Troubleshoot submission issues

---

#### 3. Get Today's Summary

**Endpoint:** `GET /api/audit/summary/today`

**Description:** Get summary of today's API activity

**Example Request:**
```
GET /api/audit/summary/today
```

**Response:**
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
- Performance tracking
- Identify issues early
- Generate daily reports

---

#### 4. Get Resubmit History

**Endpoint:** `GET /api/audit/resubmits`

**Description:** Get all invoices that were resubmitted (edited after initial submission)

**Parameters:**
| Name | Type | Required | Description |
|------|------|----------|-------------|
| fromDate | date | No | Start date (default: 7 days ago) |
| toDate | date | No | End date |

**Example Request:**
```
GET /api/audit/resubmits?fromDate=2025-01-20&toDate=2025-01-28
```

**Response:**
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
- Identify patterns (why are invoices being edited?)
- Compliance reporting

---

#### 5. Get Deletion History

**Endpoint:** `GET /api/audit/deletions`

**Description:** Get all invoices deleted from IFCAP (during resubmit operations)

**Parameters:**
| Name | Type | Required | Description |
|------|------|----------|-------------|
| fromDate | date | No | Start date (default: 7 days ago) |
| toDate | date | No | End date |

**Example Request:**
```
GET /api/audit/deletions?fromDate=2025-01-20&toDate=2025-01-28
```

**Response:**
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
- Track what was removed from IFCAP
- Audit trail for deletions
- Compliance requirements
- Link deletions to resubmissions

---

## Usage Examples

### Example 1: Submit Invoice and Check Audit Log

**Step 1 - Submit invoice:**
```
POST /api/invoices/submit?docKey=12345
```

**Step 2 - Check if it was logged:**
```
GET /api/audit/invoice/12345
```

**Response shows:**
```json
{
  "docKey": 12345,
  "totalActions": 1,
  "history": [
    {
      "logDate": "2025-01-28T10:00:00",
      "action": "Submit",
      "newJournalID": 5001,
      "success": true,
      "details": "Amount: 1500.00, Creditor: 4000/T001, Lines: 3"
    }
  ]
}
```

---

### Example 2: Edit Invoice and Resubmit

**Scenario:** Invoice submitted with wrong amount, then corrected

**Step 1 - Original submission:**
```
POST /api/invoices/submit?docKey=12345
→ Creates JournalID 5001
```

**Step 2 - User edits in AutoCount:**
- Changes amount from RM 1,500 to RM 1,600
- Adds extra line item

**Step 3 - Resubmit:**
```
POST /api/invoices/resubmit?docKey=12345
→ Deletes JournalID 5001
→ Creates JournalID 5050
```

**Step 4 - Check history:**
```
GET /api/audit/invoice/12345
```

**Response shows complete trail:**
```json
{
  "totalActions": 3,
  "history": [
    {
      "action": "Submit",
      "newJournalID": 5001,
      "details": "Amount: 1500.00, Lines: 3"
    },
    {
      "action": "Delete",
      "oldJournalID": 5001,
      "details": "Deleted as part of resubmit"
    },
    {
      "action": "Resubmit",
      "oldJournalID": 5001,
      "newJournalID": 5050,
      "details": "Amount: 1600.00, Lines: 4"
    }
  ]
}
```

---

### Example 3: Daily Monitoring Workflow

**Every morning at 8:00 AM:**

**Step 1 - Check today's activity:**
```
GET /api/audit/summary/today
```

**Step 2 - Submit yesterday's invoices:**
```
POST /api/invoices/batch/submit-by-date?fromDate=2025-01-27&toDate=2025-01-27
```

**Step 3 - Check for failures:**
```
GET /api/audit/summary/today
→ Look at "recentFailures" section
```

**Step 4 - Fix failures:**
- Creditor mapping errors → Email IT
- "Already exists" errors → Check if edited → Resubmit if needed

**Step 5 - Verify all processed:**
```
GET /api/invoices/search?status=N
→ Should show minimal pending invoices
```

---

### Example 4: Month-End Audit Report

**Generate report for January 2025:**

**All activity:**
```
GET /api/audit/logs?fromDate=2025-01-01&toDate=2025-01-31
```

**Resubmissions only:**
```
GET /api/audit/resubmits?fromDate=2025-01-01&toDate=2025-01-31
```

**Failures only:**
Filter response by `success: false`

**Generate summary:**
- Total submissions: X
- Successful: Y
- Failed: Z
- Resubmitted: N
- Success rate: Y/X × 100%

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

All errors are automatically logged to the audit table with:
- Error message
- Timestamp
- DocKey
- User IP
- Action attempted

### Common Errors

See main API Reference documentation for detailed error handling.

---

## Troubleshooting

### Issue: Cannot Find Audit Logs

**Check:**
```sql
SELECT * FROM InvoiceAuditLog 
ORDER BY LogDate DESC;
```

If empty, the audit logging might not be working. Contact IT.

---

### Issue: Too Many Audit Records

**Solution:**
Use filters to narrow down:
```
GET /api/audit/logs?fromDate=2025-01-28&action=Submit
```

---

### Issue: Need to Find Who Did Something

**Solution:**
```
GET /api/audit/invoice/12345
```

Look at `userName` and `ipAddress` fields.

---

## FAQ

### Q1: How long are audit logs kept?

**A:** Currently indefinitely. In the future, logs older than 2 years may be archived.

---

### Q2: Can I see who submitted an invoice?

**A:** Yes! Use `GET /api/audit/invoice/{docKey}` and check the `userName` and `ipAddress` fields.

---

### Q3: How do I generate a monthly report?

**A:**
```
GET /api/audit/logs?fromDate=2025-01-01&toDate=2025-01-31
```

Export the JSON and process in Excel or use SQL queries.

---

### Q4: What gets logged when I resubmit?

**A:** Two entries:
1. **Delete** - Removal of old invoice from IFCAP
2. **Resubmit** - Creation of new invoice with updated data

Both entries include Old JournalID and New JournalID.

---

### Q5: Can I see all invoices that were edited?

**A:** Yes!
```
GET /api/audit/resubmits
```

This shows all invoices that were resubmitted (edited after initial submission).

---

### Q6: How do I track errors?

**A:**
```
GET /api/audit/summary/today
```

Look at the `recentFailures` section for today's errors.

---

## Best Practices

### 1. Daily Routine
✅ Check today's summary every morning  
✅ Review failures and fix promptly  
✅ Monitor resubmission frequency  
✅ Track success rate (should be >95%)

### 2. Weekly Review
✅ Review resubmit history (why are invoices being edited?)  
✅ Check for recurring errors  
✅ Verify no unexpected deletions  
✅ Monitor user activity

### 3. Monthly Reporting
✅ Generate full audit report  
✅ Calculate success rate  
✅ Track resubmission trends  
✅ Identify improvement areas  
✅ Archive if needed

### 4. Compliance
✅ Audit logs provide compliance trail  
✅ Every action has timestamp and user  
✅ Deletions are tracked  
✅ Changes are fully documented

---

## Support

### Need Help?

**For Technical Issues:**
- API not responding
- Audit logs not showing
- Connection errors

**Contact:** IT Support  
**Email:** support@company.com  
**Phone:** Extension 1234

**For Business Issues:**
- Understanding audit reports
- Compliance questions
- Process improvements

**Contact:** Finance Department  
**Email:** finance@company.com

---

### Reporting Issues

When reporting issues, include:
1. ✅ Endpoint used
2. ✅ Parameters sent
3. ✅ Error message received
4. ✅ DocKey or DocNo
5. ✅ Check audit logs: `GET /api/audit/invoice/{docKey}`
6. ✅ Screenshot
7. ✅ Date and time

---

**Document Version:** 1.2  
**Last Updated:** January 28, 2025  
**Changes:** Added complete audit logging documentation  
**Next Review:** February 2025
