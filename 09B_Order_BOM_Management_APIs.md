# API Specification Part B: Order & BOM Management
## Torvan Medical CleanStation Production Workflow Digitalization

**Version:** 1.0
**Date:** December 2024
**Document Type:** API Specification - Part B
**Scope:** Order Management, Bill of Materials (BOM) Management

---

## Executive Summary

This document (Part B) provides detailed API specifications for Order Management and Bill of Materials (BOM) Management within the CleanStation digitalization system. It builds upon the core framework defined in Part A (API Core, Authentication & Data Models) and outlines all endpoints, request/response payloads, and specific data models relevant to creating, tracking, and managing orders and their associated BOMs, including BOM generation and review processes.

This document is part of a multi-part API specification series:
- **Part A**: Core API, Authentication & Data Models
- **Part B (This Document)**: Order & BOM Management APIs
- **Part C**: Quality Control & Assembly APIs
- **Part D**: Service Department, Admin & Integration APIs

Refer to Part A for foundational information on API architecture, authentication, standard error handling, and core data models.

---

## API Endpoint Specifications

This section details the endpoints for Order Management and BOM Management. All endpoints are prefixed by `/api/v1` as defined in Part A.

### Order Management Endpoints (`/orders`)

These endpoints are used to manage customer orders, from creation through various stages of the production lifecycle. They correspond to the functionalities outlined in `04_Phase1_Foundation_Core_Order_Management.md`.

#### List Orders
- **Endpoint:** `GET /orders`
- **Description:** Retrieves a paginated list of orders. Access may be filtered based on user role (e.g., Production Coordinators see their assigned orders, Admins see all).
- **Authorization:** Requires roles like `Admin` (`orders:read_all`), `ProductionCoordinator` (`orders:read_assigned` or `orders:read_all`), `QCPerson` (`orders:read`), `ProcurementSpecialist` (`orders:read`). Specific permissions might include `orders:read_all` or `orders:read_assigned`.
- **Query Parameters:**
    - `page?: number` (Default: 1) - Page number for pagination.
    - `limit?: number` (Default: 20, Max: 100) - Number of orders per page.
    - `sortBy?: string` (Default: `createdAt`) - Field to sort by (e.g., `poNumber`, `customerName`, `status`, `wantDate`, `createdAt`).
    - `sortOrder?: 'asc' | 'desc'` (Default: `desc`).
    - `status?: OrderStatus` - Filter by one or more order statuses (comma-separated if multiple).
    - `sinkFamily?: SinkFamily` - Filter by sink family.
    - `salesPerson?: string` - Filter by sales person (exact match or partial).
    - `customerName?: string` - Filter by customer name (partial match).
    - `poNumber?: string` - Filter by PO number (exact match or partial).
    - `assignedToMe?: boolean` - (For non-Admins) Filter to orders assigned to the current user.
    - `dateFrom?: string` (ISO 8601 Date: `YYYY-MM-DD`) - Filter orders created on or after this date.
    - `dateTo?: string` (ISO 8601 Date: `YYYY-MM-DD`) - Filter orders created on or before this date.
- **Success Response (200 OK):** `PaginatedOrderListResponse`
  ```typescript
  // Defined in Part A, uses OrderListItem from Part A's Data Models
  interface PaginatedOrderListResponse {
    items: OrderListItem[];
    totalItems: number;
    totalPages: number;
    currentPage: number;
    itemsPerPage: number;
  }

  // OrderListItem (Reference from Part A - Data Models)
  // interface OrderListItem {
  //   orderId: string;
  //   poNumber: string;
  //   customerName: string;
  //   salesPerson: string;
  //   wantDate: string; // ISO 8601 Date
  //   sinkFamily: SinkFamily;
  //   status: OrderStatus;
  //   buildCount: number;
  //   createdAt: string; // ISO 8601 Datetime
  //   // currentAssigneeName?: string;
  // }
  ```
  ```json
  // Example Success Response
  {
    "items": [
      {
        "orderId": "a1b2c3d4-e5f6-7890-1234-567890abcdef",
        "poNumber": "PO12345",
        "customerName": "General Hospital",
        "salesPerson": "Alice Smith",
        "wantDate": "2024-12-20",
        "sinkFamily": "MDRD",
        "status": "OrderCreated",
        "buildCount": 2,
        "createdAt": "2024-11-01T10:00:00Z"
      }
      // ... more OrderListItem objects
    ],
    "totalItems": 25,
    "totalPages": 3,
    "currentPage": 1,
    "itemsPerPage": 10
  }
  ```
- **Error Responses:** `400 Bad Request` (invalid query parameters), `401 Unauthorized`, `403 Forbidden`.

#### Create Order
- **Endpoint:** `POST /orders`
- **Description:** Creates a new customer order based on the multi-step wizard information detailed in `04_Phase1_Foundation_Core_Order_Management.md`.
- **Authorization:** Requires `ProductionCoordinator` or `Admin` role with `orders:create` permission.
- **Request Body:** `CreateOrderPayload` (Defined in Part A - Data Models, composed of `OrderStepOne`, `OrderStepTwo`, `SinkConfigurationPayload[]`, `AccessorySelectionItem[]`)
  ```json
  // Example Request Body (structure from Part A - Data Models)
  {
    "stepOne": {
      "poNumber": "PO67890",
      "customerName": "City Clinic",
      "projectName": "New Wing Setup",
      "salesPerson": "Bob Johnson",
      "wantDate": "2025-01-15",
      "poDocumentUrl": "https://example.com/docs/po67890.pdf",
      "notes": "Urgent order, expedite if possible.",
      "documentLanguage": "EN"
    },
    "stepTwo": {
      "sinkFamily": "Endoscope",
      "quantity": 1,
      "buildNumbers": [ { "id": "client-temp-id-1", "buildNumber": "BN-001A" } ]
    },
    "sinkConfigurations": [
      {
        "buildNumber": "BN-001A",
        "sinkModel": "T2-B2", // Example
        "sinkDimensions": { "width": 72, "length": 30 },
        "legsType": "HeightAdjustable",
        "legsModel": "DL27",
        "feetType": "LockLevelingCasters",
        "pegboard": {
          "color": "Green",
          "type": "Perforated",
          "size": { "type": "SameAsSink" }
        },
        "workflowDirection": "LeftToRight",
        "basins": [
          { "index": 0, "type": "E-Sink", "size": { "type": "Standard", "model": "B16x19x10" }, "addons": [] },
          { "index": 1, "type": "E-Sink DI", "size": { "type": "Standard", "model": "B16x19x10" }, "addons": [{"addonId": "addon01", "name": "DI Rinse", "selected": true}] }
        ],
        "faucets": [ { "type": "faucet-model-01", "quantity": 2, "placement": "CenterBasin1" } ],
        "sprayers": [ { "type": "sprayer-model-01", "quantity": 1, "location": "LeftSide" } ]
      }
    ],
    "accessorySelections": [
      { "accessoryId": "acc001", "name": "Mounting Bracket", "category": "Brackets", "quantity": 2 }
    ]
  }
  ```
- **Success Response (201 Created):** `Order` (The full Order object, defined in Part A - Data Models)
  ```json
  // Example Success Response (structure from Part A - Data Models)
  {
    "orderId": "fgh1jkl-m2n3-4567-8901-p2q3r4stuvwx",
    "poNumber": "PO67890",
    "customerName": "City Clinic",
    "projectName": "New Wing Setup",
    "salesPerson": "Bob Johnson",
    "wantDate": "2025-01-15",
    "orderLanguage": "EN",
    "sinkFamily": "Endoscope",
    "sinkSpecificDetails": { /* ... derived from sinkConfigurations ... */ },
    "status": "OrderCreated",
    "createdBy": "user_uuid_coord",
    "createdAt": "2024-11-10T14:30:00Z",
    "updatedAt": "2024-11-10T14:30:00Z",
    "builds": [
        { /* ... full SinkConfigurationPayload for BN-001A ... */ }
    ]
    // ... other Order fields ...
  }
  ```
- **Error Responses:** `400 Bad Request` (validation errors in payload), `401 Unauthorized`, `403 Forbidden`.

#### Get Order by ID
- **Endpoint:** `GET /orders/{orderId}`
- **Description:** Retrieves detailed information for a specific order.
- **Authorization:** Users with `orders:read` permission. Access may be restricted to orders they are associated with (e.g., based on `salesPerson` or assignment), unless they have `orders:read_all`.
- **Path Parameters:**
    - `orderId: string` (UUID) - The ID of the order to retrieve.
- **Success Response (200 OK):** `Order` (The full Order object, defined in Part A - Data Models)
  ```json
  // Example Success Response (structure from Part A - Data Models)
  {
    "orderId": "a1b2c3d4-e5f6-7890-1234-567890abcdef",
    "poNumber": "PO12345",
    "customerName": "General Hospital",
    // ... all fields of the Order model, including detailed sinkSpecificDetails, builds, associated documents, history, etc.
    "status": "ReadyForProduction",
    "createdAt": "2024-11-01T10:00:00Z",
    "updatedAt": "2024-11-05T16:20:00Z"
  }
  ```
- **Error Responses:** `401 Unauthorized`, `403 Forbidden`, `404 Not Found`.

#### Update Order by ID
- **Endpoint:** `PUT /orders/{orderId}`
- **Description:** Updates general information for an existing order. Restricted to certain fields and order states. For status changes, use `PUT /orders/{orderId}/status`.
- **Authorization:** Users with `orders:update` permission (e.g., `ProductionCoordinator` for their orders, `Admin` for any).
- **Path Parameters:**
    - `orderId: string` (UUID) - The ID of the order to update.
