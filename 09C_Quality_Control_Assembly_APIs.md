# API Specification Part C: Quality Control & Assembly APIs
## Torvan Medical CleanStation Production Workflow Digitalization

**Version:** 1.0
**Date:** December 2024
**Document Type:** API Specification - Part C
**Scope:** Quality Control (PreQC, In-Process QC, Final QC), Assembly Workflow, Production Checklists, Testing, Packaging, Document Management (related to QC/Assembly)

---

## Executive Summary

This document (Part C) provides detailed API specifications for Quality Control processes (including Pre-Quality Control, In-Process Assembly QC, and Final Quality Control), Assembly and Production workflows, Production Checklists, Testing procedures, Packaging, and related Document Management within the CleanStation digitalization system. It builds upon the core framework defined in Part A (API Core, Authentication & Data Models).

This document is part of a multi-part API specification series:
- **Part A**: Core API, Authentication & Data Models
- **Part B**: Order & BOM Management APIs
- **Part C (This Document)**: Quality Control & Assembly APIs
- **Part D**: Service Department, Admin & Integration APIs

Refer to Part A for foundational information on API architecture, authentication, standard error handling, and core data models. Refer to Part B for Order and BOM specific APIs.

---

## API Endpoint Specifications

This section details the endpoints for Quality Control, Assembly, Testing, and Packaging. All endpoints are prefixed by `/api/v1` as defined in Part A.

### Pre-Quality Control (PreQC) Endpoints (`/preqc`)
*(Detailed specifications for PreQC endpoints will be added here. These were formerly in Part A.)*
- `GET /preqc/templates`
- `GET /preqc/pending`
- `POST /preqc/{orderId}/complete` (or more granular like `/preqc/orders/{orderId}/builds/{buildNumber}/complete`)
- `GET /preqc/{orderId}/results` (or more granular)

### Assembly & Production Workflow Endpoints (`/assembly`, `/production`)
*(Detailed specifications for Assembly and Production Workflow endpoints will be added here.)*
- `/api/v1/assembly/tasks` (List available/assigned tasks)
- `/api/v1/assembly/tasks/{taskId}/start`
- `/api/v1/assembly/tasks/{taskId}/complete`
- `/api/v1/production/checklists/templates`
- `/api/v1/production/checklists/orders/{orderId}/submit` (Submit completed checklist for an order/build)
- `/api/v1/production/progress/orders/{orderId}` (Get assembly progress)

### Testing Management Endpoints (`/testing`)
*(Detailed specifications for Testing Management endpoints will be added here.)*
- `/api/v1/testing/templates`
- `/api/v1/testing/orders/{orderId}/results` (Submit or retrieve test results)

### Packaging Management Endpoints (`/packaging`)
*(Detailed specifications for Packaging Management endpoints will be added here.)*
- `/api/v1/packaging/requirements/orders/{orderId}`
- `/api/v1/packaging/orders/{orderId}/complete`

### Final Quality Control (FinalQC) Endpoints (`/finalqc`)
*(Detailed specifications for FinalQC endpoints will be added here.)*
- `/api/v1/finalqc/templates`
- `/api/v1/finalqc/orders/{orderId}/submit`
- `/api/v1/finalqc/orders/{orderId}/results`

### Document Management Endpoints (`/documents`) - Related to QC & Assembly
*(Detailed specifications for Document Management endpoints relevant to QC and Assembly, if not fully covered in Part D, will be added here. This may include endpoints for uploading evidence, QA reports, etc.)*
- `POST /documents/upload` (with context for QC/Assembly)
- `GET /documents/orders/{orderId}/qc` (Get QC related documents for an order)


---

## Data Models Specific to Part C

This section defines data models that are primarily used or extended within the scope of QC, Assembly, Testing, and Packaging.

```typescript
// Pre-Quality Control (PreQC) Models
// (e.g., QCFormType, QCFormTemplate, QCResultStatus, QCResult, PreQCFormPayload - to be moved/defined here from original Part A)

// Assembly & Production Workflow Models
// (e.g., ProductionChecklistSection, ProductionChecklistPayload, WorkInstruction, WorkInstructionStep, AssemblyTaskType, AssemblyTaskStatus, AssemblyTaskRequiredPart, AssemblyTaskRequiredTool, AssemblyTaskQualityCheck, AssemblyTask, TaskCompletion_DB, ProductionPhase, ProductionStatus, AssemblyProgress - to be moved/defined here)

// Testing Models
// (e.g., TestingFormTemplate, TestProcedureType, TestingProcedure, TestingResult_DB - to be moved/defined here)

// Packaging Models
// (e.g., PackagingType, PackagingRequirement - to be moved/defined here)

// Final Quality Control (FinalQC) Models
// (e.g., FinalQCPayload, FinalQCResult - to be moved/defined here)

// Document Models (related to QC/Assembly)
// (e.g., specific structures for QC evidence if different from generic DocumentReference in Part A)
```

---

## Error Handling Specific to Part C

Refer to Part A for standard error response objects and common HTTP status codes. Specific application error codes relevant to QC, Assembly, etc., will be listed here.

*(Specific error codes to be added here)*

--- 