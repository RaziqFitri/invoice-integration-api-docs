# Invoice Integration API Documentation

![API Version](https://img.shields.io/badge/version-1.1-blue)
![Status](https://img.shields.io/badge/status-active-green)

## 📖 Overview

This repository contains comprehensive documentation for the **Invoice Integration API**, which automates invoice transfers between **AutoCount** and **IFCAP365** systems.

---

## 🚀 Quick Links

- 📘 [**Quick Start Guide**](docs/QuickStart.md) - Get started in 5 minutes
- 📗 [**API Reference**](docs/APIReference.md) - Complete endpoint documentation
- ⏰ [**Task Scheduler Setup**](docs/TaskScheduler.md) - Automate daily submissions
- 🔧 [**Troubleshooting**](docs/Troubleshooting.md) - Common issues and solutions

---

## ✨ Features

✅ **Single Invoice Submission** - Submit one invoice at a time  
✅ **Resubmit Edited Invoices** - Replace existing invoices with updated data (NEW!)  
✅ **Batch Transfer by Date** - Submit all invoices in a date range  
✅ **Batch Transfer by Status** - Submit all pending invoices  
✅ **Search & Filter** - Find invoices by date and status with pagination  
✅ **Status Tracking** - Monitor invoice submission status  
✅ **Approve/Reject** - Manage invoice approvals  
✅ **Automated Scheduling** - Daily automatic submissions

---

## 🆕 What's New in v1.1

### Resubmit Feature
Now you can replace invoices that were edited in AutoCount after submission!

**Use Case:** Invoice submitted to IFCAP, then amount corrected in AutoCount
```http
POST /api/invoices/resubmit?docKey=12345
```

**What happens:**
- 🗑️ Old invoice deleted from IFCAP
- ✅ New invoice created with updated data
- 🔄 AutoCount status updated

[Learn more →](docs/APIReference.md#2-resubmit-invoice-replace-existing)

---

## 🎯 Who Should Use This?

- **Finance Team** - Daily invoice processing and corrections
- **Accounting Department** - Month-end closing and data accuracy
- **IT Support** - Setup and maintenance
- **System Administrators** - Configuration and monitoring

---

## 📋 Prerequisites

- Access to AutoCount database
- Access to IFCAP365 database
- Network connectivity to API server
- Web browser or API testing tool

---

## 🔗 API Endpoints

```
Base URL: http://your-server-name:5000/api/invoices
```

### Core Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/submit` | Submit new invoice to IFCAP |
| POST | `/resubmit` | **🆕** Replace edited invoice in IFCAP |
| GET | `/status/{docKey}` | Check invoice status |
| GET | `/search` | Search invoices with filters |
| POST | `/approve` | Approve submitted invoice |
| POST | `/reject` | Reject submitted invoice |
| POST | `/batch/submit-by-date` | Batch submit by date range |
| POST | `/batch/submit-by-status` | Batch submit by status |

**Interactive Documentation (Swagger UI):**
```
http://your-server-name:5000/swagger
```

---

## 📚 Documentation Index

### Getting Started
1. [Quick Start Guide](docs/QuickStart.md) - New users start here
2. [Installation & Setup](docs/APIReference.md#getting-started)

### API Reference
3. [Submit Single Invoice](docs/APIReference.md#1-submit-single-invoice)
4. [**🆕 Resubmit Invoice**](docs/APIReference.md#2-resubmit-invoice-replace-existing)
5. [Batch Submit by Date](docs/APIReference.md#3-batch-submit-by-date-range)
6. [Batch Submit by Status](docs/APIReference.md#4-batch-submit-by-status)
7. [Search Invoices](docs/APIReference.md#5-search-invoices)
8. [Get Invoice Status](docs/APIReference.md#6-get-invoice-status)
9. [Approve Invoice](docs/APIReference.md#7-approve-invoice)
10. [Reject Invoice](docs/APIReference.md#8-reject-invoice)

### Automation
11. [Task Scheduler Setup](docs/TaskScheduler.md) - Automate daily submissions

### Support
12. [Troubleshooting Guide](docs/Troubleshooting.md)
13. [FAQ](docs/APIReference.md#faq)

---

## 🎓 Common Use Cases

### Daily Operations
```http
POST /api/invoices/batch/submit-by-date?fromDate=2025-01-26&toDate=2025-01-26
```
Submit yesterday's invoices every morning.

### 🆕 Fix Edited Invoice
```http
POST /api/invoices/resubmit?docKey=12345
```
Replace invoice in IFCAP after editing in AutoCount.

### Month-End Closing
```http
POST /api/invoices/batch/submit-by-date?fromDate=2025-01-01&toDate=2025-01-31
```
Submit all invoices for the month.

### Check Pending Count
```http
GET /api/invoices/search?status=N&pageSize=1
```
See how many invoices are waiting to be submitted.

### Search Submitted Invoices
```http
GET /api/invoices/search?status=Y&fromDate=2025-01-01&toDate=2025-01-31
```
View all submitted invoices pending approval.

---

## 📊 Status Codes

| Code | Description | Next Action |
|------|-------------|-------------|
| `N` | Not Submitted | Invoice in AutoCount only → Submit |
| `Y` | Submitted/Pending | In IFCAP awaiting approval → Approve/Reject |
| `A` | Approved | Invoice approved → No action |
| `R` | Rejected | Invoice rejected → Review and resubmit |

---

## 🔄 Submit vs. Resubmit

| Scenario | Use `/submit` | Use `/resubmit` |
|----------|---------------|-----------------|
| First submission | ✅ | ❌ |
| Invoice edited after submission | ❌ | ✅ |
| Amount corrected in AutoCount | ❌ | ✅ |
| Line items changed | ❌ | ✅ |
| Already in IFCAP (no edits) | ❌ | ❌ (No action needed) |

**Simple Rule:**
- **First time?** → Use `/submit`
- **Edited after submission?** → Use `/resubmit`

---

## 🔧 System Architecture

```
┌─────────────┐         ┌──────────────┐         ┌─────────────┐
│  AutoCount  │ ------> │ Integration  │ ------> │   IFCAP365  │
│  (Source)   │         │     API      │         │ (Destination)│
└─────────────┘         └──────────────┘         └─────────────┘
      ↑                                                   ↓
      └───────────── Resubmit (Replace) ─────────────────┘
```

**Key Features:**
- Real-time data transfer
- Automatic creditor mapping
- Status synchronization
- **🆕 Replace edited invoices**
- Error handling and retry logic

---

## 💡 Quick Examples

### Example 1: Submit New Invoice
```bash
curl -X POST "http://your-server:5000/api/invoices/submit?docKey=12345"
```

### Example 2: 🆕 Resubmit Edited Invoice
```bash
curl -X POST "http://your-server:5000/api/invoices/resubmit?docKey=12345"
```

### Example 3: Check Status
```bash
curl -X GET "http://your-server:5000/api/invoices/status/12345"
```

### Example 4: Search Pending Invoices
```bash
curl -X GET "http://your-server:5000/api/invoices/search?status=N&pageSize=10"
```

---

## 🚨 Common Issues & Solutions

### Issue: "Invoice already exists in IFCAP"

**Was the invoice edited in AutoCount?**
- ✅ **Yes** → Use `/resubmit` to replace it
- ❌ **No** → No action needed (already in IFCAP)

### Issue: "Creditor mapping not found"

**Solution:** Contact IT with supplier code and name

### Issue: "Failed to delete old invoice from IFCAP"

**Solution:** Contact IT immediately - do not retry

[See full troubleshooting guide →](docs/Troubleshooting.md)

---

## 📞 Support

### Technical Issues
**Contact:** IT Support  
**Email:** support@company.com  
**Phone:** Ext. 1234

**Report issues with:**
- Endpoint used (`/submit` or `/resubmit`)
- DocKey and error message
- Screenshot
- Whether invoice was edited

### Business Issues
**Contact:** Finance Department  
**Email:** finance@company.com

---

## 📝 Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.1 | Jan 27, 2025 | Added resubmit endpoint for edited invoices |
| 1.0 | Jan 23, 2025 | Initial release with core features |

---

## 🎯 Getting Started

1. **Read the [Quick Start Guide](docs/QuickStart.md)** (5 minutes)
2. **Access Swagger UI** at `http://your-server:5000/swagger`
3. **Submit your first invoice** using DocKey
4. **Learn the resubmit workflow** for edited invoices
5. **Set up automation** with Task Scheduler

---

## 📖 Best Practices

### Daily Workflow
1. ✅ Submit yesterday's invoices every morning
2. ✅ Review batch results for failures
3. ✅ **🆕 Immediately resubmit any edited invoices**
4. ✅ Report creditor mapping errors to IT
5. ✅ Verify submissions in IFCAP

### Data Accuracy
1. ✅ Verify invoice data before submission
2. ✅ **🆕 If you edit an invoice after submission, resubmit it**
3. ✅ Don't submit cancelled invoices
4. ✅ Ensure line items are complete
5. ✅ Double-check supplier codes

### Error Handling
1. ✅ Read error messages carefully
2. ✅ Distinguish between "duplicate" and "needs resubmit"
3. ✅ Don't blindly retry failed submissions
4. ✅ Contact IT for recurring errors
5. ✅ Keep logs of resubmissions

---

## 🤝 Contributing

Found an issue or have a suggestion?
- Create an [Issue](../../issues)
- Contact DT Support
- Submit documentation improvements

---

## 📄 License

Internal use only - Company Confidential

---

## 🔗 Related Resources

- **AutoCount Documentation:** [Link]
- **IFCAP365 User Guide:** [Link]
- **Company IT Portal:** [Link]

---

## ⭐ Key Highlights

- ✨ **Automated Integration** - No manual data entry
- 🎯 **Accurate Data Sync** - Real-time updates
- 🔄 **Edit Support** - Resubmit changed invoices
- 📊 **Full Tracking** - Monitor every submission
- 🚀 **Batch Processing** - Handle multiple invoices
- 🔒 **Secure** - Database-level authentication
- 📱 **Easy to Use** - REST API with Swagger UI

---

**Last Updated:** January 27, 2025  
**Maintained by:** DT Department  
**Support:**