- **Request Body:** `UpdateOrderPayload`
  ```typescript
  // UpdateOrderPayload (Specific to Part B - to be added to Data Models section)
  interface UpdateOrderPayload {
    poNumber?: string;
    customerName?: string;
    projectName?: string;
    salesPerson?: string;
    wantDate?: string; // ISO 8601 Date
    orderLanguage?: DocumentLanguage; // Enum from Part A
    notes?: string;
    // Updating sinkConfigurations or accessories might require specific conditions or a different endpoint/process
    // if the order is past a certain stage (e.g. BOM approved, in production).
  }
  ```
  ```json
  // Example Request Body
  {
    "customerName": "General Hospital - West Wing",
    "wantDate": "2024-12-22",
    "notes": "Updated delivery contact and expedited request."
  }
  ```
- **Success Response (200 OK):** `Order` (The updated Order object from Part A)
- **Error Responses:** `400 Bad Request` (validation errors), `401 Unauthorized`, `403 Forbidden`, `404 Not Found`, `409 Conflict` (if update conflicts with order state, e.g., trying to change PO number after BOM generation).

#### Update Order Status
- **Endpoint:** `PUT /orders/{orderId}/status`
- **Description:** Updates the status of an existing order. This is a dedicated endpoint for status transitions, which are often controlled by specific business logic, role permissions, and completion of prerequisite tasks (e.g., PreQC completed before moving to 'ReadyForProduction').
- **Authorization:** Users with specific permissions like `orders:update_status`, `orders:set_status_production`, `orders:set_status_shipped`, etc. Permissions may depend on the target status and user role.
- **Path Parameters:**
    - `orderId: string` (UUID) - The ID of the order whose status is to be updated.
- **Request Body:** `UpdateOrderStatusPayload`
  ```typescript
  // UpdateOrderStatusPayload (Specific to Part B - to be added to Data Models section)
  interface UpdateOrderStatusPayload {
    status: OrderStatus; // Enum from Part A
    notes?: string;      // Optional notes regarding the status change
    // May include additional fields based on status, e.g., qcCompletionId if moving to ReadyForProduction
  }
  ```
  ```json
  // Example Request Body
  {
    "status": "ReadyForPreQC",
    "notes": "All initial order details confirmed."
  }
  ```
- **Success Response (200 OK):** `Order` (The updated Order object from Part A)
- **Error Responses:** `400 Bad Request` (invalid status transition, missing prerequisites), `401 Unauthorized`, `403 Forbidden`, `404 Not Found`, `409 Conflict` (if status transition is not allowed from current state).

#### Delete Order by ID
- **Endpoint:** `DELETE /orders/{orderId}`
- **Description:** Deletes an order. This is typically a soft delete, marking the order as inactive or archived. Hard deletion might be restricted or require specific privileges.
- **Authorization:** Requires `Admin` role with `orders:delete` permission.
- **Path Parameters:**
    - `orderId: string` (UUID) - The ID of the order to delete.
- **Success Response (204 No Content):** No response body.
- **Error Responses:** `401 Unauthorized`, `403 Forbidden`, `404 Not Found`, `409 Conflict` (e.g., if order is in a non-deletable state like 'Shipped' or has financial transactions).

---

### Bill of Materials (BOM) Management Endpoints (`/boms` and `/orders/{orderId}/bom`)

These endpoints manage Bill of Materials (BOMs), including generation, retrieval, and the review/approval workflow. They correspond to functionalities outlined in `04_Phase1_Foundation_Core_Order_Management.md` and `05_Phase2_Procurement_PreQC_Workflow.md`.

#### Generate BOM for Order
- **Endpoint:** `POST /orders/{orderId}/bom` (Changed from `/generate-bom` for RESTfulness)
- **Description:** Generates or re-generates a Bill of Materials for a specific order based on its current configuration. If a BOM already exists, this might create a new version or update the existing one based on system policy.
- **Authorization:** Requires `ProductionCoordinator` or `Admin` role with `boms:generate` or `orders:manage_bom` permission.
- **Path Parameters:**
    - `orderId: string` (UUID) - The ID of the order for which to generate the BOM.
- **Request Body:** (Optional) `GenerateBOMOptions`
  ```typescript
  // GenerateBOMOptions (Specific to Part B - to be added to Data Models section)
  interface GenerateBOMOptions {
    force Regenerate?: boolean; // If true, creates a new BOM version even if not strictly necessary.
    versionNotes?: string;    // Notes for this specific BOM version.
  }
  ```
- **Success Response (201 Created or 200 OK):** `BOMStructure` (The generated or updated BOM, defined in Part A - Data Models)
  ```json
  // Example Success Response (BOMStructure from Part A)
  {
    "bomId": "bom_uuid_new_or_updated",
    "orderId": "a1b2c3d4-e5f6-7890-1234-567890abcdef",
    "buildNumber": "BN-001A", // Or could be an array if order has multiple builds and BOM is per build
    "items": [
      { "id": "part_xyz", "type": "PART", "name": "Specific Screw", "partNumber": "SCRW-001", "quantity": 10, "level": 1, "parent": "assembly_abc" },
      { "id": "assembly_abc", "type": "ASSEMBLY", "name": "Main Panel", "partNumber": "ASY-002", "quantity": 1, "level": 0 }
      // ... more BOMItems
    ],
    "totalUniqueParts": 50,
    "totalAssemblies": 5,
    "generatedAt": "2024-11-10T15:00:00Z",
    "generatedById": "user_uuid_coord",
    "status": "Generated", // Initial status
    "version": 1
  }
  ```
- **Error Responses:** `400 Bad Request` (e.g., order not in a state suitable for BOM generation), `401 Unauthorized`, `403 Forbidden`, `404 Not Found` (order not found).

#### Get BOM for Order
- **Endpoint:** `GET /orders/{orderId}/bom`
- **Description:** Retrieves the current/latest Bill of Materials associated with a specific order. May allow fetching specific versions via query parameters.
- **Authorization:** Requires `orders:read` or `boms:read` permission. Accessible by roles like `ProductionCoordinator`, `Admin`, `ProcurementSpecialist`, `QCPerson`, `Assembler`.
- **Path Parameters:**
    - `orderId: string` (UUID) - The ID of the order whose BOM is requested.
- **Query Parameters:**
    - `version?: number` - Optional: to retrieve a specific version of the BOM.
- **Success Response (200 OK):** `BOMStructure` (Defined in Part A - Data Models)
- **Error Responses:** `401 Unauthorized`, `403 Forbidden`, `404 Not Found` (order or BOM not found).

#### Get BOM by ID
- **Endpoint:** `GET /boms/{bomId}`
- **Description:** Retrieves a specific Bill of Materials by its unique BOM ID.
- **Authorization:** Requires `boms:read` permission.
- **Path Parameters:**
    - `bomId: string` (UUID) - The ID of the BOM to retrieve.
- **Success Response (200 OK):** `BOMStructure` (Defined in Part A - Data Models)
- **Error Responses:** `401 Unauthorized`, `403 Forbidden`, `404 Not Found`.

#### List BOMs Pending Review
- **Endpoint:** `GET /boms/pending-review`
- **Description:** Retrieves a paginated list of BOMs that are awaiting review by Procurement.
- **Authorization:** Requires `ProcurementSpecialist` or `Admin` role with `boms:review_queue:read` or `boms:review` permission.
- **Query Parameters:**
    - `page?: number` (Default: 1)
    - `limit?: number` (Default: 20)
    - `sortBy?: string` (Default: `submittedAt`) - e.g., `priority`, `customerName`, `orderId`.
    - `sortOrder?: 'asc' | 'desc'` (Default: `asc` for due dates/priority).
    - `priority?: 'High' | 'Medium' | 'Low'` - Filter by priority.
    - `complexity?: 'Simple' | 'Moderate' | 'Complex'` - Filter by complexity.
- **Success Response (200 OK):** `PaginatedBOMReviewList`
  ```typescript
  // PaginatedBOMReviewList (Specific to Part B - to be added to Data Models section)
  // Uses BOMReviewItem from Part A Data Models
  interface PaginatedBOMReviewList {
    items: BOMReviewItem[];
    totalItems: number;
    totalPages: number;
    currentPage: number;
    itemsPerPage: number;
  }

  // BOMReviewItem (Reference from Part A - Data Models)
  // interface BOMReviewItem {
  //   bomId: string;
  //   orderId: string;
  //   poNumber: string;
  //   customerName: string;
  //   totalParts: number;
  //   totalAssemblies: number;
  //   priority: 'High' | 'Medium' | 'Low';
  //   submittedAt: string; // ISO 8601 Datetime
  //   dueDate: string; // ISO 8601 Date
  //   complexity: 'Simple' | 'Moderate' | 'Complex';
  //   reviewStatus: BOMReviewStatus; // Enum from Part A
  // }
  ```
- **Error Responses:** `401 Unauthorized`, `403 Forbidden`.

#### Submit BOM Review Actions
- **Endpoint:** `POST /boms/{bomId}/review`
- **Description:** Allows a Procurement Specialist to submit their review actions for a BOM (e.g., approve, reject, request modifications). This endpoint encapsulates multiple actions that were previously separate PUTs for approve/reject/modify.
- **Authorization:** Requires `ProcurementSpecialist` or `Admin` role with `boms:review` permission.
- **Path Parameters:**
    - `bomId: string` (UUID) - The ID of the BOM being reviewed.
- **Request Body:** `BOMReviewActionPayload`
  ```typescript
  // BOMReviewActionPayload (Specific to Part B - to be added to Data Models section)
  // Uses BOMReviewStatus enum from Part A
  interface BOMReviewActionPayload {
    reviewStatus: BOMReviewStatus.APPROVED | BOMReviewStatus.REJECTED | BOMReviewStatus.REQUIRES_MODIFICATION;
    reviewNotes?: string;
    modifications?: any; // JSONB structure detailing proposed/required changes if status is REQUIRES_MODIFICATION
                         // Could be a list of { itemId: string, field: string, oldValue: any, newValue: any, comment: string }
  }
  ```
  ```json
  // Example: Approve
  { "reviewStatus": "Approved", "reviewNotes": "All parts verified and available." }
  // Example: Reject
  { "reviewStatus": "Rejected", "reviewNotes": "Critical component X is obsolete. Please find alternative." }
  // Example: Requires Modification
  { "reviewStatus": "RequiresModification", "reviewNotes": "See attached modification list.", "modifications": [ /* ... */ ] }
  ```
