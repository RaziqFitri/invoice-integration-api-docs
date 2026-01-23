# Invoice Integration API Documentation

![API Version](https://img.shields.io/badge/version-1.0-blue)
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
✅ **Batch Transfer by Date** - Submit all invoices in a date range  
✅ **Batch Transfer by Status** - Submit all pending invoices  
✅ **Search & Filter** - Find invoices by date and status with pagination  
✅ **Status Tracking** - Monitor invoice submission status  
✅ **Approve/Reject** - Manage invoice approvals  
✅ **Automated Scheduling** - Daily automatic submissions

---

## 🎯 Who Should Use This?

- **Finance Team** - Daily invoice processing
- **Accounting Department** - Month-end closing
- **IT Support** - Setup and maintenance
- **System Administrators** - Configuration and monitoring

---

## 📋 Prerequisites

- Access to AutoCount database
- Access to IFCAP365 database
- Network connectivity to API server
- Web browser or API testing tool

---

## 🔗 API Endpoint
```
http://your-server-name:5000/api/invoices
```

**Swagger UI (Interactive Documentation):**
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
4. [Batch Submit by Date](docs/APIReference.md#2-batch-submit-by-date-range)
5. [Batch Submit by Status](docs/APIReference.md#3-batch-submit-by-status)
6. [Search Invoices](docs/APIReference.md#4-search-invoices)
7. [Get Invoice Status](docs/APIReference.md#5-get-invoice-status)
8. [Approve Invoice](docs/APIReference.md#6-approve-invoice)
9. [Reject Invoice](docs/APIReference.md#7-reject-invoice)

### Automation
10. [Task Scheduler Setup](docs/TaskScheduler.md) - Automate daily submissions

### Support
11. [Troubleshooting Guide](docs/Troubleshooting.md)
12. [FAQ](docs/APIReference.md#faq)

---

## 🎓 Common Use Cases

### Daily Operations
```
POST /api/invoices/batch/submit-by-date?fromDate=2025-01-22&toDate=2025-01-22
```
Submit yesterday's invoices every morning.

### Month-End Closing
```
POST /api/invoices/batch/submit-by-date?fromDate=2025-01-01&toDate=2025-01-31
```
Submit all invoices for the month.

### Check Pending Count
```
GET /api/invoices/search?status=N&pageSize=1
```
See how many invoices are waiting to be submitted.

---

## 📊 Status Codes

| Code | Description |
|------|-------------|
| `N` | Not Submitted - Invoice in AutoCount only |
| `Y` | Submitted - Pending in IFCAP |
| `A` | Approved - Invoice approved |
| `R` | Rejected - Invoice rejected with reason |

---

## 🔧 System Architecture
```
┌─────────────┐         ┌──────────────┐         ┌─────────────┐
│  AutoCount  │ ------> │ Integration  │ ------> │   IFCAP365  │
│  (Source)   │         │     API      │         │ (Destination)│
└─────────────┘         └──────────────┘         └─────────────┘
```

---

## 📞 Support

### Technical Issues
**Contact:** IT Support  
**Email:** support@company.com  
**Phone:** Ext. 1234

### Business Issues
**Contact:** Finance Department  
**Email:** finance@company.com

---

## 📝 Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | Jan 2025 | Initial release with core features |

---

## 📄 License

Internal use only - Company Confidential

---

## 🤝 Contributing

Found an issue or have a suggestion? 
- Create an [Issue](../../issues)
- Contact IT Support

---

**Last Updated:** January 23, 2025  
**Maintained by:** IT Department
