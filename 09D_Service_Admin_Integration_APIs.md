# API Specification Part D: Service, Admin & Integration APIs
## Torvan Medical CleanStation Production Workflow Digitalization

**Version:** 1.0
**Date:** December 2024
**Document Type:** API Specification - Part D
**Scope:** Service Department, Administration, Notifications, QR Codes, Reporting, Data Export, System Integration APIs (including WebSockets)

---

## Executive Summary

This document (Part D) provides detailed API specifications for Service Department operations, System Administration functionalities, User Notifications, QR Code Management, Reporting, Data Export, and other system-level Integration APIs (like WebSockets) within the CleanStation digitalization system. It builds upon the core framework defined in Part A (API Core, Authentication & Data Models).

This document is part of a multi-part API specification series:
- **Part A**: Core API, Authentication & Data Models
- **Part B**: Order & BOM Management APIs
- **Part C**: Quality Control & Assembly APIs
- **Part D (This Document)**: Service, Admin & Integration APIs

Refer to Part A for foundational information on API architecture, authentication, standard error handling, and core data models. Refer to Parts B and C for their respective API domains.

---

## API Endpoint Specifications

This section details the endpoints for Service, Admin, Notifications, QR Codes, Reports, Exports, and WebSockets. All endpoints are prefixed by `/api/v1` as defined in Part A.

### Notification Endpoints (`/notifications`)
*(Detailed specifications for Notification endpoints will be added here. These were formerly in Part A.)*
- `GET /notifications` (Get notifications for the current user)
- `POST /notifications/send` (System-level to send notifications - Admin/System)
- `PUT /notifications/{notificationId}/read` (Mark a notification as read)
- `DELETE /notifications/{notificationId}` (Delete a notification)

### Service Department Endpoints (`/service`)
*(Detailed specifications for Service Department endpoints will be added here. These were formerly in Part A.)*
- `POST /service/requests` (Create a service part request)
- `GET /service/requests` (List service part requests)
- `GET /service/requests/{requestId}` (Get details of a service request)
- `PUT /service/requests/{requestId}/status` (Update status of a service request)

### QR Code Management Endpoints (`/qr`)
*(Detailed specifications for QR Code Management endpoints will be added here. These were formerly in Part A.)*
- `POST /qr/generate/{orderId}` (Generate QR code for an order/build)
- `GET /qr/{qrId}/info` (Get information linked to a QR ID)
- `GET /qr/codes/order/{orderId}` (List QR codes for an order)

### Administration Endpoints (`/admin`)
*(Detailed specifications for System Administration endpoints will be added here.)*
- `/admin/users` (already covered in Part A, but admin-specific user management actions might be here)
- `/admin/config` (Get/Set system-wide configurations)
- `/admin/roles` (Manage roles and permissions - if not covered by user management)
- `/admin/auditlog` (Access system audit logs - more extensive than user-facing history)
- `/admin/maintenance` (Trigger maintenance tasks)

### Reporting Endpoints (`/reports`)
*(Detailed specifications for Reporting endpoints will be added here. These were formerly in Part A.)*
- `POST /reports/generate` (Generate a report based on type or configuration)
- `GET /reports/status/{reportId}` (Check status of a generated report)
- `GET /reports/download/{reportId}` (Download a completed report)
- `GET /reports/configurations` (List available report configurations)
- `POST /reports/configurations` (Create/Update report configurations)

### Data Export Endpoints (`/export`)
*(Detailed specifications for Data Export endpoints will be added here.)*
- `POST /export/data` (Request a data export, e.g., orders in a date range)
- `GET /export/status/{exportId}`
- `GET /export/download/{exportId}`

### WebSocket Endpoints (`/websocket`)
*(Detailed specifications for WebSocket communication for real-time updates will be added here.)*
- `wss://api.cleanstation.torvan.com/v1/ws` (Base WebSocket connection URL)
- Topics/Events: (e.g., `order_updates`, `notification_new`, `task_status_change`)

### Document Management Endpoints (`/documents`) - General/Admin
*(Detailed specifications for general or administrative Document Management if not covered in Part C. This could include managing document types, storage policies, etc.)*

---

## Data Models Specific to Part D

This section defines data models primarily used within Service, Admin, Notifications, QR, Reports, and Exports.

```typescript
// Notification Models
// (e.g., NotificationType, Notification, SendNotificationPayload - to be moved/defined here from original Part A)

// Service Department Models
// (e.g., ServiceUrgency, ServiceRequestType, ServiceRequestStatus, ServiceCustomerInfo, RequestedServiceItem, ServicePartRequest, ServiceOrder_DB - to be moved/defined here from original Part A)

// QR Code Models
// (e.g., QRType, QRCodeRecord, GenerateQRRequest - to be moved/defined here from original Part A)

// Reporting & Export Models
// (e.g., ReportType, ReportConfiguration, GenerateReportPayload, GeneratedReportStatus - to be moved/defined here from original Part A)

// Admin specific models (e.g. for system configurations)

// WebSocket message structures
```

---

## Error Handling Specific to Part D

Refer to Part A for standard error response objects and common HTTP status codes. Specific application error codes relevant to these domains will be listed here.

*(Specific error codes to be added here)*

--- 