- **Success Response (200 OK):** `BOMReview_DB` (The updated BOM review record, or the BOMStructure with updated status. From Part A Data Models)
  ```json
  // Example Response (BOMReview_DB from Part A)
  {
    "reviewId": "review_uuid_123",
    "bomId": "bom_uuid_xyz",
    "reviewerId": "user_uuid_proc",
    "reviewStatus": "Approved",
    "reviewNotes": "All parts verified and available.",
    "reviewedAt": "2024-11-12T10:00:00Z",
    "approvedAt": "2024-11-12T10:00:00Z"
  }
  ```
- **Error Responses:** `400 Bad Request` (invalid payload, invalid status transition), `401 Unauthorized`, `403 Forbidden`, `404 Not Found` (BOM not found).

*(Note: The previous separate endpoints `PUT /boms/[bomId]/approve`, `PUT /boms/[bomId]/reject`, `PUT /boms/[bomId]/modify` from `05_Phase2` are consolidated into the `POST /boms/{bomId}/review` endpoint for a more unified review action interface. If separate endpoints are strictly required, they can be re-detailed.)*

---

## Data Models Specific to Part B

This section defines data models that are primarily used or extended within the scope of Order, BOM, Catalog, and Outsourcing management. Core data models are defined in Part A.

```typescript
// Order & Production Enums
enum OrderStatus {
  OrderCreated = 'OrderCreated',
  PartsSent = 'PartsSent',
  ReadyForPreQC = 'ReadyForPreQC',
  ReadyForProduction = 'ReadyForProduction',
  TestingComplete = 'TestingComplete',
  PackagingComplete = 'PackagingComplete',
  ReadyForFinalQC = 'ReadyForFinalQC',
  ReadyForShip = 'ReadyForShip',
  Shipped = 'Shipped',
}

enum SinkFamily {
  MDRD = 'MDRD',
  ENDOSCOPE = 'Endoscope',
  INSTROSINK = 'InstroSink',
}

```

---

## Error Handling Specific to Part B
// ... existing code ...

```

# Clicked Code Block to apply Changes From

// ... existing code ...
---

## Data Models Specific to Part B

This section defines data models that are primarily used or extended within the scope of Order, BOM, Catalog, and Outsourcing management. Core data models are defined in Part A.

```typescript
// Order & Production Enums
enum OrderStatus {
  OrderCreated = 'OrderCreated',
  PartsSent = 'PartsSent',
  ReadyForPreQC = 'ReadyForPreQC',
  ReadyForProduction = 'ReadyForProduction',
  TestingComplete = 'TestingComplete',
  PackagingComplete = 'PackagingComplete',
  ReadyForFinalQC = 'ReadyForFinalQC',
  ReadyForShip = 'ReadyForShip',
  Shipped = 'Shipped',
}

enum SinkFamily {
  MDRD = 'MDRD',
  ENDOSCOPE = 'Endoscope',
  INSTROSINK = 'InstroSink',
}

```

---

## Error Handling Specific to Part B
// ... existing code ...

```

# Clicked Code Block to apply Changes From

// ... existing code ...
---

## Data Models Specific to Part B

This section defines data models that are primarily used or extended within the scope of Order, BOM, Catalog, and Outsourcing management. Core data models are defined in Part A.

```typescript
// Order & Production Enums
enum OrderStatus {
  OrderCreated = 'OrderCreated',
  PartsSent = 'PartsSent',
  ReadyForPreQC = 'ReadyForPreQC',
  ReadyForProduction = 'ReadyForProduction',
  TestingComplete = 'TestingComplete',
  PackagingComplete = 'PackagingComplete',
  ReadyForFinalQC = 'ReadyForFinalQC',
  ReadyForShip = 'ReadyForShip',
  Shipped = 'Shipped',
}

enum SinkFamily {
  MDRD = 'MDRD',
  ENDOSCOPE = 'Endoscope',
  INSTROSINK = 'InstroSink',
}

```

---

## Error Handling Specific to Part B
// ... existing code ...

```

# Clicked Code Block to apply Changes From

// ... existing code ...
---

## Data Models Specific to Part B

This section defines data models that are primarily used or extended within the scope of Order, BOM, Catalog, and Outsourcing management. Core data models are defined in Part A.

```typescript
// Order & Production Enums
enum OrderStatus {
  OrderCreated = 'OrderCreated',
  PartsSent = 'PartsSent',
  ReadyForPreQC = 'ReadyForPreQC',
  ReadyForProduction = 'ReadyForProduction',
  TestingComplete = 'TestingComplete',
  PackagingComplete = 'PackagingComplete',
  ReadyForFinalQC = 'ReadyForFinalQC',
  ReadyForShip = 'ReadyForShip',
  Shipped = 'Shipped',
}

enum SinkFamily {
  MDRD = 'MDRD',
  ENDOSCOPE = 'Endoscope',
  INSTROSINK = 'InstroSink',
}

```

---

## Error Handling Specific to Part B
// ... existing code ...

```

# Clicked Code Block to apply Changes From

// ... existing code ...
---

## Data Models Specific to Part B

This section defines data models that are primarily used or extended within the scope of Order, BOM, Catalog, and Outsourcing management. Core data models are defined in Part A.

```typescript
// Order & Production Enums
enum OrderStatus {
  OrderCreated = 'OrderCreated',
  PartsSent = 'PartsSent',
  ReadyForPreQC = 'ReadyForPreQC',
  ReadyForProduction = 'ReadyForProduction',
  TestingComplete = 'TestingComplete',
  PackagingComplete = 'PackagingComplete',
  ReadyForFinalQC = 'ReadyForFinalQC',
  ReadyForShip = 'ReadyForShip',
  Shipped = 'Shipped',
}

enum SinkFamily {
  MDRD = 'MDRD',
  ENDOSCOPE = 'Endoscope',
  INSTROSINK = 'InstroSink',
}

```

---

## Error Handling Specific to Part B
// ... existing code ...

```

# Clicked Code Block to apply Changes From

// ... existing code ...
---

## Data Models Specific to Part B

This section defines data models that are primarily used or extended within the scope of Order, BOM, Catalog, and Outsourcing management. Core data models are defined in Part A.

```typescript
// Order & Production Enums
enum OrderStatus {
  OrderCreated = 'OrderCreated',
  PartsSent = 'PartsSent',
  ReadyForPreQC = 'ReadyForPreQC',
  ReadyForProduction = 'ReadyForProduction',
  TestingComplete = 'TestingComplete',
  PackagingComplete = 'PackagingComplete',
  ReadyForFinalQC = 'ReadyForFinalQC',
  ReadyForShip = 'ReadyForShip',
  Shipped = 'Shipped',
}

enum SinkFamily {
  MDRD = 'MDRD',
  ENDOSCOPE = 'Endoscope',
  INSTROSINK = 'InstroSink',
}

```

---

## Error Handling Specific to Part B
// ... existing code ...

```

# Clicked Code Block to apply Changes From

// ... existing code ...
---

## Data Models Specific to Part B

This section defines data models that are primarily used or extended within the scope of Order, BOM, Catalog, and Outsourcing management. Core data models are defined in Part A.

```typescript
// Order & Production Enums
enum OrderStatus {
  OrderCreated = 'OrderCreated',
  PartsSent = 'PartsSent',
  ReadyForPreQC = 'ReadyForPreQC',
  ReadyForProduction = 'ReadyForProduction',
  TestingComplete = 'TestingComplete',
  PackagingComplete = 'PackagingComplete',
  ReadyForFinalQC = 'ReadyForFinalQC',
  ReadyForShip = 'ReadyForShip',
  Shipped = 'Shipped',
}

enum SinkFamily {
  MDRD = 'MDRD',
  ENDOSCOPE = 'Endoscope',
  INSTROSINK = 'InstroSink',
}

```

---

## Error Handling Specific to Part B
// ... existing code ...

```

# Clicked Code Block to apply Changes From

// ... existing code ...
---

## Data Models Specific to Part B

This section defines data models that are primarily used or extended within the scope of Order, BOM, Catalog, and Outsourcing management. Core data models are defined in Part A.

```typescript
// Order & Production Enums
enum OrderStatus {
  OrderCreated = 'OrderCreated',
  PartsSent = 'PartsSent',
  ReadyForPreQC = 'ReadyForPreQC',
  ReadyForProduction = 'ReadyForProduction',
  TestingComplete = 'TestingComplete',
  PackagingComplete = 'PackagingComplete',
  ReadyForFinalQC = 'ReadyForFinalQC',
  ReadyForShip = 'ReadyForShip',
  Shipped = 'Shipped',
}

enum SinkFamily {
  MDRD = 'MDRD',
  ENDOSCOPE = 'Endoscope',
  INSTROSINK = 'InstroSink',
}

```

---

## Error Handling Specific to Part B
// ... existing code ...

```

# Clicked Code Block to apply Changes From

// ... existing code ...
---

## Data Models Specific to Part B

This section defines data models that are primarily used or extended within the scope of Order, BOM, Catalog, and Outsourcing management. Core data models are defined in Part A.

