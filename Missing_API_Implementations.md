# Missing API Implementation Details for Documentation

This document outlines the missing items and areas that require completion to fully populate the refactored API specification documents (Parts A, B, C, and D). This list is based on the planned distribution of content from the original comprehensive API document and the goal of creating focused, sectional API guides.

**Note:** Successful completion of many of these items, particularly for Part B, is currently blocked by issues encountered when attempting to apply edits to `09B_Order_BOM_Management_APIs.md`.

---

## 1. For `09B_Order_BOM_Management_APIs.md` (Order & BOM Management)

### 1.1. API Endpoint Specifications:
The main endpoints for `/orders` and `/boms` are outlined, but detailed specifications for each sub-endpoint need to be fully developed and documented. This includes, but is not limited to:
- **Order Endpoints (under `/orders`):**
    - `GET /orders`: Ensure all query parameters, authorization details, and response structures are fully documented.
    - `POST /orders`: Confirm request body (`CreateOrderPayload`) details and response (`Order`).
    - `GET /orders/{orderId}`: Document response structure (`Order`).
    - `PUT /orders/{orderId}`: Detail request body (`UpdateOrderPayload`) and response (`Order`).
    - `PUT /orders/{orderId}/status`: Detail request body (`UpdateOrderStatusPayload`) and response (`Order`), including logic for status transitions.
    - `DELETE /orders/{orderId}`: Clarify soft vs. hard delete policies.
- **BOM Endpoints (under `/orders/{orderId}/bom` and `/boms`):**
    - `POST /orders/{orderId}/bom`: Detail request body (`GenerateBOMOptions`), response (`BOMStructure`), and logic for regeneration/versioning.
    - `GET /orders/{orderId}/bom`: Document query parameters (e.g., `version`) and response (`BOMStructure`).
    - `GET /boms/{bomId}`: Document response (`BOMStructure`).
    - `GET /boms/pending-review`: Detail query parameters and response (`PaginatedBOMReviewList`).
    - `POST /boms/{bomId}/review`: Detail request body (`BOMReviewActionPayload`) and response (`BOMReview_DB` or `BOMStructure`).

### 1.2. Data Models Specific to Part B:
The "Data Models Specific to Part B" section needs to be comprehensively populated with all relevant data models and enums for:
-   **Order Management:**
    -   `UpdateOrderPayload`
    -   `UpdateOrderStatusPayload`
    -   (Review `CreateOrderPayload` for any B-specific components)
    -   Full `Order` model details (if extensions from Part A are needed)
    -   `OrderListItem` (if extensions from Part A are needed)
    -   `OrderHistory`
    -   `OrderDocument`
    -   Related interfaces/enums moved from or based on `09A_API_Core_Authentication_Models.md`: `OrderStepOne`, `OrderStepTwo`, `BuildNumberItem`, `SinkConfigurationPayload`, `DimensionSpec`, `PegboardConfig`, `CustomBasinSize`, `StandardBasinSize`, `BasinAddon`, `BasinConfigurationItem`, `FaucetConfigurationItem`, `SprayerConfigurationItem`, `AccessorySelectionItem`.
-   **BOM Management:**
    -   `GenerateBOMOptions`
    -   Full `BOMStructure` model details
    -   `BOMItem`
    -   `BOMApproval`
    -   `BOMReview_DB` (confirm fields)
    -   `BOMReviewRequest`
    -   `BOMReviewItem` (confirm fields)
    -   `PaginatedBOMReviewList` (structure)
    -   `BOMReviewActionPayload`
-   **Catalog Management (relevant to Order/BOM context):**
    -   `Category`, `Subcategory`
    -   `Part` (and related enums `PartType`, `PartStatus`)
    -   `Assembly` (and related enums `AssemblyType`, `AssemblyCatalogStatus`, and interface `KitComponentDetails`)
    -   `AssemblyComponent`
    -   `Accessory`
-   **Outsourcing Management:**
    -   `OutsourcingStatus` (enum)
    -   `OutsourcedPart`
    -   `IdentifyOutsourcingPayload`
    -   `UpdateOutsourcingStatusPayload`

### 1.3. Error Handling Specific to Part B:
Populate this section with specific application error codes and scenarios relevant to order and BOM management (e.g., `BOM_GENERATION_FAILED`, `INVALID_ORDER_STATE_FOR_BOM`, `BOM_REVIEW_CONFLICT`, `ORDER_UPDATE_INVALID_STATE`).

---

## 2. For `09C_Quality_Control_Assembly_APIs.md` (Quality Control & Assembly)

### 2.1. API Endpoint Specifications:
Detail all endpoints under each category, including authorization, request/response bodies, query parameters, and examples.
-   **Pre-Quality Control (PreQC) Endpoints (`/preqc`):**
    -   `GET /preqc/templates`
    -   `GET /preqc/pending`
    -   `POST /preqc/{orderId}/complete` (or `/preqc/orders/{orderId}/builds/{buildNumber}/complete`)
    -   `GET /preqc/{orderId}/results` (or more granular)
-   **Assembly & Production Workflow Endpoints (`/assembly`, `/production`):**
    -   `GET /assembly/tasks` (available/assigned)
    -   `POST /assembly/tasks/{taskId}/start`
    -   `POST /assembly/tasks/{taskId}/complete`
    -   `GET /production/checklists/templates`
    -   `POST /production/checklists/orders/{orderId}/submit`
    -   `GET /production/progress/orders/{orderId}`
-   **Testing Management Endpoints (`/testing`):**
    -   `GET /testing/templates`
    -   `POST /testing/orders/{orderId}/results` (submit or retrieve)