```typescript
// Order & Production Enums
enum OrderStatus {
  OrderCreated = 'OrderCreated',
  PartsSent = 'PartsSent',
  ReadyForPreQC = 'ReadyForPreQC',
  ReadyForProduction = 'ReadyForProduction',
  TestingComplete = 'TestingComplete',
  PackagingComplete = 'PackagingComplete',
  ReadyForFinalQC = 'ReadyForFinalQC',
  ReadyForShip = 'ReadyForShip',
  Shipped = 'Shipped',
}

enum SinkFamily {
  MDRD = 'MDRD',
  ENDOSCOPE = 'Endoscope',
  INSTROSINK = 'InstroSink',
}

```

---

## Error Handling Specific to Part B
// ... existing code ...

```

# Clicked Code Block to apply Changes From

// ... existing code ...
---

## Data Models Specific to Part B

This section defines data models that are primarily used or extended within the scope of Order, BOM, Catalog, and Outsourcing management. Core data models are defined in Part A.

```typescript
// Order & Production Enums
enum OrderStatus {
  OrderCreated = 'OrderCreated',
  PartsSent = 'PartsSent',
  ReadyForPreQC = 'ReadyForPreQC',
  ReadyForProduction = 'ReadyForProduction',
  TestingComplete = 'TestingComplete',
  PackagingComplete = 'PackagingComplete',
  ReadyForFinalQC = 'ReadyForFinalQC',
  ReadyForShip = 'ReadyForShip',
  Shipped = 'Shipped',
}

enum SinkFamily {
  MDRD = 'MDRD',
  ENDOSCOPE = 'Endoscope',
  INSTROSINK = 'InstroSink',
}

```

---

## Error Handling Specific to Part B
// ... existing code ...

```

# Clicked Code Block to apply Changes From

// ... existing code ...
---

## Data Models Specific to Part B

This section defines data models that are primarily used or extended within the scope of Order, BOM, Catalog, and Outsourcing management. Core data models are defined in Part A.

```typescript
// Order & Production Enums
enum OrderStatus {
  OrderCreated = 'OrderCreated',
  PartsSent = 'PartsSent',
  ReadyForPreQC = 'ReadyForPreQC',
  ReadyForProduction = 'ReadyForProduction',
  TestingComplete = 'TestingComplete',
  PackagingComplete = 'PackagingComplete',
  ReadyForFinalQC = 'ReadyForFinalQC',
  ReadyForShip = 'ReadyForShip',
  Shipped = 'Shipped',
}

enum SinkFamily {
  MDRD = 'MDRD',
  ENDOSCOPE = 'Endoscope',
  INSTROSINK = 'InstroSink',
}

```

---

## Error Handling Specific to Part B
// ... existing code ...

```

# Clicked Code Block to apply Changes From

// ... existing code ...
---

## Data Models Specific to Part B

This section defines data models that are primarily used or extended within the scope of Order, BOM, Catalog, and Outsourcing management. Core data models are defined in Part A.

```typescript
// Order & Production Enums
enum OrderStatus {
  OrderCreated = 'OrderCreated',
  PartsSent = 'PartsSent',
  ReadyForPreQC = 'ReadyForPreQC',
  ReadyForProduction = 'ReadyForProduction',
  TestingComplete = 'TestingComplete',
  PackagingComplete = 'PackagingComplete',
  ReadyForFinalQC = 'ReadyForFinalQC',
  ReadyForShip = 'ReadyForShip',
  Shipped = 'Shipped',
}

enum SinkFamily {
  MDRD = 'MDRD',
  ENDOSCOPE = 'Endoscope',
  INSTROSINK = 'InstroSink',
}

```

---

## Error Handling Specific to Part B
// ... existing code ...

```

# Clicked Code Block to apply Changes From

// ... existing code ...
---

## Data Models Specific to Part B

This section defines data models that are primarily used or extended within the scope of Order, BOM, Catalog, and Outsourcing management. Core data models are defined in Part A.

```typescript
// Order & Production Enums
enum OrderStatus {
  OrderCreated = 'OrderCreated',
  PartsSent = 'PartsSent',
  ReadyForPreQC = 'ReadyForPreQC',
  ReadyForProduction = 'ReadyForProduction',
  TestingComplete = 'TestingComplete',
  PackagingComplete = 'PackagingComplete',
  ReadyForFinalQC = 'ReadyForFinalQC',
  ReadyForShip = 'ReadyForShip',
  Shipped = 'Shipped',
}

enum SinkFamily {
  MDRD = 'MDRD',
  ENDOSCOPE = 'Endoscope',
  INSTROSINK = 'InstroSink',
}

```

---

## Error Handling Specific to Part B
// ... existing code ...

```

# Clicked Code Block to apply Changes From

// ... existing code ...
---

## Data Models Specific to Part B

This section defines data models that are primarily used or extended within the scope of Order, BOM, Catalog, and Outsourcing management. Core data models are defined in Part A.

```typescript
// Order & Production Enums
enum OrderStatus {
  OrderCreated = 'OrderCreated',
  PartsSent = 'PartsSent',
  ReadyForPreQC = 'ReadyForPreQC',
  ReadyForProduction = 'ReadyForProduction',
  TestingComplete = 'TestingComplete',
  PackagingComplete = 'PackagingComplete',
  ReadyForFinalQC = 'ReadyForFinalQC',
  ReadyForShip = 'ReadyForShip',
  Shipped = 'Shipped',
}

enum SinkFamily {
  MDRD = 'MDRD',
  ENDOSCOPE = 'Endoscope',
  INSTROSINK = 'InstroSink',
}

```

---

## Error Handling Specific to Part B
// ... existing code ...

```

# Clicked Code Block to apply Changes From

// ... existing code ...
---

## Data Models Specific to Part B

This section defines data models that are primarily used or extended within the scope of Order, BOM, Catalog, and Outsourcing management. Core data models are defined in Part A.

```typescript
// Order & Production Enums
enum OrderStatus {
  OrderCreated = 'OrderCreated',
  PartsSent = 'PartsSent',
  ReadyForPreQC = 'ReadyForPreQC',
  ReadyForProduction = 'ReadyForProduction',
  TestingComplete = 'TestingComplete',
  PackagingComplete = 'PackagingComplete',
  ReadyForFinalQC = 'ReadyForFinalQC',
  ReadyForShip = 'ReadyForShip',
  Shipped = 'Shipped',
}

enum SinkFamily {
  MDRD = 'MDRD',
  ENDOSCOPE = 'Endoscope',
  INSTROSINK = 'InstroSink',
}

```

---

## Error Handling Specific to Part B
// ... existing code ...

```

# Clicked Code Block to apply Changes From

// ... existing code ...
---

## Data Models Specific to Part B

This section defines data models that are primarily used or extended within the scope of Order, BOM, Catalog, and Outsourcing management. Core data models are defined in Part A.

```typescript
// Order & Production Enums
enum OrderStatus {
  OrderCreated = 'OrderCreated',
  PartsSent = 'PartsSent',
  ReadyForPreQC = 'ReadyForPreQC',
  ReadyForProduction = 'ReadyForProduction',
  TestingComplete = 'TestingComplete',
  PackagingComplete = 'PackagingComplete',
  ReadyForFinalQC = 'ReadyForFinalQC',
  ReadyForShip = 'ReadyForShip',
  Shipped = 'Shipped',
}

enum SinkFamily {
  MDRD = 'MDRD',
  ENDOSCOPE = 'Endoscope',
  INSTROSINK = 'InstroSink',
}

```

---

## Error Handling Specific to Part B
// ... existing code ...

```

# Clicked Code Block to apply Changes From

// ... existing code ...
---

## Data Models Specific to Part B

This section defines data models that are primarily used or extended within the scope of Order, BOM, Catalog, and Outsourcing management. Core data models are defined in Part A.

```typescript
// Order & Production Enums
enum OrderStatus {
  OrderCreated = 'OrderCreated',
  PartsSent = 'PartsSent',
  ReadyForPreQC = 'ReadyForPreQC',
  ReadyForProduction = 'ReadyForProduction',
  TestingComplete = 'TestingComplete',
  PackagingComplete = 'PackagingComplete',
  ReadyForFinalQC = 'ReadyForFinalQC',
  ReadyForShip = 'ReadyForShip',
  Shipped = 'Shipped',
}

enum SinkFamily {
  MDRD = 'MDRD',
  ENDOSCOPE = 'Endoscope',
  INSTROSINK = 'InstroSink',
}

```

---

## Error Handling Specific to Part B
// ... existing code ...

```

# Clicked Code Block to apply Changes From

// ... existing code ...
---

## Data Models Specific to Part B

This section defines data models that are primarily used or extended within the scope of Order, BOM, Catalog, and Outsourcing management. Core data models are defined in Part A.

```typescript
// Order & Production Enums
enum OrderStatus {
  OrderCreated = 'OrderCreated',
  PartsSent = 'PartsSent',
  ReadyForPreQC = 'ReadyForPreQC',
  ReadyForProduction = 'ReadyForProduction',
  TestingComplete = 'TestingComplete',
  PackagingComplete = 'PackagingComplete',
  ReadyForFinalQC = 'ReadyForFinalQC',
  ReadyForShip = 'ReadyForShip',
  Shipped = 'Shipped',
}

enum SinkFamily {
  MDRD = 'MDRD',
  ENDOSCOPE = 'Endoscope',
  INSTROSINK = 'InstroSink',
}

```

---

## Error Handling Specific to Part B
// ... existing code ...

```

# Clicked Code Block to apply Changes From

// ... existing code ...
---

## Data Models Specific to Part B

This section defines data models that are primarily used or extended within the scope of Order, BOM, Catalog, and Outsourcing management. Core data models are defined in Part A.

```typescript
// Order & Production Enums
enum OrderStatus {
  OrderCreated = 'OrderCreated',
  PartsSent = 'PartsSent',
  ReadyForPreQC = 'ReadyForPreQC',
  ReadyForProduction = 'ReadyForProduction',
  TestingComplete = 'TestingComplete',
  PackagingComplete = 'PackagingComplete',
  ReadyForFinalQC = 'ReadyForFinalQC',
  ReadyForShip = 'ReadyForShip',
  Shipped = 'Shipped',
}

enum SinkFamily {
  MDRD = 'MDRD',
  ENDOSCOPE = 'Endoscope',
  INSTROSINK = 'InstroSink',
}

```

---

## Error Handling Specific to Part B
// ... existing code ...

```

# Clicked Code Block to apply Changes From

// ... existing code ...
---

## Data Models Specific to Part B

This section defines data models that are primarily used or extended within the scope of Order, BOM, Catalog, and Outsourcing management. Core data models are defined in Part A.

```typescript
// Order & Production Enums
enum OrderStatus {
  OrderCreated = 'OrderCreated',
  PartsSent = 'PartsSent',
  ReadyForPreQC = 'ReadyForPreQC',
  ReadyForProduction = 'ReadyForProduction',
  TestingComplete = 'TestingComplete',
  PackagingComplete = 'PackagingComplete',
  ReadyForFinalQC = 'ReadyForFinalQC',
  ReadyForShip = 'ReadyForShip',
  Shipped = 'Shipped',
}

enum SinkFamily {
  MDRD = 'MDRD',
  ENDOSCOPE = 'Endoscope',
  INSTROSINK = 'InstroSink',
}

```

---

## Error Handling Specific to Part B
// ... existing code ...

```

# Clicked Code Block to apply Changes From

// ... existing code ...
---

## Data Models Specific to Part B

This section defines data models that are primarily used or extended within the scope of Order, BOM, Catalog, and Outsourcing management. Core data models are defined in Part A.

```typescript
// Order & Production Enums
enum OrderStatus {
  OrderCreated = 'OrderCreated',
  PartsSent = 'PartsSent',
  ReadyForPreQC = 'ReadyForPreQC',
  ReadyForProduction = 'ReadyForProduction',
  TestingComplete = 'TestingComplete',
  PackagingComplete = 'PackagingComplete',
  ReadyForFinalQC = 'ReadyForFinalQC',
  ReadyForShip = 'ReadyForShip',
  Shipped = 'Shipped',
}

enum SinkFamily {
  MDRD = 'MDRD',
  ENDOSCOPE = 'Endoscope',
  INSTROSINK = 'InstroSink',
}

```

---

## Error Handling Specific to Part B
// ... existing code ...

```

# Clicked Code Block to apply Changes From

// ... existing code ...
---

## Data Models Specific to Part B

This section defines data models that are primarily used or extended within the scope of Order, BOM, Catalog, and Outsourcing management. Core data models are defined in Part A.

```typescript
// Order & Production Enums
enum OrderStatus {
  OrderCreated = 'OrderCreated',
  PartsSent = 'PartsSent',
  ReadyForPreQC = 'ReadyForPreQC',
  ReadyForProduction = 'ReadyForProduction',
  TestingComplete = 'TestingComplete',
  PackagingComplete = 'PackagingComplete',
  ReadyForFinalQC = 'ReadyForFinalQC',
  ReadyForShip = 'ReadyForShip',
  Shipped = 'Shipped',
}

enum SinkFamily {
  MDRD = 'MDRD',
  ENDOSCOPE = 'Endoscope',
  INSTROSINK = 'InstroSink',
}

```

---

## Error Handling Specific to Part B
// ... existing code ...

```

# Clicked Code Block to apply Changes From

// ... existing code ...
---

## Data Models Specific to Part B

This section defines data models that are primarily used or extended within the scope of Order, BOM, Catalog, and Outsourcing management. Core data models are defined in Part A.

```typescript
// Order & Production Enums
enum OrderStatus {
  OrderCreated = 'OrderCreated',
  PartsSent = 'PartsSent',
  ReadyForPreQC = 'ReadyForPreQC',
  ReadyForProduction = 'ReadyForProduction',
  TestingComplete = 'TestingComplete',
  PackagingComplete = 'PackagingComplete',
  ReadyForFinalQC = 'ReadyForFinalQC',
  ReadyForShip = 'ReadyForShip',
  Shipped = 'Shipped',
}

enum SinkFamily {
  MDRD = 'MDRD',
  ENDOSCOPE = 'Endoscope',
  INSTROSINK = 'InstroSink',
}

```

---

## Error Handling Specific to Part B
// ... existing code ...

```

# Clicked Code Block to apply Changes From

// ... existing code ...
---

## Data Models Specific to Part B

This section defines data models that are primarily used or extended within the scope of Order, BOM, Catalog, and Outsourcing management. Core data models are defined in Part A.

```typescript
// Order & Production Enums
enum OrderStatus {
  OrderCreated = 'OrderCreated',
  PartsSent = 'PartsSent',
  ReadyForPreQC = 'ReadyForPreQC',
  ReadyForProduction = 'ReadyForProduction',
  TestingComplete = 'TestingComplete',
  PackagingComplete = 'PackagingComplete',
  ReadyForFinalQC = 'ReadyForFinalQC',
  ReadyForShip = 'ReadyForShip',
  Shipped = 'Shipped',
}

enum SinkFamily {
  MDRD = 'MDRD',
  ENDOSCOPE = 'Endoscope',
  INSTROSINK = 'InstroSink',
}

```

---

## Error Handling Specific to Part B
// ... existing code ...

```

# Clicked Code Block to apply Changes From

// ... existing code ...
---

## Data Models Specific to Part B

This section defines data models that are primarily used or extended within the scope of Order, BOM, Catalog, and Outsourcing management. Core data models are defined in Part A.

```typescript
// Order & Production Enums
enum OrderStatus {
  OrderCreated = 'OrderCreated',
  PartsSent = 'PartsSent',
  ReadyForPreQC = 'ReadyForPreQC',
  ReadyForProduction = 'ReadyForProduction',
  TestingComplete = 'TestingComplete',
  PackagingComplete = 'PackagingComplete',
  ReadyForFinalQC = 'ReadyForFinalQC',
  ReadyForShip = 'ReadyForShip',
  Shipped = 'Shipped',
}

enum SinkFamily {
  MDRD = 'MDRD',
  ENDOSCOPE = 'Endoscope',
  INSTROSINK = 'InstroSink',
}

```

---

## Error Handling Specific to Part B
// ... existing code ...

```

# Clicked Code Block to apply Changes From

// ... existing code ...
---

## Data Models Specific to Part B

This section defines data models that are primarily used or extended within the scope of Order, BOM, Catalog, and Outsourcing management. Core data models are defined in Part A.

```typescript
// Order & Production Enums
enum OrderStatus {
  OrderCreated = 'OrderCreated',
  PartsSent = 'PartsSent',
  ReadyForPreQC = 'ReadyForPreQC',
  ReadyForProduction = 'ReadyForProduction',
  TestingComplete = 'TestingComplete',
  PackagingComplete = 'PackagingComplete',
  ReadyForFinalQC = 'ReadyForFinalQC',
  ReadyForShip = 'ReadyForShip',
  Shipped = 'Shipped',
}

enum SinkFamily {
  MDRD = 'MDRD',
  ENDOSCOPE = 'Endoscope',
  INSTROSINK = 'InstroSink',
}

```

---

## Error Handling Specific to Part B
// ... existing code ...

```

# Clicked Code Block to apply Changes From

// ... existing code ...
---

## Data Models Specific to Part B

This section defines data models that are primarily used or extended within the scope of Order, BOM, Catalog, and Outsourcing management. Core data models are defined in Part A.

```typescript
// Order & Production Enums
enum OrderStatus {
  OrderCreated = 'OrderCreated',
  PartsSent = 'PartsSent',
  ReadyForPreQC = 'ReadyForPreQC',
  ReadyForProduction = 'ReadyForProduction',
  TestingComplete = 'TestingComplete',
  PackagingComplete = 'PackagingComplete',
  ReadyForFinalQC = 'ReadyForFinalQC',
  ReadyForShip = 'ReadyForShip',
  Shipped = 'Shipped',
}

enum SinkFamily {
  MDRD = 'MDRD',
  ENDOSCOPE = 'Endoscope',
  INSTROSINK = 'InstroSink',
}

```

---

## Error Handling Specific to Part B
// ... existing code ...

```

# Clicked Code Block to apply Changes From

// ... existing code ...
---

## Data Models Specific to Part B

This section defines data models that are primarily used or extended within the scope of Order, BOM, Catalog, and Outsourcing management. Core data models are defined in Part A.

```typescript
// Order & Production Enums
enum OrderStatus {
  OrderCreated = 'OrderCreated',
  PartsSent = 'PartsSent',
  ReadyForPreQC = 'ReadyForPreQC',
  ReadyForProduction = 'ReadyForProduction',
  TestingComplete = 'TestingComplete',
  PackagingComplete = 'PackagingComplete',
  ReadyForFinalQC = 'ReadyForFinalQC',
  ReadyForShip = 'ReadyForShip',
  Shipped = 'Shipped',
}

enum SinkFamily {
  MDRD = 'MDRD',
  ENDOSCOPE = 'Endoscope',
  INSTROSINK = 'InstroSink',
}

```

---

## Error Handling Specific to Part B
// ... existing code ...

```

# Clicked Code Block to apply Changes From

// ... existing code ...
---

## Data Models Specific to Part B

This section defines data models that are primarily used or extended within the scope of Order, BOM, Catalog, and Outsourcing management. Core data models are defined in Part A.

```typescript
// Order & Production Enums
enum OrderStatus {
  OrderCreated = 'OrderCreated',
  PartsSent = 'PartsSent',
  ReadyForPreQC = 'ReadyForPreQC',
  ReadyForProduction = 'ReadyForProduction',
  TestingComplete = 'TestingComplete',
  PackagingComplete = 'PackagingComplete',
  ReadyForFinalQC = 'ReadyForFinalQC',
  ReadyForShip = 'ReadyForShip',
  Shipped = 'Shipped',
}

enum SinkFamily {
  MDRD = 'MDRD',
  ENDOSCOPE = 'Endoscope',
  INSTROSINK = 'InstroSink',
}

```