-   **Packaging Management Endpoints (`/packaging`):**
    -   `GET /packaging/requirements/orders/{orderId}`
    -   `POST /packaging/orders/{orderId}/complete`
-   **Final Quality Control (FinalQC) Endpoints (`/finalqc`):**
    -   `GET /finalqc/templates`
    -   `POST /finalqc/orders/{orderId}/submit`
    -   `GET /finalqc/orders/{orderId}/results`
-   **Document Management Endpoints (`/documents`) related to QC & Assembly:**
    -   `POST /documents/upload` (with context for QC/Assembly)
    -   `GET /documents/orders/{orderId}/qc`

### 2.2. Data Models Specific to Part C:
Move, define, and detail all relevant data models and enums from the original `09A_API_Core_Authentication_Models.md` or new ones specific to Part C.
-   **Pre-Quality Control (PreQC) Models:** `QCFormType` (enum), `QCFormTemplate`, `QCResultStatus` (enum), `QCResult`, `PreQCFormPayload`.
-   **Assembly & Production Workflow Models:** `ProductionChecklistSection` (enum), `ProductionChecklistPayload`, `WorkInstruction`, `WorkInstructionStep`, `Tool`, `AssemblyTaskType` (enum), `AssemblyTaskStatus` (enum), `AssemblyTaskRequiredPart`, `AssemblyTaskRequiredTool`, `AssemblyTaskQualityCheck`, `AssemblyTask`, `TaskCompletion_DB`, `ProductionPhase` (enum), `ProductionStatus` (enum), `AssemblyProgress`.
-   **Testing Models:** `TestingFormTemplate`, `TestProcedureType` (enum), `TestProcedureResultStatus` (enum), `TestingProcedure`, `TestingResult_DB`.
-   **Packaging Models:** `PackagingType` (enum), `PackagingRequirement`.
-   **Final Quality Control (FinalQC) Models:** `FinalQCPayload`, `FinalQCResult`.
-   **Document Models (related to QC/Assembly):** Specific structures for QC evidence if different from generic `DocumentReference` in Part A.

### 2.3. Error Handling Specific to Part C:
Populate with specific error codes (e.g., `QC_CHECKLIST_VALIDATION_FAILED`, `ASSEMBLY_TASK_NOT_FOUND`, `TEST_PROCEDURE_ERROR`, `PACKAGING_REQUIREMENT_MISSING`).

---

## 3. For `09D_Service_Admin_Integration_APIs.md` (Service, Admin & Integration)

### 3.1. API Endpoint Specifications:
Detail all endpoints under each category.
-   **Notification Endpoints (`/notifications`):**
    -   `GET /notifications`
    -   `POST /notifications/send`
    -   `PUT /notifications/{notificationId}/read`
    -   `DELETE /notifications/{notificationId}`
-   **Service Department Endpoints (`/service`):**
    -   `POST /service/requests`
    -   `GET /service/requests`
    -   `GET /service/requests/{requestId}`
    -   `PUT /service/requests/{requestId}/status`
-   **QR Code Management Endpoints (`/qr`):**
    -   `POST /qr/generate/{orderId}`
    -   `GET /qr/{qrId}/info`
    -   `GET /qr/codes/order/{orderId}`
-   **Administration Endpoints (`/admin`):**
    -   `GET /admin/config`, `POST /admin/config`
    -   `GET /admin/auditlog` (and any POST for configuration)
    -   `POST /admin/maintenance/trigger`
    -   (User/Role management links to Part A, specify any admin-only views/actions if distinct here)
-   **Reporting Endpoints (`/reports`):**
    -   `POST /reports/generate`
    -   `GET /reports/status/{reportId}`
    -   `GET /reports/download/{reportId}`
    -   `GET /reports/configurations`
    -   `POST /reports/configurations`
-   **Data Export Endpoints (`/export`):**
    -   `POST /export/data`
    -   `GET /export/status/{exportId}`
    -   `GET /export/download/{exportId}`
-   **WebSocket Endpoints (`/websocket`):**
    -   Define base URL (`wss://...`)
    -   Authentication mechanism for WebSockets.
    -   Key topics/event structures (e.g., `order_updates`, `notification_new`).
-   **Document Management Endpoints (`/documents`) - General/Admin:**
    -   Define if any general/admin-specific document APIs are needed beyond Parts B/C.

### 3.2. Data Models Specific to Part D:
Move, define, and detail all relevant data models and enums.
-   **Notification Models:** `NotificationType` (enum), `Notification`, `SendNotificationPayload`.
-   **Service Department Models:** `ServiceUrgency` (enum), `ServiceRequestType` (enum), `ServiceRequestStatus` (enum), `ServiceCustomerInfo`, `RequestedServiceItem`, `ServicePartRequest`, `ServiceOrder_DB`.
-   **QR Code Models:** `QRType` (enum), `QRCodeRecord`, `GenerateQRRequest`.
-   **Reporting & Export Models:** `ReportType` (enum), `ReportConfiguration`, `GenerateReportPayload`, `GeneratedReportStatus`.
-   **Admin specific models:** (e.g., for system configurations, audit log filter parameters).
-   **WebSocket message structures:** Define expected payloads for different events.

### 3.3. Error Handling Specific to Part D:
Populate with specific error codes (e.g., `NOTIFICATION_SEND_FAILED`, `SERVICE_REQUEST_INVALID_STATE`, `REPORT_GENERATION_UNAVAILABLE`, `WEBSOCKET_AUTH_FAILED`).

--- 