---

## Error Handling Specific to Part B
// ... existing code ...

```

# Clicked Code Block to apply Changes From

// ... existing code ...
---

## Data Models Specific to Part B

This section defines data models that are primarily used or extended within the scope of Order, BOM, Catalog, and Outsourcing management. Core data models are defined in Part A.

```typescript
// Order & Production Enums
enum OrderStatus {
  OrderCreated = 'OrderCreated',
  PartsSent = 'PartsSent',
  ReadyForPreQC = 'ReadyForPreQC',
  ReadyForProduction = 'ReadyForProduction',
  TestingComplete = 'TestingComplete',
  PackagingComplete = 'PackagingComplete',
  ReadyForFinalQC = 'ReadyForFinalQC',
  ReadyForShip = 'ReadyForShip',
  Shipped = 'Shipped',
}

enum SinkFamily {
  MDRD = 'MDRD',
  ENDOSCOPE = 'Endoscope',
  INSTROSINK = 'InstroSink',
}

```

---

## Error Handling Specific to Part B
// ... existing code ...

```

# Clicked Code Block to apply Changes From

// ... existing code ...
---

## Data Models Specific to Part B

This section defines data models that are primarily used or extended within the scope of Order, BOM, Catalog, and Outsourcing management. Core data models are defined in Part A.

```typescript
// Order & Production Enums
enum OrderStatus {
  OrderCreated = 'OrderCreated',
  PartsSent = 'PartsSent',
  ReadyForPreQC = 'ReadyForPreQC',
  ReadyForProduction = 'ReadyForProduction',
  TestingComplete = 'TestingComplete',
  PackagingComplete = 'PackagingComplete',
  ReadyForFinalQC = 'ReadyForFinalQC',
  ReadyForShip = 'ReadyForShip',
  Shipped = 'Shipped',
}

enum SinkFamily {
  MDRD = 'MDRD',
  ENDOSCOPE = 'Endoscope',
  INSTROSINK = 'InstroSink',
}

```

---

## Error Handling Specific to Part B
// ... existing code ...

```

# Clicked Code Block to apply Changes From

// ... existing code ...
---

## Data Models Specific to Part B

This section defines data models that are primarily used or extended within the scope of Order, BOM, Catalog, and Outsourcing management. Core data models are defined in Part A.

```typescript
// Order & Production Enums
enum OrderStatus {
  OrderCreated = 'OrderCreated',
  PartsSent = 'PartsSent',
  ReadyForPreQC = 'ReadyForPreQC',
  ReadyForProduction = 'ReadyForProduction',
  TestingComplete = 'TestingComplete',
  PackagingComplete = 'PackagingComplete',
  ReadyForFinalQC = 'ReadyForFinalQC',
  ReadyForShip = 'ReadyForShip',
  Shipped = 'Shipped',
}

enum SinkFamily {
  MDRD = 'MDRD',
  ENDOSCOPE = 'Endoscope',
  INSTROSINK = 'InstroSink',
}

```

---

## Error Handling Specific to Part B
// ... existing code ...

```

# Clicked Code Block to apply Changes From

// ... existing code ...
---

## Data Models Specific to Part B

This section defines data models that are primarily used or extended within the scope of Order, BOM, Catalog, and Outsourcing management. Core data models are defined in Part A.

```typescript
// Order & Production Enums
enum OrderStatus {
  OrderCreated = 'OrderCreated',
  PartsSent = 'PartsSent',
  ReadyForPreQC = 'ReadyForPreQC',
  ReadyForProduction = 'ReadyForProduction',
  TestingComplete = 'TestingComplete',
  PackagingComplete = 'PackagingComplete',
  ReadyForFinalQC = 'ReadyForFinalQC',
  ReadyForShip = 'ReadyForShip',
  Shipped = 'Shipped',
}

enum SinkFamily {
  MDRD = 'MDRD',
  ENDOSCOPE = 'Endoscope',
  INSTROSINK = 'InstroSink',
}

```

---

## Error Handling Specific to Part B
// ... existing code ...

```

# Clicked Code Block to apply Changes From

// ... existing code ...
---

## Data Models Specific to Part B

This section defines data models that are primarily used or extended within the scope of Order, BOM, Catalog, and Outsourcing management. Core data models are defined in Part A.

```typescript
// Order & Production Enums
enum OrderStatus {
  OrderCreated = 'OrderCreated',
  PartsSent = 'PartsSent',
  ReadyForPreQC = 'ReadyForPreQC',
  ReadyForProduction = 'ReadyForProduction',
  TestingComplete = 'TestingComplete',
  PackagingComplete = 'PackagingComplete',
  ReadyForFinalQC = 'ReadyForFinalQC',
  ReadyForShip = 'ReadyForShip',
  Shipped = 'Shipped',
}

enum SinkFamily {
  MDRD = 'MDRD',
  ENDOSCOPE = 'Endoscope',
  INSTROSINK = 'InstroSink',
}

```

---

## Error Handling Specific to Part B
// ... existing code ...

```

# Clicked Code Block to apply Changes From

// ... existing code ...
---

## Data Models Specific to Part B

This section defines data models that are primarily used or extended within the scope of Order, BOM, Catalog, and Outsourcing management. Core data models are defined in Part A.

```typescript
// Order & Production Enums
enum OrderStatus {
  OrderCreated = 'OrderCreated',
  PartsSent = 'PartsSent',
  ReadyForPreQC = 'ReadyForPreQC',
  ReadyForProduction = 'ReadyForProduction',
  TestingComplete = 'TestingComplete',
  PackagingComplete = 'PackagingComplete',
  ReadyForFinalQC = 'ReadyForFinalQC',
  ReadyForShip = 'ReadyForShip',
  Shipped = 'Shipped',
}

enum SinkFamily {
  MDRD = 'MDRD',
  ENDOSCOPE = 'Endoscope',
  INSTROSINK = 'InstroSink',
}

```

---

## Error Handling Specific to Part B
// ... existing code ...

```

# Clicked Code Block to apply Changes From

// ... existing code ...
---

## Data Models Specific to Part B

This section defines data models that are primarily used or extended within the scope of Order, BOM, Catalog, and Outsourcing management. Core data models are defined in Part A.

```typescript
// Order & Production Enums
enum OrderStatus {
  OrderCreated = 'OrderCreated',
  PartsSent = 'PartsSent',
  ReadyForPreQC = 'ReadyForPreQC',
  ReadyForProduction = 'ReadyForProduction',
  TestingComplete = 'TestingComplete',
  PackagingComplete = 'PackagingComplete',
  ReadyForFinalQC = 'ReadyForFinalQC',
  ReadyForShip = 'ReadyForShip',
  Shipped = 'Shipped',
}

enum SinkFamily {
  MDRD = 'MDRD',
  ENDOSCOPE = 'Endoscope',
  INSTROSINK = 'InstroSink',
}

```

---

## Error Handling Specific to Part B
// ... existing code ...

```

# Clicked Code Block to apply Changes From

// ... existing code ...
---

## Data Models Specific to Part B

This section defines data models that are primarily used or extended within the scope of Order, BOM, Catalog, and Outsourcing management. Core data models are defined in Part A.

```typescript
// Order & Production Enums
enum OrderStatus {
  OrderCreated = 'OrderCreated',
  PartsSent = 'PartsSent',
  ReadyForPreQC = 'ReadyForPreQC',
  ReadyForProduction = 'ReadyForProduction',
  TestingComplete = 'TestingComplete',
  PackagingComplete = 'PackagingComplete',
  ReadyForFinalQC = 'ReadyForFinalQC',
  ReadyForShip = 'ReadyForShip',
  Shipped = 'Shipped',
}

enum SinkFamily {
  MDRD = 'MDRD',
  ENDOSCOPE = 'Endoscope',
  INSTROSINK = 'InstroSink',
}

```

---

## Error Handling Specific to Part B
// ... existing code ...

```

# Clicked Code Block to apply Changes From

// ... existing code ...
---

## Data Models Specific to Part B

This section defines data models that are primarily used or extended within the scope of Order, BOM, Catalog, and Outsourcing management. Core data models are defined in Part A.

```typescript
// Order & Production Enums
enum OrderStatus {
  OrderCreated = 'OrderCreated',
  PartsSent = 'PartsSent',
  ReadyForPreQC = 'ReadyForPreQC',
  ReadyForProduction = 'ReadyForProduction',
  TestingComplete = 'TestingComplete',
  PackagingComplete = 'PackagingComplete',
  ReadyForFinalQC = 'ReadyForFinalQC',
  ReadyForShip = 'ReadyForShip',
  Shipped = 'Shipped',
}

enum SinkFamily {
  MDRD = 'MDRD',
  ENDOSCOPE = 'Endoscope',
  INSTROSINK = 'InstroSink',
}

```

---

## Error Handling Specific to Part B
// ... existing code ...

```

# Clicked Code Block to apply Changes From

// ... existing code ...
---

## Data Models Specific to Part B

This section defines data models that are primarily used or extended within the scope of Order, BOM, Catalog, and Outsourcing management. Core data models are defined in Part A.

```typescript
// Order & Production Enums
enum OrderStatus {
  OrderCreated = 'OrderCreated',
  PartsSent = 'PartsSent',
  ReadyForPreQC = 'ReadyForPreQC',
  ReadyForProduction = 'ReadyForProduction',
  TestingComplete = 'TestingComplete',
  PackagingComplete = 'PackagingComplete',
  ReadyForFinalQC = 'ReadyForFinalQC',
  ReadyForShip = 'ReadyForShip',
  Shipped = 'Shipped',
}

enum SinkFamily {
  MDRD = 'MDRD',
  ENDOSCOPE = 'Endoscope',
  INSTROSINK = 'InstroSink',
}

```

---

## Error Handling Specific to Part B
// ... existing code ...

```

# Clicked Code Block to apply Changes From

// ... existing code ...
---

## Data Models Specific to Part B

This section defines data models that are primarily used or extended within the scope of Order, BOM, Catalog, and Outsourcing management. Core data models are defined in Part A.

```typescript
// Order & Production Enums
enum OrderStatus {
  OrderCreated = 'OrderCreated',
  PartsSent = 'PartsSent',
  ReadyForPreQC = 'ReadyForPreQC',
  ReadyForProduction = 'ReadyForProduction',
  TestingComplete = 'TestingComplete',
  PackagingComplete = 'PackagingComplete',
  ReadyForFinalQC = 'ReadyForFinalQC',
  ReadyForShip = 'ReadyForShip',
  Shipped = 'Shipped',
}

enum SinkFamily {
  MDRD = 'MDRD',
  ENDOSCOPE = 'Endoscope',
  INSTROSINK = 'InstroSink',
}

```

---

## Error Handling Specific to Part B
// ... existing code ...

```

# Clicked Code Block to apply Changes From

// ... existing code ...
---

## Data Models Specific to Part B

This section defines data models that are primarily used or extended within the scope of Order, BOM, Catalog, and Outsourcing management. Core data models are defined in Part A.

```typescript
// Order & Production Enums
enum OrderStatus {
  OrderCreated = 'OrderCreated',
  PartsSent = 'PartsSent',
  ReadyForPreQC = 'ReadyForPreQC',
  ReadyForProduction = 'ReadyForProduction',
  TestingComplete = 'TestingComplete',
  PackagingComplete = 'PackagingComplete',
  ReadyForFinalQC = 'ReadyForFinalQC',
  ReadyForShip = 'ReadyForShip',
  Shipped = 'Shipped',
}

enum SinkFamily {
  MDRD = 'MDRD',
  ENDOSCOPE = 'Endoscope',
  INSTROSINK = 'InstroSink',
}

```

---

## Error Handling Specific to Part B
// ... existing code ...

```

# Clicked Code Block to apply Changes From

// ... existing code ...
---

## Data Models Specific to Part B

This section defines data models that are primarily used or extended within the scope of Order, BOM, Catalog, and Outsourcing management. Core data models are defined in Part A.

```typescript
// Order & Production Enums
enum OrderStatus {
  OrderCreated = 'OrderCreated',
  PartsSent = 'PartsSent',
  ReadyForPreQC = 'ReadyForPreQC',
  ReadyForProduction = 'ReadyForProduction',
  TestingComplete = 'TestingComplete',
  PackagingComplete = 'PackagingComplete',
  ReadyForFinalQC = 'ReadyForFinalQC',
  ReadyForShip = 'ReadyForShip',
  Shipped = 'Shipped',
}

enum SinkFamily {
  MDRD = 'MDRD',
  ENDOSCOPE = 'Endoscope',
  INSTROSINK = 'InstroSink',
}

```

---

## Error Handling Specific to Part B
// ... existing code ...

```

# Clicked Code Block to apply Changes From

// ... existing code ...
---

## Data Models Specific to Part B

This section defines data models that are primarily used or extended within the scope of Order, BOM, Catalog, and Outsourcing management. Core data models are defined in Part A.

```typescript
// Order & Production Enums
enum OrderStatus {
  OrderCreated = 'OrderCreated',
  PartsSent = 'PartsSent',
  ReadyForPreQC = 'ReadyForPreQC',
  ReadyForProduction = 'ReadyForProduction',
  TestingComplete = 'TestingComplete',
  PackagingComplete = 'PackagingComplete',
  ReadyForFinalQC = 'ReadyForFinalQC',
  ReadyForShip = 'ReadyForShip',
  Shipped = 'Shipped',
}

enum SinkFamily {
  MDRD = 'MDRD',
  ENDOSCOPE = 'Endoscope',
  INSTROSINK = 'InstroSink',
}

```

---

## Error Handling Specific to Part B
// ... existing code ...

```

# Clicked Code Block to apply Changes From

// ... existing code ...
---

## Data Models Specific to Part B

This section defines data models that are primarily used or extended within the scope of Order, BOM, Catalog, and Outsourcing management. Core data models are defined in Part A.

```typescript
// Order & Production Enums
enum OrderStatus {
  OrderCreated = 'OrderCreated',
  PartsSent = 'PartsSent',
  ReadyForPreQC = 'ReadyForPreQC',
  ReadyForProduction = 'ReadyForProduction',
  TestingComplete = 'TestingComplete',
  PackagingComplete = 'PackagingComplete',
  ReadyForFinalQC = 'ReadyForFinalQC',
  ReadyForShip = 'ReadyForShip',
  Shipped = 'Shipped',
}

enum SinkFamily {
  MDRD = 'MDRD',
  ENDOSCOPE = 'Endoscope',
  INSTROSINK = 'InstroSink',
}

```

---

## Error Handling Specific to Part B
// ... existing code ...

```

# Clicked Code Block to apply Changes From

// ... existing code ...
---

## Data Models Specific to Part B

This section defines data models that are primarily used or extended within the scope of Order, BOM, Catalog, and Outsourcing management. Core data models are defined in Part A.

```typescript
// Order & Production Enums
enum OrderStatus {
  OrderCreated = 'OrderCreated',
  PartsSent = 'PartsSent',
  ReadyForPreQC = 'ReadyForPreQC',
  ReadyForProduction = 'ReadyForProduction',
  TestingComplete = 'TestingComplete',
  PackagingComplete = 'PackagingComplete',
  ReadyForFinalQC = 'ReadyForFinalQC',
  ReadyForShip = 'ReadyForShip',
  Shipped = 'Shipped',
}

enum SinkFamily {
  MDRD = 'MDRD',
  ENDOSCOPE = 'Endoscope',
  INSTROSINK = 'InstroSink',
}

```

---

## Error Handling Specific to Part B
// ... existing code ...

```

# Clicked Code Block to apply Changes From

// ... existing code ...
---

## Data Models Specific to Part B

This section defines data models that are primarily used or extended within the scope of Order, BOM, Catalog, and Outsourcing management. Core data models are defined in Part A.

```typescript
// Order & Production Enums
enum OrderStatus {
  OrderCreated = 'OrderCreated',
  PartsSent = 'PartsSent',
  ReadyForPreQC = 'ReadyForPreQC',
  ReadyForProduction = 'ReadyForProduction',
  TestingComplete = 'TestingComplete',
  PackagingComplete = 'PackagingComplete',
  ReadyForFinalQC = 'ReadyForFinalQC',
  ReadyForShip = 'ReadyForShip',
  Shipped = 'Shipped',
}

enum SinkFamily {
  MDRD = 'MDRD',
  ENDOSCOPE = 'Endoscope',
  INSTROSINK = 'InstroSink',
}

```

---

## Error Handling Specific to Part B
// ... existing code ...

```

# Clicked Code Block to apply Changes From

// ... existing code ...
---

## Data Models Specific to Part B

This section defines data models that are primarily used or extended within the scope of Order, BOM, Catalog, and Outsourcing management. Core data models are defined in Part A.

```typescript
// Order & Production Enums
enum OrderStatus {
  OrderCreated = 'OrderCreated',
  PartsSent = 'PartsSent',
  ReadyForPreQC = 'ReadyForPreQC',
  ReadyForProduction = 'ReadyForProduction',
  TestingComplete = 'TestingComplete',
  PackagingComplete = 'PackagingComplete',
  ReadyForFinalQC = 'ReadyForFinalQC',
  ReadyForShip = 'ReadyForShip',
  Shipped = 'Shipped',
}

enum SinkFamily {
  MDRD = 'MDRD',
  ENDOSCOPE = 'Endoscope',
  INSTROSINK = 'InstroSink',
}

```

---

## Error Handling Specific to Part B
// ... existing code ...

```

# Clicked Code Block to apply Changes From

// ... existing code ...
---

## Data Models Specific to Part B

This section defines data models that are primarily used or extended within the scope of Order, BOM, Catalog, and Outsourcing management. Core data models are defined in Part A.

```typescript
// Order & Production Enums
enum OrderStatus {
  OrderCreated = 'OrderCreated',
  PartsSent = 'PartsSent',
  ReadyForPreQC = 'ReadyForPreQC',
  ReadyForProduction = 'ReadyForProduction',
  TestingComplete = 'TestingComplete',
  PackagingComplete = 'PackagingComplete',
  ReadyForFinalQC = 'ReadyForFinalQC',
  ReadyForShip = 'ReadyForShip',
  Shipped = 'Shipped',
}

enum SinkFamily {
  MDRD = 'MDRD',
  ENDOSCOPE = 'Endoscope',
  INSTROSINK = 'InstroSink',
}

```

---

## Error Handling Specific to Part B
// ... existing code ...

```

# Clicked Code Block to apply Changes From

// ... existing code ...
---

## Data Models Specific to Part B

This section defines data models that are primarily used or extended within the scope of Order, BOM, Catalog, and Outsourcing management. Core data models are defined in Part A.

```typescript
// Order & Production Enums
enum OrderStatus {
  OrderCreated = 'OrderCreated',
  PartsSent = 'PartsSent',
  ReadyForPreQC = 'ReadyForPreQC',
  ReadyForProduction = 'ReadyForProduction',
  TestingComplete = 'TestingComplete',
  PackagingComplete = 'PackagingComplete',
  ReadyForFinalQC = 'ReadyForFinalQC',
  ReadyForShip = 'ReadyForShip',
  Shipped = 'Shipped',
}

enum SinkFamily {
  MDRD = 'MDRD',
  ENDOSCOPE = 'Endoscope',
  INSTROSINK = 'InstroSink',
}

```

---

## Error Handling Specific to Part B
// ... existing code ...

```

# Clicked Code Block to apply Changes From

// ... existing code ...
---

## Data Models Specific to Part B

This section defines data models that are primarily used or extended within the scope of Order, BOM, Catalog, and Outsourcing management. Core data models are defined in Part A.

```typescript
// Order & Production Enums
enum OrderStatus {
  OrderCreated = 'OrderCreated',
  PartsSent = 'PartsSent',
  ReadyForPreQC = 'ReadyForPreQC',
  ReadyForProduction = 'ReadyForProduction',
  TestingComplete = 'TestingComplete',
  PackagingComplete = 'PackagingComplete',
  ReadyForFinalQC = 'ReadyForFinalQC',
  ReadyForShip = 'ReadyForShip',
  Shipped = 'Shipped',
}

enum SinkFamily {
  MDRD = 'MDRD',
  ENDOSCOPE = 'Endoscope',
  INSTROSINK = 'InstroSink',
}

```

---

## Error Handling Specific to Part B
// ... existing code ...

```

# Clicked Code Block to apply Changes From

// ... existing code ...
---

## Data Models Specific to Part B

This section defines data models that are primarily used or extended within the scope of Order, BOM, Catalog, and Outsourcing management. Core data models are defined in Part A.

```typescript
// Order & Production Enums
enum OrderStatus {
  OrderCreated = 'OrderCreated',
  PartsSent = 'PartsSent',
  ReadyForPreQC = 'ReadyForPreQC',
  ReadyForProduction = 'ReadyForProduction',
  TestingComplete = 'TestingComplete',
  PackagingComplete = 'PackagingComplete',
  ReadyForFinalQC = 'ReadyForFinalQC',
  ReadyForShip = 'ReadyForShip',
  Shipped = 'Shipped',
}

enum SinkFamily {
  MDRD = 'MDRD',
  ENDOSCOPE = 'Endoscope',
  INSTROSINK = 'InstroSink',
}

```

---

## Error Handling Specific to Part B
// ... existing code ...

```

# Clicked Code Block to apply Changes From

// ... existing code ...
---

## Data Models Specific to Part B

This section defines data models that are primarily used or extended within the scope of Order, BOM, Catalog, and Outsourcing management. Core data models are defined in Part A.

```typescript
// Order & Production Enums
enum OrderStatus {
  OrderCreated = 'OrderCreated',
  PartsSent = 'PartsSent',
  ReadyForPreQC = 'ReadyForPreQC',
  ReadyForProduction = 'ReadyForProduction',
  TestingComplete = 'TestingComplete',
  PackagingComplete = 'PackagingComplete',
  ReadyForFinalQC = 'ReadyForFinalQC',
  ReadyForShip = 'ReadyForShip',
  Shipped = 'Shipped',
}

enum SinkFamily {
  MDRD = 'MDRD',
  ENDOSCOPE = 'Endoscope',
  INSTROSINK = 'InstroSink',
}

```

---

## Error Handling Specific to Part B
// ... existing code ...

```

# Clicked Code Block to apply Changes From

// ... existing code ...
---

## Data Models Specific to Part B

This section defines data models that are primarily used or extended within the scope of Order, BOM, Catalog, and Outsourcing management. Core data models are defined in Part A.

```typescript
// Order & Production Enums
enum OrderStatus {
  OrderCreated = 'OrderCreated',
  PartsSent = 'PartsSent',
  ReadyForPreQC = 'ReadyForPreQC',
  ReadyForProduction = 'ReadyForProduction',
  TestingComplete = 'TestingComplete',
  PackagingComplete = 'PackagingComplete',
  ReadyForFinalQC = 'ReadyForFinalQC',
  ReadyForShip = 'ReadyForShip',
  Shipped = 'Shipped',
}

enum SinkFamily {
  MDRD = 'MDRD',
  ENDOSCOPE = 'Endoscope',
  INSTROSINK = 'InstroSink',
}

```

---

## Error Handling Specific to Part B
// ... existing code ...

```

# Clicked Code Block to apply Changes From

// ... existing code ...
---

## Data Models Specific to Part B

This section defines data models that are primarily used or extended within the scope of Order, BOM, Catalog, and Outsourcing management. Core data models are defined in Part A.

```typescript
// Order & Production Enums
enum OrderStatus {
  OrderCreated = 'OrderCreated',
  PartsSent = 'PartsSent',
  ReadyForPreQC = 'ReadyForPreQC',
  ReadyForProduction = 'ReadyForProduction',
  TestingComplete = 'TestingComplete',
  PackagingComplete = 'PackagingComplete',
  ReadyForFinalQC = 'ReadyForFinalQC',
  ReadyForShip = 'ReadyForShip',
  Shipped = 'Shipped',
}

enum SinkFamily {
  MDRD = 'MDRD',
  ENDOSCOPE = 'Endoscope',
  INSTROSINK = 'InstroSink',
}

```

---

## Error Handling Specific to Part B
// ... existing code ...

```

# Clicked Code Block to apply Changes From

// ... existing code ...
---

## Data Models Specific to Part B

This section defines data models that are primarily used or extended within the scope of Order, BOM, Catalog, and Outsourcing management. Core data models are defined in Part A.

```typescript
// Order & Production Enums
enum OrderStatus {
  OrderCreated = 'OrderCreated',
  PartsSent = 'PartsSent',
  ReadyForPreQC = 'ReadyForPreQC',
  ReadyForProduction = 'ReadyForProduction',
  TestingComplete = 'TestingComplete',
  PackagingComplete = 'PackagingComplete',
  ReadyForFinalQC = 'ReadyForFinalQC',
  ReadyForShip = 'ReadyForShip',
  Shipped = 'Shipped',
}

enum SinkFamily {
  MDRD = 'MDRD',
  ENDOSCOPE = 'Endoscope',
  INSTROSINK = 'InstroSink',
}

```

---

## Error Handling Specific to Part B
// ... existing code ...

```

# Clicked Code Block to apply Changes From

// ... existing code ...
---

## Data Models Specific to Part B

This section defines data models that are primarily used or extended within the scope of Order, BOM, Catalog, and Outsourcing management. Core data models are defined in Part A.

```typescript
// Order & Production Enums
enum OrderStatus {
  OrderCreated = 'OrderCreated',
  PartsSent = 'PartsSent',
  ReadyForPreQC = 'ReadyForPreQC',
  ReadyForProduction = 'ReadyForProduction',
  TestingComplete = 'TestingComplete',
  PackagingComplete = 'PackagingComplete',
  ReadyForFinalQC = 'ReadyForFinalQC',
  ReadyForShip = 'ReadyForShip',
  Shipped = 'Shipped',
}

enum SinkFamily {
  MDRD = 'MDRD',
  ENDOSCOPE = 'Endoscope',
  INSTROSINK = 'InstroSink',
}

```

---

## Error Handling Specific to Part B
// ... existing code ...

```

# Clicked Code Block to apply Changes From

// ... existing code ...
---

## Data Models Specific to Part B

This section defines data models that are primarily used or extended within the scope of Order, BOM, Catalog, and Outsourcing management. Core data models are defined in Part A.

```typescript
// Order & Production Enums
enum OrderStatus {
  OrderCreated = 'OrderCreated',
  PartsSent = 'PartsSent',
  ReadyForPreQC = 'ReadyForPreQC',
  ReadyForProduction = 'ReadyForProduction',
  TestingComplete = 'TestingComplete',
  PackagingComplete = 'PackagingComplete',
  ReadyForFinalQC = 'ReadyForFinalQC',
  ReadyForShip = 'ReadyForShip',
  Shipped = 'Shipped',
}

enum SinkFamily {
  MDRD = 'MDRD',
  ENDOSCOPE = 'Endoscope',
  INSTROSINK = 'InstroSink',
}

```

---

## Error Handling Specific to Part B
// ... existing code ...

```

# Clicked Code Block to apply Changes From

// ... existing code ...
---

## Data Models Specific to Part B

This section defines data models that are primarily used or extended within the scope of Order, BOM, Catalog, and Outsourcing management. Core data models are defined in Part A.

```typescript
// Order & Production Enums
enum OrderStatus {
  OrderCreated = 'OrderCreated',
  PartsSent = 'PartsSent',
  ReadyForPreQC = 'ReadyForPreQC',
  ReadyForProduction = 'ReadyForProduction',
  TestingComplete = 'TestingComplete',
  PackagingComplete = 'PackagingComplete',
  ReadyForFinalQC = 'ReadyForFinalQC',
  ReadyForShip = 'ReadyForShip',
  Shipped = 'Shipped',
}

enum SinkFamily {
  MDRD = 'MDRD',
  ENDOSCOPE = 'Endoscope',
  INSTROSINK = 'InstroSink',
}

```

---

## Error Handling Specific to Part B
// ... existing code ...

```

# Clicked Code Block to apply Changes From

// ... existing code ...
---

## Data Models Specific to Part B

This section defines data models that are primarily used or extended within the scope of Order, BOM, Catalog, and Outsourcing management. Core data models are defined in Part A.

```typescript
// Order & Production Enums
enum OrderStatus {
  OrderCreated = 'OrderCreated',
  PartsSent = 'PartsSent',
  ReadyForPreQC = 'ReadyForPreQC',
  ReadyForProduction = 'ReadyForProduction',
  TestingComplete = 'TestingComplete',
  PackagingComplete = 'PackagingComplete',
  ReadyForFinalQC = 'ReadyForFinalQC',
  ReadyForShip = 'ReadyForShip',
  Shipped = 'Shipped',
}

enum SinkFamily {
}

interface BOMReview_DB { // from 'bom_reviews' table
  reviewId: string; // UUID, Primary Key
--- 