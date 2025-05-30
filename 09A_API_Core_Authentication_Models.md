# API Core, Authentication & Data Models
## Torvan Medical CleanStation Production Workflow Digitalization

**Version:** 1.0  
**Date:** December 2024  
**Document Type:** API Specification - Part A  
**Scope:** Core API, Authentication, Security & Data Models

---

## Executive Summary

This document provides the foundational API specifications for the CleanStation digitalization system. It covers the core API architecture, comprehensive authentication and authorization mechanisms (including JWT, MFA, RBAC, and session management policies), security protocols, fundamental data models derived from the database and operational workflows, and a complete listing of all API endpoints. This is Part A of a comprehensive 4-part API specification series, serving as the definitive source for all core system interfaces and foundational data structures.

### Document Series Overview
- **Part A (This Document)**: Core API, Authentication & Data Models
- **Part B**: Order & BOM Management APIs  
- **Part C**: Quality Control & Assembly APIs
- **Part D**: Service Department, Admin & Integration APIs

---

## API Architecture Overview

### Base Configuration
```typescript
interface APIConfiguration {
  baseURL: 'https://api.cleanstation.torvan.com/v1';
  authentication: 'JWT Bearer Token with RSA256 signing, MFA support, and detailed RBAC policies';
  contentType: 'application/json';
  versioning: 'URL path versioning (/v1, /v2)';
  rateLimit: {
    authenticated: '5000 requests/hour';
    admin: '10000 requests/hour';
    public: '100 requests/hour'; // e.g., for login attempts, public catalog if any
  };
  timeout: '30 seconds'; // Default server response timeout
  retries: 3; // Default retry attempts for idempotent operations by clients
  websocket: 'wss://api.cleanstation.torvan.com/v1/ws';
  securityPolicies: {
    accessTokenExpiry: '1 hour';
    refreshTokenExpiry: '30 days';
    mfaEnforcedRoles: ['Admin', 'QC Person', 'Production Coordinator'];
    idleSessionTimeout: '2 hours';
    absoluteSessionTimeout: '8 hours';
  };
}
```

### Core Endpoints Structure
A high-level overview of API endpoint groups. Detailed specifications for endpoints marked for Part A are within this document. For others, refer to the specified Part B, C, or D documents in this series.

`/api/v1`
├── `/auth`                       # Authentication & session management (Detailed in this document - Part A)
│   ├── /login
│   ├── /logout
│   ├── /refresh
│   ├── /session
│   ├── /reset-password
│   └── /verify
│
├── `/users`                      # User management (Detailed in this document - Part A)
│   ├── /
│   ├── /{userId}
│   ├── /{userId}/profile
│   ├── /{userId}/preferences
│   └── /{userId}/sessions
│
├── `/health`                     # General system health (Detailed in this document - Part A)
│   ├── /status
│   ├── /metrics
│   └── /version
│
├── `/orders`                     # Order management (Detailed in Part B: Order & BOM Management APIs)
├── `/boms`                       # Bill of Materials (BOM) management (Detailed in Part B: Order & BOM Management APIs)
├── `/catalog`                    # Product Catalog (Detailed in Part B: Order & BOM Management APIs)
├── `/outsourcing`                # Parts outsourcing management (Detailed in Part B: Order & BOM Management APIs)
├── `/documents`                  # Document management (Detailed in Part C: Quality Control & Assembly APIs - TBC, or D)
│
├── `/preqc`                      # Pre-Quality Control management (Detailed in Part C: Quality Control & Assembly APIs)
├── `/assembly`                   # Assembly and production workflow (Detailed in Part C: Quality Control & Assembly APIs)
├── `/production/checklists`      # Production checklists (Detailed in Part C: Quality Control & Assembly APIs)
├── `/testing`                    # Testing management (Detailed in Part C: Quality Control & Assembly APIs)
├── `/packaging`                  # Packaging management (Detailed in Part C: Quality Control & Assembly APIs)
├── `/finalqc`                    # Final Quality Control management (Detailed in Part C: Quality Control & Assembly APIs)
│
├── `/notifications`              # User notifications (Detailed in Part D: Service, Admin & Integration APIs)
├── `/service`                    # Service Department operations (Detailed in Part D: Service, Admin & Integration APIs)
├── `/qr`                         # QR Code management (Detailed in Part D: Service, Admin & Integration APIs)
├── `/admin`                      # System administration (Detailed in Part D: Service, Admin & Integration APIs)
├── `/reports`                    # Reporting (Detailed in Part D: Service, Admin & Integration APIs)
├── `/export`                     # Data export (Detailed in Part D: Service, Admin & Integration APIs)
└── `/websocket`                  # Real-time communication (Detailed in Part D: Service, Admin & Integration APIs)

### API Design Principles
```typescript
interface APIDesignPrinciples {
  restful: 'RESTful design with standard HTTP methods';
  stateless: 'Stateless operations with JWT tokens';
  cacheable: 'Cacheable responses with appropriate headers';
  layered: 'Layered architecture with clear separation';
  uniform: 'Uniform interface across all endpoints';
  hypermedia: 'HATEOAS for discoverable API navigation';
}
```

---

## Data Models

This section defines the core data structures used throughout the API, including representations of database entities and request/response payloads. All timestamps are in ISO 8601 format (e.g., `YYYY-MM-DDTHH:mm:ss.sssZ`) unless otherwise specified.

### User Management Models

```typescript
// UserProfile, AuthenticationRequest, AuthenticationResponse, SessionInfo, TokenRefreshRequest, TokenRefreshResponse
// are defined in the "Authentication & Authorization" section and are the primary reference.

// UserRole enum is defined in the "Authentication & Authorization" section (RBAC subsection).

interface User { // Represents the 'users' database table structure, primarily for internal or admin use.
  userId: string;                      // UUID, Primary Key, from 'users' table
  username: string;                    // Unique username
  email: string;                       // Unique email address
  passwordHash: string;                // Hashed password (NEVER exposed in API responses)
  fullName: string;
  initials?: string;                   // For QC/Assembly sign-offs
  role: UserRole;                      // User's primary role (references UserRole enum from Auth section)
  isActive: boolean;                   // Account status
  department?: string;                  // User's department
  // mfaEnabled is part of UserProfile, which is the API representation.
  // This model is closer to DB, so mfa secret storage details would be here but not exposed.
  lastLogin?: string;                  // ISO 8601 datetime string
  createdAt: string;                   // ISO 8601 datetime string
  updatedAt: string;                   // ISO 8601 datetime string
}

interface UserSession_DB { // Represents the 'user_sessions' database table structure.
    sessionId: string; // UUID, Primary Key, from 'user_sessions' table
    userId: string; // UUID, Foreign Key to users(user_id)
    createdAt: string; // ISO 8601 datetime string
    expiresAt: string; // ISO 8601 datetime string
    ipAddress?: string; // INET
    userAgent?: string;
    // refreshTokenHash would be stored here, not the raw token.
}
```

### Audit Log Models (Core)
```typescript
// From 07_Phase4 - Audit Log & Reports (Core part retained here)
// CREATE TYPE audit_event_type_enum AS ENUM (
// 'UserAction', 'SystemAction', 'DataChange', 'SecurityEvent', 'PerformanceEvent'
// );
enum AuditEventType {
  USER_ACTION = 'UserAction',
  SYSTEM_ACTION = 'SystemAction',
  DATA_CHANGE = 'DataChange',
  SECURITY_EVENT = 'SecurityEvent',
  PERFORMANCE_EVENT = 'PerformanceEvent',
}

// CREATE TYPE audit_action_enum AS ENUM (
// 'Create', 'Read', 'Update', 'Delete', 'Login', 'Logout', 'Export', 'Import'
// );
enum AuditAction {
  CREATE = 'Create',
  READ = 'Read',
  UPDATE = 'Update',
  DELETE = 'Delete',
  LOGIN = 'Login',
  LOGOUT = 'Logout',
  EXPORT = 'Export',
  IMPORT = 'Import',
  APPROVE = 'Approve', // Adding more specific actions
  REJECT = 'Reject',
  REVIEW = 'Review',
  START_TASK = 'StartTask',
  COMPLETE_TASK = 'CompleteTask',
  STATUS_CHANGE = 'StatusChange',
  // etc.
}

interface SystemAuditLogEntry {
  auditId: string; // UUID, Primary Key from 'system_audit_log' table
  userId?: string; // User ID, if action performed by a user
  sessionId?: string; // Session ID, if available
  eventType: AuditEventType;
  entityType?: string; // Table or object type affected (e.g., 'Order', 'User', 'Part')
  entityId?: string; // ID of the affected entity
  action: AuditAction; // Or a more descriptive string if not strictly enum
  oldValues?: any; // JSONB - Previous values for Update actions
  newValues?: any; // JSONB - New values for Create/Update actions
  ipAddress?: string; // INET
  userAgent?: string;
  timestamp: string;
  success: boolean;
  errorMessage?: string;
  // additionalContext from 11_Security might be part of newValues or a separate field
}
```

### Inventory & Catalog Models

```typescript
interface Category {
  categoryId: string; // e.g., "718", Primary Key from 'categories' table
  name: string;
  description?: string;
  createdAt: string;
  updatedAt: string;
}

interface Subcategory {
  subcategoryId: string; // e.g., "718.001", Primary Key from 'subcategories' table
  parentCategoryId: string; // Foreign Key to categories(category_id)
  name: string;
  description?: string;
  createdAt: string;
  updatedAt: string;
}

// Enum for PartType (from 'parts' table)
// CREATE TYPE part_type_enum AS ENUM ('COMPONENT', 'MATERIAL');
enum PartType {
  COMPONENT = 'COMPONENT',
  MATERIAL = 'MATERIAL',
}

// Enum for PartStatus (from 'parts' table)
// CREATE TYPE part_status_enum AS ENUM ('ACTIVE', 'INACTIVE', 'DISCONTINUED');
enum PartStatus {
  ACTIVE = 'ACTIVE',
  INACTIVE = 'INACTIVE',
  DISCONTINUED = 'DISCONTINUED',
}

interface Part {
  partId: string; // Primary Key from 'parts' table
  name: string;
  manufacturerPartNumber?: string;
  manufacturerInfo?: string;
  type: PartType;
  status: PartStatus;
  photoUrl?: string;
  technicalDrawingUrl?: string;
  description?: string;
  createdAt: string;
  updatedAt: string;
  // Fields for catalog management by Admin might include cost, supplier, etc.
}

// Enum for AssemblyType (from 'assemblies' table)
// CREATE TYPE assembly_type_enum AS ENUM ('SIMPLE', 'COMPLEX', 'SERVICE_PART', 'KIT');
enum AssemblyType {
  SIMPLE = 'SIMPLE',
  COMPLEX = 'COMPLEX',
  SERVICE_PART = 'SERVICE_PART',
  KIT = 'KIT',
}

// Enum for AssemblyStatus (from 'assemblies' table, for catalog status)
// CREATE TYPE assembly_status_enum AS ENUM ('ACTIVE', 'INACTIVE', 'DISCONTINUED');
enum AssemblyCatalogStatus { // Renamed to avoid conflict with production assembly status
  ACTIVE = 'ACTIVE',
  INACTIVE = 'INACTIVE',
  DISCONTINUED = 'DISCONTINUED',
}

interface Assembly {
  assemblyId: string; // Primary Key from 'assemblies' table
  name: string;
  type: AssemblyType;
  categoryCode?: string; // Foreign Key to categories(category_id)
  subcategoryCode?: string; // Foreign Key to subcategories(subcategory_id)
  canOrder: boolean;
  isKit: boolean;
  status: AssemblyCatalogStatus;
  photoUrl?: string;
  technicalDrawingUrl?: string;
  qrData?: string; // For QR code generation, might be URI or encoded data
  kitComponents?: any; // JSONB - Define specific structure if known, e.g., KitComponentDetails[]
  workInstructionId?: string; // UUID, Foreign Key to work_instructions
  description?: string;
  createdAt: string;
  updatedAt: string;
}

interface KitComponentDetails { // Example structure for Assembly.kitComponents
  partId: string;
  quantity: number;
  notes?: string;
}

interface AssemblyComponent {
  componentId: string; // UUID, Primary Key from 'assembly_components' table
  parentAssemblyId: string; // Foreign Key to assemblies(assembly_id)
  childPartId?: string; // Foreign Key to parts(part_id)
  childAssemblyId?: string; // Foreign Key to assemblies(assembly_id)
  quantity: number;
  notes?: string;
  isOptional: boolean;
  createdAt: string;
  // check_child_reference constraint handled by backend logic
}

// From 04_Phase1_Foundation_Core_Order_Management.md (for GET /api/accessories)
interface Accessory extends Part { // Accessories are often specialized parts or simple assemblies
  // Inherits from Part, or could be a union type if significantly different
  category: string; // Category name for UI grouping
}
```

### Order & Production Models

```typescript
// Enum for OrderStatus (from 'orders' table)
// CREATE TYPE order_status_enum AS ENUM (
//     'OrderCreated', 'PartsSent', 'ReadyForPreQC', 'ReadyForProduction',
//     'TestingComplete', 'PackagingComplete', 'ReadyForFinalQC', 'ReadyForShip', 'Shipped'
// );
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
  // Potentially more granular statuses like 'InAssembly', 'InTesting', etc.
}

// Enum for Language (from 'orders' table)
// CREATE TYPE language_enum AS ENUM ('EN', 'FR', 'SP');
enum DocumentLanguage {
  EN = 'EN',
  FR = 'FR',
  SP = 'SP',
}

// Enum for SinkFamily (from 'orders' table)
// CREATE TYPE sink_family_enum AS ENUM ('MDRD', 'Endoscope', 'InstroSink');
enum SinkFamily {
  MDRD = 'MDRD',
  ENDOSCOPE = 'Endoscope',
  INSTROSINK = 'InstroSink',
}

// From 04_Phase1 (Order Creation Wizard)
interface OrderStepOne {
  poNumber: string;
  customerName: string;
  projectName?: string;
  salesPerson: string;
  wantDate: string; // ISO 8601 Date (YYYY-MM-DD)
  poDocumentUrl?: string; // URL after upload
  notes?: string;
  documentLanguage: DocumentLanguage;
}

interface BuildNumberItem { // From 04_Phase1
  id: string; // client-side temporary ID
  buildNumber: string;
  isValid?: boolean;
}

interface OrderStepTwo { // From 04_Phase1
  sinkFamily: SinkFamily;
  quantity: number;
  buildNumbers: BuildNumberItem[];
}

interface DimensionSpec { // From 04_Phase1
  width: number; // inches
  length: number; // inches
  depth?: number; // inches, optional
}

interface PegboardConfig { // From 04_Phase1
  color: 'Green' | 'Black' | 'Yellow' | 'Grey' | 'Red' | 'Blue' | 'Orange' | 'White';
  type: 'Perforated' | 'Solid';
  size: {
    type: 'SameAsSink' | 'Custom';
    width?: number; // if custom
    length?: number; // if custom
  };
}

interface CustomBasinSize { // From 04_Phase1
  type: 'Custom';
  width: number;
  length: number;
  depth: number;
}

interface StandardBasinSize { // From 04_Phase1
  type: 'Standard';
  model: string; // e.g., 'B16x19x10'
}

interface BasinAddon { // From 04_Phase1
  addonId: string;
  name: string;
  selected: boolean;
}

interface BasinConfigurationItem { // From 04_Phase1
  index: number; // 0-indexed or 1-indexed, clarify
  type: 'E-Sink' | 'E-Sink DI' | 'E-Drain';
  size: StandardBasinSize | CustomBasinSize;
  addons: BasinAddon[];
}

interface FaucetConfigurationItem { // From 04_Phase1
  type: string; // Predefined faucet model ID/name
  quantity: number;
  placement: string; // e.g., 'Center', 'LeftOfBasin1', specific to sink model and basin count
}

interface SprayerConfigurationItem { // From 04_Phase1
  type: string; // Predefined sprayer model ID/name
  quantity: 1 | 2;
  location: 'LeftSide' | 'RightSide';
}

interface SinkConfigurationPayload { // From 04_Phase1, per build number
  buildNumber: string;
  sinkModel: 'T2-B1' | 'T2-B2' | 'T2-B3'; // Example models, should come from catalog
  sinkDimensions: DimensionSpec;
  legsType: 'HeightAdjustable' | 'FixedHeight';
  legsModel: string; // Specific model based on legsType, e.g., 'DL27', 'DL14', 'LC1'
  feetType: 'LockLevelingCasters' | 'SSAdjustableSeismicFeet';
  pegboard?: PegboardConfig | null;
  workflowDirection: 'LeftToRight' | 'RightToLeft';
  basins: BasinConfigurationItem[];
  faucets: FaucetConfigurationItem[];
  sprayers: SprayerConfigurationItem[];
}

interface AccessorySelectionItem { // From 04_Phase1
  accessoryId: string; // Part or Assembly ID
  name: string;
  category: string;
  quantity: number;
  // canOrder was for UI, not needed in payload if backend validates
}

// Represents the full payload for creating an order (POST /api/orders)
interface CreateOrderPayload {
  stepOne: OrderStepOne;
  stepTwo: OrderStepTwo;
  sinkConfigurations: SinkConfigurationPayload[]; // One per build number
  accessorySelections: AccessorySelectionItem[];
  // Additional overall order notes or fields if any
}

interface Order {
  orderId: string; // UUID, Primary Key from 'orders' table
  poNumber: string;
  // buildNumber: string; // This seems to be per sink instance inside an order, see sink_configurations
  customerName: string;
  projectName?: string;
  salesPerson: string;
  wantDate: string; // ISO 8601 Date
  orderLanguage: DocumentLanguage;
  sinkFamily: SinkFamily;

  // Sink configurations (denormalized or from related tables)
  // For a list view, this might be a summary. For GET /{orderId}, it's detailed.
  // The SinkConfigurationPayload (from create) is a good structure for the detailed view.
  // For database `orders` table, these JSONB fields store this information:
  // sink_dimensions: JSONB, legs_type: VARCHAR, legs_model: VARCHAR, feet_type: VARCHAR,
  // has_pegboard: BOOLEAN, pegboard_color: VARCHAR, pegboard_type: VARCHAR, pegboard_size: JSONB,
  // workflow_direction: VARCHAR,
  // basin_configurations: JSONB, faucet_configurations: JSONB, sprayer_configurations: JSONB,
  // selected_accessories: JSONB
  // It's better to model these as structured objects in the API response:
  sinkSpecificDetails: any; // This needs to be a well-defined interface based on SinkConfigurationPayload
                           // or a simplified summary for list views.

  status: OrderStatus;
  currentAssigneeId?: string; // UUID, Foreign key to users(user_id)
  generatedBomId?: string; // UUID, Foreign key to boms(bom_id)
  createdBy: string; // User ID
  createdAt: string;
  updatedAt: string;
  // Include buildNumbers associated with this order if it has multiple builds:
  builds?: SinkConfigurationPayload[]; // Reflects the multiple builds within one PO
}

// For GET /api/orders (list view item)
interface OrderListItem {
  orderId: string;
  poNumber: string;
  customerName: string;
  salesPerson: string;
  wantDate: string;
  sinkFamily: SinkFamily;
  status: OrderStatus;
  buildCount: number;
  createdAt: string;
  // Add other summary fields as needed, e.g., currentAssigneeName
}

interface OrderHistory {
  historyId: string; // UUID, Primary Key from 'order_history' table
  orderId: string; // Foreign Key to orders(order_id)
  userId: string; // Foreign Key to users(user_id)
  action: string;
  oldStatus?: OrderStatus;
  newStatus?: OrderStatus;
  notes?: string;
  createdAt: string;
}

interface OrderDocument {
  documentId: string; // UUID, Primary Key from 'order_documents' table
  orderId: string; // Foreign Key to orders(order_id)
  filename: string;
  fileUrl: string;
  fileType?: string;
  fileSize?: number; // in bytes
  uploadedById: string; // User ID
  uploadedAt: string;
  documentType?: string; // e.g., 'PO', 'TechnicalDrawing', 'QCForm'
}
```

### Bill of Materials (BOM) Models

```typescript
// From 04_Phase1 - BOM Structure
interface BOMItem {
  id: string; // Part or Assembly ID
  type: 'PART' | 'ASSEMBLY';
  name: string;
  partNumber: string; // Manufacturer part number or internal assembly ID
  quantity: number;
  level: number; // Hierarchy level in the BOM (0 for top-level)
  parent?: string; // Parent assembly ID if applicable
  children?: BOMItem[]; // Child components for assemblies
  notes?: string;
  // Add other relevant fields like description, manufacturer, etc.
}

interface BOMStructure {
  bomId: string; // UUID, Primary Key from 'boms' table if persisted, or generated ID
  orderId: string; // Foreign key if order-specific BOM
  buildNumber?: string; // If BOM is per build number within an order
  items: BOMItem[];
  totalUniqueParts: number;
  totalAssemblies: number;
  generatedAt: string;
  generatedById: string; // User ID
  status: 'Generated' | 'Approved' | 'Rejected' | 'Superseded'; // from boms table or logical status
  approvedById?: string; // User ID
  approvedAt?: string;
  version: number;
  // bom_items from 'boms' table (JSONB) would map to `items: BOMItem[]`
}

interface BOMApproval {
  approvalId: string; // UUID, Primary key from 'bom_approvals' table
  bomId: string; // Foreign Key to boms(bom_id)
  userId: string; // User ID of reviewer/approver
  action: 'Approved' | 'Rejected' | 'Modified' | 'Reviewed'; // Extended based on bom_review_status_enum
  notes?: string;
  createdAt: string;
}

// From 05_Phase2 - BOM Review
// CREATE TYPE bom_review_status_enum AS ENUM (
//     'Pending', 'InReview', 'Approved', 'Rejected', 'RequiresModification'
// );
enum BOMReviewStatus {
  PENDING = 'Pending',
  IN_REVIEW = 'InReview',
  APPROVED = 'Approved',
  REJECTED = 'Rejected',
  REQUIRES_MODIFICATION = 'RequiresModification',
}

interface BOMReview_DB { // from 'bom_reviews' table
  reviewId: string; // UUID, Primary Key
  bomId: string; // Foreign Key to boms(bom_id)
  reviewerId: string; // User ID
  reviewStatus: BOMReviewStatus;
  reviewNotes?: string;
  modifications?: any; // JSONB - define structure if known (e.g., list of changes)
  reviewedAt: string;
  approvedAt?: string;
  rejectedAt?: string;
}

interface BOMReviewRequest { // For POST /api/boms/{bomId}/review
  reviewStatus: BOMReviewStatus;
  reviewNotes?: string;
  modifications?: any; // Proposed modifications
}

interface BOMReviewItem { // From 05_Phase2 (for dashboard)
  bomId: string;
  orderId: string;
  poNumber: string;
  customerName: string;
  totalParts: number;
  totalAssemblies: number;
  priority: 'High' | 'Medium' | 'Low';
  submittedAt: string;
  dueDate: string;
  complexity: 'Simple' | 'Moderate' | 'Complex';
  reviewStatus: BOMReviewStatus;
}
```

### Quality Control (QC) Models

```typescript
// Enum for QCFormType (from 'qc_form_templates' table & 05_Phase2 preqc_completions)
// CREATE TYPE qc_form_type_enum AS ENUM ('PreQC', 'FinalQC', 'InProcessAssembly');
enum QCFormType {
  PRE_QC = 'PreQC',
  FINAL_QC = 'FinalQC',
  IN_PROCESS_ASSEMBLY = 'InProcessAssembly',
}

interface QCFormTemplate {
  templateId: string; // UUID, Primary Key from 'qc_form_templates' table
  formName: string;
  formType: QCFormType;
  version: number;
  isActive: boolean;
  checklistItems: any; // JSONB - This needs a very detailed structure based on actual checklists
                       // e.g., QCSection[] where QCSection has QCItem[] etc.
  createdById: string; // User ID
  createdAt: string;
  updatedAt: string;
}

// Enum for QCStatus (overall status of a QC result, from 'qc_results' table)
// CREATE TYPE qc_status_enum AS ENUM ('Pass', 'Fail', 'Incomplete', 'InProgress');
enum QCResultStatus {
  PASS = 'Pass',
  FAIL = 'Fail',
  INCOMPLETE = 'Incomplete',
  IN_PROGRESS = 'InProgress',
  CONDITIONAL_PASS = 'ConditionalPass', // From PreQC
  REWORK = 'Rework', // From ProductionChecklist
  HOLD = 'Hold', // From FinalQC
  CONDITIONAL_RELEASE = 'ConditionalRelease' // From FinalQC
}

// Generic QC Result structure, can be base for specific QC types
interface QCResult {
  resultId: string; // UUID, Primary Key from 'qc_results' (or specific like 'preqc_completions')
  orderId: string; // Foreign Key to orders(order_id)
  buildNumber?: string;
  templateId: string; // Foreign Key to qc_form_templates(template_id)
  qcType: QCFormType; // Or a more specific type if not using generic template
  performedById: string; // User ID
  jobId?: string; // As per checklist requirements
  overallStatus: QCResultStatus;
  itemResults: any; // JSONB - Detailed results for each checklist item, structure depends on template
  digitalSignature: string; // User initials + timestamp or crypto signature
  performedAt: string;
  notes?: string;
  attachments?: any; // JSONB from PreQC - e.g., { name: string, url: string }[]
}

// From 05_Phase2 - PreQC specifics
interface PreQCFormPayload { // For POST /api/preqc/{orderId}/complete
  templateId: string;
  jobId?: string;
  numberOfBasins?: number; // Example specific field
  // checklist_results: follows structure of qc_form_templates.checklistItems for PreQC type
  checklistResults: any; // This will be the actual filled checklist
  overallResult: QCResultStatus.PASS | QCResultStatus.FAIL | QCResultStatus.CONDITIONAL_PASS;
  digitalSignature: string;
  notes?: string;
  attachments?: { name: string, url: string }[];
}

// From 06_Phase3 - Production Checklist specifics
// CREATE TYPE production_section_enum AS ENUM ('Section2_Assembly', 'Section3_Installation');
enum ProductionChecklistSection {
  SECTION2_ASSEMBLY = 'Section2_Assembly',
  SECTION3_INSTALLATION = 'Section3_Installation',
}

interface ProductionChecklistPayload { // For POST /api/production/checklists/complete
  orderId: string;
  buildNumber: string;
  section: ProductionChecklistSection;
  templateId: string;
  jobId?: string;
  checklistResults: any; // Filled checklist data for the section
  measurements?: any; // JSONB
  digitalSignature: string;
  overallResult: QCResultStatus.PASS | QCResultStatus.FAIL | QCResultStatus.REWORK;
  notes?: string;
}

// From 07_Phase4 - Final QC specifics
interface FinalQCPayload { // For POST /api/finalqc/{orderId}/complete
  templateId: string;
  qcDate: string; // YYYY-MM-DD
  jobId?: string;
  // Specific check sections as JSON objects, matching the FinalQCForm structure from 07_...md
  visualInspection: any;
  functionalTesting: any;
  documentationReview: any;
  finalMeasurements: any;
  complianceVerification: any;
  overallResult: QCResultStatus.PASS | QCResultStatus.FAIL | QCResultStatus.CONDITIONAL_RELEASE | QCResultStatus.HOLD;
  digitalSignature: string;
  notes?: string;
}

interface FinalQCResult extends QCResult { // Extends base QCResult or has specific fields
  releaseAuthorization: boolean;
  authorizedById?: string; // User ID
  authorizationTimestamp?: string;
  // Includes the detailed check sections from FinalQCPayload
  visualInspectionResults: any;
  functionalTestingResults: any;
  documentationReviewResults: any;
  finalMeasurementsResults: any;
  complianceVerificationResults: any;
}
```

### Work Instructions & Task Models

```typescript
interface WorkInstruction {
  instructionId: string; // UUID, Primary Key from 'work_instructions' table
  title: string;
  associatedAssemblyId?: string; // Foreign Key to assemblies(assembly_id)
  steps: WorkInstructionStep[]; // JSONB - [{stepNumber, description, visualUrl, safetyNotes}]
  version: number;
  isActive: boolean;
  createdById: string; // User ID
  createdAt: string;
  updatedAt: string;
}

interface WorkInstructionStep {
  stepNumber: number;
  description: string;
  visualUrl?: string;
  safetyNotes?: string;
  estimatedTime?: number; // minutes, from 06_Phase3 TaskExecutionInterface
  verificationRequired?: boolean; // from 06_Phase3 TaskExecutionInterface
}

interface Tool {
  toolId: string; // UUID, Primary Key from 'tools' table
  name: string;
  description?: string;
  imageUrl?: string;
  isActive: boolean;
  createdAt: string;
}

// From 06_Phase3 - Assembly Task Management
// CREATE TYPE task_type_enum AS ENUM (
//     'Assembly', 'QualityCheck', 'Testing', 'Packaging', 'Documentation', 'Verification'
// );
enum AssemblyTaskType {
  ASSEMBLY = 'Assembly',
  QUALITY_CHECK = 'QualityCheck',
  TESTING = 'Testing',
  PACKAGING = 'Packaging',
  DOCUMENTATION = 'Documentation',
  VERIFICATION = 'Verification',
}

// CREATE TYPE task_status_enum AS ENUM (
//     'Pending', 'Ready', 'InProgress', 'Blocked', 'Completed', 'Failed', 'Skipped'
// );
enum AssemblyTaskStatus {
  PENDING = 'Pending',
  READY = 'Ready',
  IN_PROGRESS = 'InProgress',
  BLOCKED = 'Blocked',
  COMPLETED = 'Completed',
  FAILED = 'Failed',
  SKIPPED = 'Skipped',
}

interface AssemblyTaskRequiredPart {
  partId: string;
  quantity: number;
  verified?: boolean;
  location?: string; // from 06_Phase3 TaskExecutionInterface
  alternativeParts?: string[]; // from 06_Phase3 TaskExecutionInterface
}

interface AssemblyTaskRequiredTool {
  toolId: string;
  name: string; // Denormalized for convenience
}

interface AssemblyTaskQualityCheck {
  checkId: string; // Identifier for the quality check step
  description: string;
  required: boolean;
  result?: 'Pass' | 'Fail' | 'NotApplicable'; // Or more specific enum
  notes?: string;
}

interface AssemblyTask {
  taskId: string; // UUID, Primary Key from 'assembly_tasks' table
  orderId: string;
  buildNumber: string;
  taskType: AssemblyTaskType;
  sequenceNumber: number;
  title: string;
  description?: string;
  requiredParts?: AssemblyTaskRequiredPart[]; // JSONB
  requiredTools?: AssemblyTaskRequiredTool[]; // JSONB
  workInstructionId?: string; // Foreign Key to work_instructions
  qualityChecks?: AssemblyTaskQualityCheck[]; // JSONB
  status: AssemblyTaskStatus;
  assignedToId?: string; // User ID
  estimatedDuration?: number; // minutes
  startedAt?: string;
  completedAt?: string;
  completedById?: string; // User ID
  completionNotes?: string;
  createdAt: string;
  updatedAt: string;
  // Fields from 06_Phase3 AssemblerDashboard.AssemblyTask for UI
  priority?: 'High' | 'Medium' | 'Low';
  requiredSkills?: string[];
  dueDate?: string;
}

interface TaskCompletion_DB { // from 'task_completions' table (generic)
  completionId: string; // UUID, Primary Key
  task_list_id: string; // This seems to refer to a generic 'task_lists' table not fully detailed for assembly tasks
  orderId: string;
  taskSequence: number;
  completedById: string; // User ID
  completedAt: string;
  notes?: string;
}
```

### Testing & Results Models

```typescript
interface TestingFormTemplate {
  templateId: string; // UUID, Primary Key from 'testing_form_templates' table
  formName: string;
  testType: string; // Could be an enum if standardized e.g., PressureTest, LeakTest
  version: number;
  isActive: boolean;
  testItems: any; // JSONB - Structure defining test steps, parameters, expected outcomes
  createdById: string; // User ID
  createdAt: string;
}

// From 06_Phase3 - More detailed testing enums
// CREATE TYPE test_type_enum AS ENUM (
//     'PressureTest', 'LeakTest', 'FunctionalTest', 'SafetyTest', 'PerformanceTest'
// );
enum TestProcedureType {
  PRESSURE_TEST = 'PressureTest',
  LEAK_TEST = 'LeakTest',
  FUNCTIONAL_TEST = 'FunctionalTest',
  SAFETY_TEST = 'SafetyTest',
  PERFORMANCE_TEST = 'PerformanceTest',
}

// CREATE TYPE test_result_enum AS ENUM ('Pass', 'Fail', 'Conditional', 'Retest');
enum TestProcedureResultStatus {
  PASS = 'Pass',
  FAIL = 'Fail',
  CONDITIONAL = 'Conditional',
  RETEST = 'Retest',
}

interface TestingProcedure {
  procedureId: string; // UUID, Primary Key from 'testing_procedures' table
  orderId: string;
  buildNumber: string;
  testType: TestProcedureType;
  testParameters: any; // JSONB - Test-specific input parameters
  testResults: any; // JSONB - Recorded results, measurements, observations
  performedById: string; // User ID
  testingEquipment?: any; // JSONB - e.g., { equipmentId: string, name: string, calibrationDate: string }[]
  environmentalConditions?: any; // JSONB - e.g., { temperature: string, humidity: string }
  result: TestProcedureResultStatus;
  performedAt: string;
  verifiedById?: string; // User ID
  verificationNotes?: string;
}

interface TestingResult_DB { // from 'testing_results' table (generic)
  resultId: string; // UUID, Primary Key
  orderId: string;
  templateId: string; // Foreign Key to testing_form_templates
  performedById: string; // User ID
  overallStatus: TestProcedureResultStatus; // CREATE TYPE test_status_enum AS ENUM ('Pass', 'Fail', 'Incomplete'); - this was simpler
  testResults: any; // JSONB
  performedAt: string;
  notes?: string;
}
```

### Service Department Models

```typescript
// From 07_Phase4 - Enhanced service request models
// CREATE TYPE service_urgency_enum AS ENUM ('Low', 'Medium', 'High', 'Emergency');
enum ServiceUrgency {
  LOW = 'Low',
  MEDIUM = 'Medium',
  HIGH = 'High',
  EMERGENCY = 'Emergency',
}

// CREATE TYPE service_request_type_enum AS ENUM ('Replacement', 'Upgrade', 'Maintenance', 'Warranty');
enum ServiceRequestType {
  REPLACEMENT = 'Replacement',
  UPGRADE = 'Upgrade',
  MAINTENANCE = 'Maintenance',
  WARRANTY = 'Warranty',
}

// CREATE TYPE service_request_status_enum AS ENUM (
// 'Pending', 'Approved', 'Rejected', 'Processing', 'Shipped', 'Delivered', 'Completed'
// );
enum ServiceRequestStatus {
  PENDING = 'Pending',
  APPROVED = 'Approved',
  REJECTED = 'Rejected',
  PROCESSING = 'Processing',
  SHIPPED = 'Shipped',
  DELIVERED = 'Delivered',
  COMPLETED = 'Completed',
  // Original 'Fulfilled' from service_order_status_enum could be mapped to 'Delivered' or 'Completed'
}

interface ServiceCustomerInfo {
  name: string;
  contactPerson?: string;
  email?: string;
  phone?: string;
  address?: any; // { street, city, state, zip, country }
}

interface RequestedServiceItem {
  partId: string;
  quantity: number;
  reason?: string;
  customerPartNumber?: string; // If customer has their own P/N
  failureDescription?: string; // For warranty/replacement
}

interface ServicePartRequest {
  requestId: string; // UUID, Primary Key from 'service_part_requests' table
  requestedById: string; // User ID
  requestTimestamp: string;
  customerInfo: ServiceCustomerInfo; // JSONB
  urgencyLevel: ServiceUrgency;
  requestType: ServiceRequestType;
  requestedItems: RequestedServiceItem[]; // JSONB
  status: ServiceRequestStatus;
  assignedToId?: string; // User ID
  approvalRequired: boolean;
  approvedById?: string; // User ID
  approvedAt?: string;
  approvalNotes?: string;
  processedById?: string; // User ID
  processedTimestamp?: string;
  processingNotes?: string;
  estimatedDelivery?: string; // ISO 8601 Date
  trackingNumber?: string;
}

// Original simpler service_orders from DB schema
// CREATE TYPE service_order_status_enum AS ENUM ('Pending', 'Approved', 'Rejected', 'Fulfilled');
interface ServiceOrder_DB {
  serviceOrderId: string; // UUID, Primary Key
  requestedById: string; // User ID
  requestTimestamp: string;
  status: 'Pending' | 'Approved' | 'Rejected' | 'Fulfilled';
  items: { partId: string, quantity: number }[]; // JSONB
  processedById?: string; // User ID
  processedTimestamp?: string;
  processingNotes?: string;
}
```

### Other Models (Notifications, Outsourcing, Packaging, Progress, QR, Audit, Reports)

```typescript
// From 05_Phase2 - Notifications
// CREATE TYPE notification_type_enum AS ENUM (
//     'BOMReviewRequired', 'BOMApproved', 'BOMRejected', 'PreQCRequired',
//     'PreQCCompleted', 'OrderStatusChanged', 'DocumentRequired', 'OutsourcingUpdate'
// );
enum NotificationType {
  BOM_REVIEW_REQUIRED = 'BOMReviewRequired',
  BOM_APPROVED = 'BOMApproved',
  BOM_REJECTED = 'BOMRejected',
  PRE_QC_REQUIRED = 'PreQCRequired',
  PRE_QC_COMPLETED = 'PreQCCompleted',
  ORDER_STATUS_CHANGED = 'OrderStatusChanged',
  DOCUMENT_REQUIRED = 'DocumentRequired',
  OUTSOURCING_UPDATE = 'OutsourcingUpdate',
  // Add more as system evolves, e.g., ASSEMBLY_TASK_ASSIGNED, FINAL_QC_PENDING etc.
}

interface Notification {
  notificationId: string; // UUID, Primary Key from 'notifications' table
  userId: string; // Foreign Key to users(user_id) - recipient
  type: NotificationType;
  title: string;
  message: string;
  relatedOrderId?: string;
  relatedBomId?: string;
  // Add other related entity IDs as needed: taskId, qcResultId etc.
  isRead: boolean;
  createdAt: string;
  expiresAt?: string;
  // For UI from 05_Phase2 NotificationRequest
  urgency?: 'Low' | 'Medium' | 'High' | 'Critical';
  actionRequired?: boolean;
}

interface SendNotificationPayload { // For POST /api/notifications/send (system)
  recipients: string[]; // Array of User IDs
  type: NotificationType;
  title: string;
  message: string;
  urgency?: 'Low' | 'Medium' | 'High' | 'Critical';
  relatedOrderId?: string;
  relatedBomId?: string;
  actionRequired?: boolean;
  expiresAt?: string;
}

// From 05_Phase2 - Outsourcing
// CREATE TYPE outsourcing_status_enum AS ENUM (
//     'Identified', 'Quoted', 'Ordered', 'InProduction',
//     'Shipped', 'Received', 'QCFailed', 'Completed'
// );
enum OutsourcingStatus {
  IDENTIFIED = 'Identified',
  QUOTED = 'Quoted',
  ORDERED = 'Ordered',
  IN_PRODUCTION = 'InProduction',
  SHIPPED = 'Shipped',
  RECEIVED = 'Received',
  QC_FAILED = 'QCFailed',
  COMPLETED = 'Completed',
}

interface OutsourcedPart {
  outsourcingId: string; // UUID, Primary Key from 'outsourced_parts' table
  bomId: string; // Foreign Key to boms(bom_id)
  partId: string; // Foreign Key to parts(part_id)
  assemblyId?: string; // Foreign Key to assemblies(assembly_id), if part is within an assembly
  quantity: number;
  vendorName?: string;
  expectedDelivery?: string; // ISO 8601 Date
  status: OutsourcingStatus;
  trackingNumber?: string;
  notes?: string;
  createdById: string; // User ID
  createdAt: string;
  updatedAt: string;
}

interface IdentifyOutsourcingPayload { // For POST /api/outsourcing/identify
  bomId: string;
  items: { partId: string; quantity: number; reason?: string }[];
}

interface UpdateOutsourcingStatusPayload { // For PUT /api/outsourcing/{id}/status
  status: OutsourcingStatus;
  vendorName?: string;
  expectedDelivery?: string;
  trackingNumber?: string;
  notes?: string;
}

// From 06_Phase3 - Packaging
// CREATE TYPE packaging_type_enum AS ENUM (
//     'StandardPackaging', 'CustomPackaging', 'ShippingPackaging', 'ProtectivePackaging'
// );
enum PackagingType {
  STANDARD_PACKAGING = 'StandardPackaging',
  CUSTOM_PACKAGING = 'CustomPackaging',
  SHIPPING_PACKAGING = 'ShippingPackaging',
  PROTECTIVE_PACKAGING = 'ProtectivePackaging',
}

interface PackagingRequirement {
  requirementId: string; // UUID, Primary Key from 'packaging_requirements' table
  orderId: string;
  buildNumber: string;
  packagingType: PackagingType;
  requirements: any; // JSONB - e.g., { crateSize: string, materials: string[], instructions: string }
  completed: boolean;
  completedById?: string; // User ID
  completedAt?: string;
  verificationPhotos?: { name: string, url: string }[]; // JSONB
  notes?: string;
}

// From 06_Phase3 - Assembly Progress
// CREATE TYPE assembly_phase_enum AS ENUM (
// 'PreAssembly', 'MainAssembly', 'QualityChecks', 'Testing', 'Packaging', 'ReadyForFinalQC'
// );
enum ProductionPhase {
  PRE_ASSEMBLY = 'PreAssembly',
  MAIN_ASSEMBLY = 'MainAssembly',
  QUALITY_CHECKS = 'QualityChecks',
  TESTING = 'Testing',
  PACKAGING = 'Packaging',
  READY_FOR_FINAL_QC = 'ReadyForFinalQC',
}

// CREATE TYPE assembly_status_enum AS ENUM (
// 'NotStarted', 'InProgress', 'OnHold', 'Completed', 'Failed'
// );
enum ProductionStatus {
  NOT_STARTED = 'NotStarted',
  IN_PROGRESS = 'InProgress',
  ON_HOLD = 'OnHold',
  COMPLETED = 'Completed',
  FAILED = 'Failed',
}

interface AssemblyProgress {
  progressId: string; // UUID, Primary Key from 'assembly_progress' table
  orderId: string;
  buildNumber: string;
  totalTasks: number;
  completedTasks: number;
  failedTasks: number;
  blockedTasks: number;
  estimatedCompletion?: string; // ISO 8601 datetime string
  actualStart?: string;
  actualCompletion?: string;
  currentPhase: ProductionPhase;
  overallStatus: ProductionStatus;
  updatedAt: string;
}

// From 07_Phase4 - QR Codes
// CREATE TYPE qr_type_enum AS ENUM ('Assembly', 'SubAssembly', 'Component', 'Documentation');
enum QRType {
  ASSEMBLY = 'Assembly',
  SUB_ASSEMBLY = 'SubAssembly',
  COMPONENT = 'Component',
  DOCUMENTATION = 'Documentation',
}

interface QRCodeRecord {
  qrId: string; // UUID, Primary Key from 'qr_codes' table
  orderId: string;
  buildNumber: string;
  assemblyId?: string; // If QR is for a specific assembly
  qrData: string; // Encoded data or unique identifier
  qrType: QRType;
  displayUrl: string; // URL the QR code points to or that displays info about this item
  generatedAt: string;
  generatedById: string; // User ID
  isActive: boolean;
  printed: boolean;
  printedAt?: string;
}

interface GenerateQRRequest { // For POST /api/qr/generate/{orderId}
  buildNumber: string;
  qrType: QRType;
  targetEntityId?: string; // e.g., assemblyId if type is Assembly
  // other necessary data to embed or link via qrData/displayUrl
}

// From 07_Phase4 - Audit Log & Reports
// CREATE TYPE audit_event_type_enum AS ENUM (
// 'UserAction', 'SystemAction', 'DataChange', 'SecurityEvent', 'PerformanceEvent'
// );
enum AuditEventType {
  USER_ACTION = 'UserAction',
  SYSTEM_ACTION = 'SystemAction',
  DATA_CHANGE = 'DataChange',
  SECURITY_EVENT = 'SecurityEvent',
  PERFORMANCE_EVENT = 'PerformanceEvent',
}

// CREATE TYPE audit_action_enum AS ENUM (
// 'Create', 'Read', 'Update', 'Delete', 'Login', 'Logout', 'Export', 'Import'
// );
enum AuditAction {
  CREATE = 'Create',
  READ = 'Read',
  UPDATE = 'Update',
  DELETE = 'Delete',
  LOGIN = 'Login',
  LOGOUT = 'Logout',
  EXPORT = 'Export',
  IMPORT = 'Import',
  APPROVE = 'Approve', // Adding more specific actions
  REJECT = 'Reject',
  REVIEW = 'Review',
  START_TASK = 'StartTask',
  COMPLETE_TASK = 'CompleteTask',
  STATUS_CHANGE = 'StatusChange',
  // etc.
}

interface SystemAuditLogEntry {
  auditId: string; // UUID, Primary Key from 'system_audit_log' table
  userId?: string; // User ID, if action performed by a user
  sessionId?: string; // Session ID, if available
  eventType: AuditEventType;
  entityType?: string; // Table or object type affected (e.g., 'Order', 'User', 'Part')
  entityId?: string; // ID of the affected entity
  action: AuditAction; // Or a more descriptive string if not strictly enum
  oldValues?: any; // JSONB - Previous values for Update actions
  newValues?: any; // JSONB - New values for Create/Update actions
  ipAddress?: string; // INET
  userAgent?: string;
  timestamp: string;
  success: boolean;
  errorMessage?: string;
  // additionalContext from 11_Security might be part of newValues or a separate field
}

// CREATE TYPE report_type_enum AS ENUM (
// 'OrderSummary', 'QualityMetrics', 'ProductionAnalytics',
// 'ComplianceReport', 'PerformanceReport', 'CustomReport'
// );
enum ReportType {
  ORDER_SUMMARY = 'OrderSummary',
  QUALITY_METRICS = 'QualityMetrics',
  PRODUCTION_ANALYTICS = 'ProductionAnalytics',
  COMPLIANCE_REPORT = 'ComplianceReport',
  PERFORMANCE_REPORT = 'PerformanceReport',
  CUSTOM_REPORT = 'CustomReport',
}

interface ReportConfiguration {
  configId: string; // UUID, Primary Key from 'report_configurations' table
  name: string;
  description?: string;
  reportType: ReportType;
  configuration: any; // JSONB - Report parameters, filters, columns, grouping etc.
  schedule?: any; // JSONB - e.g., { type: 'daily', time: '02:00' } or cron string
  createdById: string; // User ID
  createdAt: string;
  updatedAt: string;
  isActive: boolean;
  isPublic: boolean;
}

interface GenerateReportPayload { // For POST /api/reports/generate
  reportType?: ReportType; // Can specify type directly
  configId?: string; // Or use a saved configuration
  parameters?: any; // Ad-hoc parameters overriding or supplementing config
  outputFormat?: 'PDF' | 'CSV' | 'JSON'; // Default to PDF or as per config
}

interface GeneratedReportStatus {
  reportId: string;
  status: 'Pending' | 'Processing' | 'Completed' | 'Failed';
  message?: string;
  downloadUrl?: string; // If completed
  createdAt: string;
  completedAt?: string;
}
```

---

## Enumerations

This section consolidates all enumerations (TypeScript string literal union types or enums) used in the API data models for easy reference. Many of these correspond to `ENUM` types in the PostgreSQL database.

```typescript
// User Management
// UserRole is defined in the Authentication & Authorization / RBAC section.
// Example: enum UserRole { Admin = 'Admin', ... }

// Audit Log (Core)
enum AuditEventType {
  USER_ACTION = 'UserAction',
  SYSTEM_ACTION = 'SystemAction',
  DATA_CHANGE = 'DataChange',
  SECURITY_EVENT = 'SecurityEvent',
  PERFORMANCE_EVENT = 'PerformanceEvent',
}

enum AuditAction {
  CREATE = 'Create',
  READ = 'Read',
  UPDATE = 'Update',
  DELETE = 'Delete',
  LOGIN = 'Login',
  LOGOUT = 'Logout',
  EXPORT = 'Export',
  IMPORT = 'Import',
  APPROVE = 'Approve',
  REJECT = 'Reject',
  REVIEW = 'Review',
  START_TASK = 'StartTask',
  COMPLETE_TASK = 'CompleteTask',
  STATUS_CHANGE = 'StatusChange',
}

// DocumentLanguage is used in Order, but also potentially user preferences or system defaults. Keep for now.
enum DocumentLanguage {
  EN = 'EN',
  FR = 'FR',
  SP = 'SP',
}

```

---

## Error Handling

The API uses standard HTTP status codes to indicate the success or failure of a request. Error responses will generally include a JSON body with more specific details.

### Standard Error Response Object
```json
{
  "error": {
    "code": "ERROR_CODE_STRING",
    "message": "A human-readable description of the error.",
    "details": [ // Optional: Array of specific field errors or additional info
      {
        "field": "fieldName",
        "issue": "Description of the issue with this field."
      }
    ],
    "timestamp": "YYYY-MM-DDTHH:mm:ss.sssZ",
    "requestId": "unique-request-identifier" // Optional: for tracing
  }
}
```

### Common HTTP Status Codes
- **200 OK:** Request succeeded.
- **201 Created:** Request succeeded and a new resource was created.
- **204 No Content:** Request succeeded but there is no content to return (e.g., for DELETE or some PUT operations).
- **400 Bad Request:** The request was malformed, contained invalid syntax, or could not be processed (e.g., validation errors).
- **401 Unauthorized:** Authentication is required and has failed or has not yet been provided. This is typically due to missing, invalid, or expired credentials (e.g., access token).
- **403 Forbidden:** The authenticated user does not have the necessary permissions to perform the requested action on the resource.
- **404 Not Found:** The requested resource could not be found on the server.
- **409 Conflict:** The request could not be completed due to a conflict with the current state of the resource (e.g., trying to create a resource that already exists with a unique constraint).
- **422 Unprocessable Entity:** The request was well-formed but was unable to be followed due to semantic errors (often used for validation errors not caught by 400).
- **429 Too Many Requests:** The user has sent too many requests in a given amount of time (rate limiting).
- **500 Internal Server Error:** An unexpected condition was encountered on the server that prevented it from fulfilling the request.
- **503 Service Unavailable:** The server is currently unable to handle the request due to temporary overloading or maintenance.

### Common Application Error Codes (`error.code` strings)
- `VALIDATION_ERROR`: Input validation failed (details array will contain specifics).
- `INVALID_CREDENTIALS`: Supplied username/password is incorrect.
- `MFA_REQUIRED`: Multi-factor authentication is required for this action/user.
- `MFA_INVALID_CODE`: The supplied MFA code is incorrect or expired.
- `ACCOUNT_LOCKED`: Account is locked due to too many failed login attempts.
- `TOKEN_EXPIRED`: Access token has expired (client should use refresh token).
- `INVALID_TOKEN`: Access or refresh token is invalid, malformed, or revoked.
- `PERMISSION_DENIED`: User does not have permission for the action or resource.
- `RESOURCE_NOT_FOUND`: The specific resource (e.g., order, user, part) was not found.
- `RESOURCE_CONFLICT`: Action conflicts with existing state (e.g., unique constraint violation).
- `RATE_LIMITED`: Request was rejected due to rate limiting.
- `OPERATION_FAILED`: A general failure for an operation that doesn't fit other codes.
- `EXTERNAL_SERVICE_ERROR`: An error occurred with a downstream service.
- `NOT_IMPLEMENTED`: The requested functionality is not yet implemented.

(This list will be expanded as specific error scenarios are defined for each endpoint.)

---

## API Endpoint Specifications

This section provides detailed specifications for each API endpoint, including authorization requirements, request/response formats, and example interactions. All endpoints are prefixed with `/api/v1`.

### User Management Endpoints (`/users`)

These endpoints manage user accounts, profiles, and related information.

#### List Users
- **Endpoint:** `GET /users`
- **Description:** Retrieves a paginated list of users. Admin-only, or requires specific `users:read_all` permission.
- **Authorization:** Requires `Admin` role or `users:read_all` permission.
- **Query Parameters:**
    - `page?: number` (Default: 1) - Page number for pagination.
    - `limit?: number` (Default: 20, Max: 100) - Number of users per page.
    - `sortBy?: string` (Default: `createdAt`) - Field to sort by (e.g., `username`, `email`, `role`, `createdAt`).
    - `sortOrder?: 'asc' | 'desc'` (Default: `desc`).
    - `role?: UserRole` - Filter by user role.
    - `isActive?: boolean` - Filter by account status.
    - `search?: string` - Search term for username, email, or full name.
- **Success Response (200 OK):** `PaginatedUserList`
  ```typescript
  interface PaginatedUserList {
    items: UserProfile[]; // Using UserProfile as it's the API representation
    totalItems: number;
    totalPages: number;
    currentPage: number;
    itemsPerPage: number;
  }
  ```
  ```json
  // Example Success Response
  {
    "items": [
      {
        "userId": "550e8400-e29b-41d4-a716-446655440000",
        "username": "johndoe",
        "email": "johndoe@torvan.com",
        "fullName": "John Doe",
        "role": "ProductionCoordinator",
        "isActive": true,
        "mfaEnabled": true,
        "lastLogin": "2024-12-01T10:30:00Z",
        "createdAt": "2023-01-15T09:00:00Z",
        "updatedAt": "2024-11-20T14:00:00Z"
      },
      // ... more users
    ],
    "totalItems": 50,
    "totalPages": 5,
    "currentPage": 1,
    "itemsPerPage": 10
  }
  ```
- **Error Responses:** `401 Unauthorized`, `403 Forbidden`.

#### Create User
- **Endpoint:** `POST /users`
- **Description:** Creates a new user account. Typically Admin-only.
- **Authorization:** Requires `Admin` role or `users:create` permission.
- **Request Body:** `CreateUserPayload`
  ```typescript
  interface CreateUserPayload {
    username: string;
    email: string;
    password: string; // Must meet password policy
    fullName: string;
    initials?: string;
    role: UserRole;
    isActive?: boolean; // Default: true
    department?: string;
    sendWelcomeEmail?: boolean; // Optional: to trigger a welcome email with temp password/setup link
  }
  ```
  ```json
  // Example Request Body
  {
    "username": "newuser",
    "email": "newuser@torvan.com",
    "password": "ComplexPassword123!",
    "fullName": "New User",
    "role": "Assembler",
    "isActive": true,
    "sendWelcomeEmail": true
  }
  ```
- **Success Response (201 Created):** `UserProfile`
  ```json
  // Example Success Response (mirrors UserProfile structure)
  {
    "userId": "123e4567-e89b-12d3-a456-426614174000",
    "username": "newuser",
    "email": "newuser@torvan.com",
    "fullName": "New User",
    "role": "Assembler",
    "isActive": true,
    "mfaEnabled": false, // Typically false on creation, user sets up later
    "createdAt": "2024-12-10T10:00:00Z",
    "updatedAt": "2024-12-10T10:00:00Z"
  }
  ```
- **Error Responses:** `400 Bad Request` (validation errors, e.g., email/username exists, password policy violation), `401 Unauthorized`, `403 Forbidden`.

#### Get User by ID
- **Endpoint:** `GET /users/{userId}`
- **Description:** Retrieves details for a specific user.
- **Authorization:** `Admin` role, or the authenticated user retrieving their own profile (`users:read_self`), or users with `users:read` permission.
- **Path Parameters:**
    - `userId: string` (UUID) - The ID of the user to retrieve.
- **Success Response (200 OK):** `UserProfile`
  ```json
  // Example Success Response (mirrors UserProfile structure)
  {
    "userId": "550e8400-e29b-41d4-a716-446655440000",
    "username": "johndoe",
    "email": "johndoe@torvan.com",
    // ... other UserProfile fields
  }
  ```
- **Error Responses:** `401 Unauthorized`, `403 Forbidden`, `404 Not Found`.

#### Update User by ID
- **Endpoint:** `PUT /users/{userId}`
- **Description:** Updates details for a specific user. Admins can update any user. Regular users might update their own profile with limited fields.
- **Authorization:** `Admin` role (can update all fields, including role, isActive), or the authenticated user updating their own profile (`users:update_self` - limited fields allowed), or users with `users:update`.
- **Path Parameters:**
    - `userId: string` (UUID) - The ID of the user to update.
- **Request Body:** `UpdateUserPayload`
  ```typescript
  interface UpdateUserPayload {
    username?: string;     // Admins might change this
    email?: string;        // Admins might change this
    fullName?: string;
    initials?: string;
    role?: UserRole;       // Admin only
    isActive?: boolean;    // Admin only
    department?: string;
    // Password changes should go through a separate dedicated endpoint (e.g., /users/{userId}/change-password or /auth/reset-password)
    // MFA status changes (enable/disable) should go through dedicated MFA management endpoints.
  }
  ```
  ```json
  // Example Request Body (Admin updating another user)
  {
    "fullName": "John A. Doe",
    "role": "ProductionCoordinator",
    "isActive": false
  }
  // Example Request Body (User updating their own profile)
  {
    "fullName": "Johnathan Doe",
    "initials": "JD"
  }
  ```
- **Success Response (200 OK):** `UserProfile` (the updated user profile)
- **Error Responses:** `400 Bad Request` (validation errors), `401 Unauthorized`, `403 Forbidden`, `404 Not Found`.

#### Delete User by ID
- **Endpoint:** `DELETE /users/{userId}`
- **Description:** Deletes a user account (soft delete or hard delete based on policy). Admin-only.
- **Authorization:** Requires `Admin` role or `users:delete` permission.
- **Path Parameters:**
    - `userId: string` (UUID) - The ID of the user to delete.
- **Success Response (204 No Content):** No response body.
- **Error Responses:** `401 Unauthorized`, `403 Forbidden`, `404 Not Found`.

#### Get User Profile (Self)
- **Endpoint:** `GET /users/profile` (or `GET /users/me` - consider aliasing or choosing one)
- **Description:** Retrieves the profile of the currently authenticated user.
- **Authorization:** Any authenticated user (`users:read_self` or implicit).
- **Success Response (200 OK):** `UserProfile`
- **Error Responses:** `401 Unauthorized`.

#### Update User Profile (Self)
- **Endpoint:** `PUT /users/profile` (or `PUT /users/me`)
- **Description:** Allows the currently authenticated user to update their own profile information (limited fields).
- **Authorization:** Any authenticated user (`users:update_self` or implicit).
- **Request Body:** `UpdateSelfProfilePayload`
  ```typescript
  interface UpdateSelfProfilePayload {
    fullName?: string;
    initials?: string;
    department?: string;
    // Potentially preferences if not a separate endpoint
    // User cannot change their own email, username, role, isActive status here.
  }
  ```
- **Success Response (200 OK):** `UserProfile`
- **Error Responses:** `400 Bad Request`, `401 Unauthorized`.

#### Get User Preferences (Self)
- **Endpoint:** `GET /users/preferences` (or `GET /users/me/preferences`)
- **Description:** Retrieves preferences for the currently authenticated user.
- **Authorization:** Any authenticated user.
- **Success Response (200 OK):** `UserPreferences`
  ```typescript
  interface UserPreferences {
    userId: string;
    theme?: 'light' | 'dark' | 'system';
    language?: DocumentLanguage; // Default UI language
    notifications?: {
      emailEnabled?: boolean;
      inAppEnabled?: boolean;
      subscribedEvents?: NotificationType[];
    };
    dashboardLayouts?: any; // JSON for custom dashboard layouts
    // Add other preferences as needed
  }
  ```
- **Error Responses:** `401 Unauthorized`.

#### Update User Preferences (Self)
- **Endpoint:** `PUT /users/preferences` (or `PUT /users/me/preferences`)
- **Description:** Updates preferences for the currently authenticated user.
- **Authorization:** Any authenticated user.
- **Request Body:** `Partial<UserPreferences>` (allows updating only specific preferences)
- **Success Response (200 OK):** `UserPreferences` (the updated preferences)
- **Error Responses:** `400 Bad Request`, `401 Unauthorized`.

#### Get User Sessions (Self or Admin)
- **Endpoint:** `GET /users/sessions` (for self) or `GET /users/{userId}/sessions` (for Admin)
- **Description:** Retrieves active sessions for the currently authenticated user or a specific user (Admin).
- **Authorization:** Authenticated user for their own sessions; `Admin` or `users:manage_sessions` for other users.
- **Query Parameters (for Admin):** `page`, `limit`.
- **Success Response (200 OK):** `PaginatedSessionList` or `SessionInfo[]`
  ```typescript
  interface PaginatedSessionList {
    items: SessionInfo[];
    totalItems: number;
    // ... pagination fields
  }
  ```
- **Error Responses:** `401 Unauthorized`, `403 Forbidden`, `404 Not Found` (if Admin specifies non-existent userId).

#### Invalidate User Session (Self or Admin)
- **Endpoint:** `DELETE /users/sessions/{sessionId}` (or `POST /users/sessions/{sessionId}/invalidate`)
- **Description:** Invalidates a specific user session (e.g., to log out a device remotely).
- **Authorization:** Authenticated user for their own sessions; `Admin` or `users:manage_sessions` for other users' sessions.
- **Path Parameters:**
    - `sessionId: string` - The ID of the session to invalidate.
- **Success Response (204 No Content):** No response body.
- **Error Responses:** `401 Unauthorized`, `403 Forbidden`, `404 Not Found`.

---

## Authentication & Authorization

This section details the authentication mechanisms, authorization policies, and related data structures for the CleanStation API.

### JWT Token Implementation

The API uses JSON Web Tokens (JWT) for authenticating requests. Tokens are signed using the RS256 algorithm (RSA signature with SHA-256).

#### Token Configuration
- **Algorithm:** RS256
- **Issuer (`iss`):** `cleanstation-auth-service` (as per `11_Security_Compliance_Framework.md`)
- **Audience (`aud`):** `cleanstation-api` (as per `11_Security_Compliance_Framework.md`)
- **Access Token Expiry:** 1 hour (configurable, as per `11_Security_Compliance_Framework.md`)
- **Refresh Token Expiry:** 30 days (configurable, as per `11_Security_Compliance_Framework.md`)
- **Key Rotation:** Keys are rotated every 90 days, or as needed for security.
- **Key Identifier (`kid`):** Included in the JWT header to allow for seamless key rotation.

#### Token Structure

##### JWT Header
```typescript
interface JWTHeader {
  alg: 'RS256';           // Algorithm: RSA signature with SHA-256
  typ: 'JWT';             // Token type
  kid: string;            // Key Identifier for signature verification key
}
```

##### JWT Payload
```typescript
interface JWTPayload {
  // Standard Claims (as per RFC 7519)
  iss: 'cleanstation-auth-service'; // Issuer of the token
  sub: string;                      // Subject (User ID - `user_id` from the users table)
  aud: 'cleanstation-api';          // Audience (intended recipient of the token)
  exp: number;                      // Expiration time (Unix timestamp)
  nbf: number;                      // Not Before time (Unix timestamp)
  iat: number;                      // Issued At time (Unix timestamp)
  jti: string;                      // JWT ID (unique identifier for this token)

  // Custom Claims
  username: string;                 // User's username
  email: string;                    // User's email address
  role: UserRole;                   // Primary user role (enum defined below)
  permissions: Permission[];        // Array of specific permissions granted to the user
  sessionId: string;                // Session identifier (`session_id` from user_sessions table)
  department?: string;               // User's department (if applicable)
  lastLogin: number;                // Timestamp of the user's last login
  mfa_enabled: boolean;             // Indicates if MFA is enabled for the user's account
  mfa_verified_session: boolean;    // Indicates if MFA was verified for the current session
  scope?: string[];                  // OAuth-style scopes (if used for granular access to resources)
  deviceFingerprint?: string;        // Fingerprint of the device used for login
}
```

##### Permission Structure
```typescript
interface Permission {
  resource: string;        // Resource type (e.g., 'orders', 'boms', 'users', 'system:*')
  actions: string[];       // Allowed actions (e.g., ['create', 'read', 'update', 'delete', 'approve', '*'])
  conditions?: object;     // Optional conditions or constraints for the permission (e.g., based on attributes)
}
```

### Password Policy
The system enforces the following password requirements for user accounts:
- **Minimum Length:** 12 characters
- **Complexity:** Must include at least one of each:
    - Uppercase letter (A-Z)
    - Lowercase letter (a-z)
    - Number (0-9)
    - Special character (e.g., !@#$%^&*)
- **Prohibition:** Common passwords and patterns are prohibited.
- **History:** The last 12 passwords cannot be reused.
- **Maximum Age:** Passwords must be changed every 90 days.
- **Account Lockout:** Accounts are temporarily locked after multiple (e.g., 5) failed login attempts.

### Multi-Factor Authentication (MFA)
MFA is implemented to enhance security for user accounts.
- **Status:** Enabled
- **Supported Methods:**
    - TOTP (Time-based One-Time Password) authenticator apps (e.g., Google Authenticator, Authy)
    - SMS (text message codes) - for select roles or as a backup
    - Email codes - for select roles or as a backup
- **Backup Codes:** Users are provided with 10 single-use backup codes upon MFA setup.
- **Enforcement:** MFA is mandatory for users with the following roles:
    - `Admin`
    - `QCPerson`
    - `ProductionCoordinator`
- Users can enable MFA voluntarily for other roles.

### Session Management
User sessions are managed with the following policies:
- **Idle Timeout:** Sessions automatically expire after 2 hours of inactivity. Users will be required to re-authenticate.
- **Absolute Timeout:** Sessions have a maximum duration of 8 hours, after which re-authentication is mandatory, regardless of activity.
- **Concurrent Sessions:** A user can have a maximum of 3 active sessions simultaneously. Attempting to create a new session beyond this limit may invalidate the oldest session or be prevented.
- **Device Tracking:** Device information (User Agent, IP Address) is logged with each session for security monitoring and can be used for device-specific session management.
- **Secure Cookies:** Refresh tokens are stored in HTTP-only, secure cookies to mitigate XSS attacks. Access tokens may be returned in the response body for client-side handling but have a short expiry.

### Role-Based Access Control (RBAC)
The API employs a Role-Based Access Control model to manage user permissions. Each user is assigned a primary role, and that role is associated with a set of permissions.

#### User Roles (`UserRole` enum)
The following user roles are defined in the system (corresponds to `user_role_enum` in the database):
```typescript
enum UserRole {
  Admin = 'Admin',
  ProductionCoordinator = 'ProductionCoordinator',
  ProcurementSpecialist = 'ProcurementSpecialist',
  QCPerson = 'QCPerson',
  Assembler = 'Assembler',
  ServiceDepartment = 'ServiceDepartment'
}
```

#### Role Permissions
Permissions define what actions a user can perform on specific resources. The general format for a permission string is `resource:action` (e.g., `orders:create`, `users:read`). A wildcard `*` can be used to denote all actions or all resources (e.g., `users:*` or `*:read`).

- **Admin:**
    - `users:*` (manage all users)
    - `roles:*` (manage roles and permissions - if applicable as a separate resource)
    - `catalog:*` (manage all catalog items: parts, assemblies, categories, accessories, tools)
    - `orders:*` (full control over all orders)
    - `boms:*` (full control over all BOMs)
    - `qc:*` (full control over all QC processes, templates, and results)
    - `production:*` (full control over production workflows, tasks, checklists)
    - `testing:*` (full control over testing procedures and results)
    - `packaging:*` (full control over packaging)
    - `service:*` (full control over service requests and parts)
    - `documents:*` (manage all documents)
    - `notifications:send_all` (system-wide notifications)
    - `system:*` (manage system settings, health, audit logs, reports, exports)
    - `audit:read` (access to all audit logs)
    - `reports:*` (manage and generate all reports)
    - `export:*` (perform all data exports)
    - `qr:*` (manage QR codes)

- **ProductionCoordinator:**
    - `orders:create`, `orders:read`, `orders:update` (manage orders they are responsible for or all, TBD by policy)
    - `orders:status_change` (specific status transitions)
    - `boms:generate`, `boms:read` (generate and view BOMs for their orders)
    - `users:read` (view user information, limited scope)
    - `parts:read`, `assemblies:read`, `categories:read`, `accessories:read` (view catalog items)
    - `qc:read` (view QC status and results for their orders)
    - `production:read` (view production progress for their orders)
    - `documents:upload`, `documents:read` (for their orders)
    - `notifications:read`, `notifications:send` (related to their scope)
    - `reports:generate` (for their assigned reports)
    - `qr:generate` (for their orders)

- **ProcurementSpecialist:**
    - `boms:read`, `boms:review`, `boms:approve`, `boms:reject`, `boms:modify`
    - `orders:read` (view order details relevant to procurement)
    - `parts:read`, `parts:update_sourcing_info` (manage sourcing info for parts)
    - `assemblies:read`
    - `outsourcing:*` (manage parts outsourcing process)
    - `vendors:*` (manage vendor information - if a separate resource)
    - `notifications:read`, `notifications:send`

- **QCPerson:**
    - `qc:create`, `qc:read`, `qc:submit`, `qc:update` (perform QC checks and manage QC forms - PreQC, InProcess, FinalQC)
    - `qc_templates:read`, `qc_templates:use`
    - `orders:read` (view order details relevant to QC)
    - `orders:update_status` (e.g., to 'ReadyForFinalQC', 'Pass', 'Fail')
    - `parts:read` (view part details)
    - `assemblies:read` (view assembly details)
    - `documents:read`, `documents:verify` (related to QC documentation)
    - `testing_results:read`, `testing_results:verify`
    - `notifications:read`

- **Assembler:**
    - `assembly_tasks:read_assigned`, `assembly_tasks:read_available`
    - `assembly_tasks:start`, `assembly_tasks:complete`, `assembly_tasks:update_status` (manage their assigned assembly tasks)
    - `production_checklists:complete` (fill out their assigned checklists)
    - `work_instructions:read`
    - `parts:read`, `tools:read` (view details of parts and tools needed for tasks)
    - `testing_procedures:execute` (perform tests as part of assembly)
    - `packaging:complete` (perform packaging tasks)
    - `notifications:read`

- **ServiceDepartment:**
    - `service_requests:*` (manage service part requests)
    - `service_parts_catalog:read`
    - `customers:read` (view customer information for service requests - if a separate resource)
    - `orders:read` (lookup past order details for service context)
    - `parts:read`
    - `notifications:read`, `notifications:send`

This list is a guideline and the exact granularity will be enforced by the backend. Permissions are additive; a user possesses all permissions granted to their role.

### Authentication Flow

#### 1. User Login
The user provides credentials (email/username and password). If MFA is enabled and required for the role or user, an MFA code is also required.
- **Endpoint:** `POST /api/v1/auth/login`
- **Request:** `AuthenticationRequest`
- **Response:** `AuthenticationResponse` (contains access and refresh tokens, user profile, permissions, and session info)

#### 2. API Requests
For subsequent requests to protected endpoints, the client must include the `accessToken` in the `Authorization` header using the Bearer scheme:
`Authorization: Bearer <accessToken>`

#### 3. Token Expiration & Refresh
- If the `accessToken` is expired, the API will respond with a `401 Unauthorized` error.
- The client can then use the `refreshToken` (obtained during login) to request a new `accessToken` and `refreshToken`.
- **Endpoint:** `POST /api/v1/auth/refresh`
- **Request:** `TokenRefreshRequest`
- **Response:** `TokenRefreshResponse`

#### 4. Logout
- To log out, the client should discard the tokens and call the logout endpoint to invalidate the session server-side.
- **Endpoint:** `POST /api/v1/auth/logout`
- **Request:** (Optionally, the refreshToken to invalidate it specifically)
- **Response:** `204 No Content` or success message.

### Data Models for Authentication

```typescript
interface UserProfile {
  userId: string;                      // UUID, Primary Key
  username: string;                    // Unique username
  email: string;                       // Unique email address
  fullName: string;
  initials?: string;                   // For QC/Assembly sign-offs
  role: UserRole;                      // User's primary role
  isActive: boolean;                   // Account status
  department?: string;                  // User's department
  mfaEnabled: boolean;                 // True if user has MFA configured
  lastLogin?: string;                  // ISO 8601 datetime string
  createdAt: string;                   // ISO 8601 datetime string
  updatedAt: string;                   // ISO 8601 datetime string
  // Include other relevant fields from the 'users' table as needed
  // e.g., preferences, profile picture URL, etc.
}

interface AuthenticationRequest {
  email?: string;          // User's email address (either email or username is required)
  username?: string;       // User's username (either email or username is required)
  password: string;        // User's password
  remember?: boolean;      // Request extended session for refresh token (e.g., 30 days vs. session-only)
  mfaCode?: string;        // Multi-factor authentication code, if MFA is enabled and required
  deviceFingerprint?: string; // Optional device identifier for tracking and security
}

interface AuthenticationResponse {
  accessToken: string;     // JWT access token (e.g., 1 hour expiry)
  refreshToken: string;    // JWT refresh token (e.g., 30 days expiry, stored in secure HttpOnly cookie by client if possible)
  expiresIn: number;       // Access token expiration in seconds (e.g., 3600 for 1 hour)
  tokenType: 'Bearer';     // Token type
  user: UserProfile;       // Detailed user profile information
  permissions: Permission[]; // Effective permissions for the user
  sessionInfo: SessionInfo;  // Metadata about the created session
}

interface SessionInfo {
  sessionId: string;        // Unique session identifier (from `user_sessions` table)
  userId: string;           // User ID associated with the session
  createdAt: string;        // ISO 8601 datetime string, when the session was created
  expiresAt: string;        // ISO 8601 datetime string, when the session will absolutely expire
  idleExpiresAt: string;    // ISO 8601 datetime string, when the session will expire due to inactivity
  ipAddress?: string;       // IP address from which the session originated
  userAgent?: string;       // User agent of the client
  deviceInfo?: {            // Optional, more detailed device info if captured
    type?: string;        // e.g., 'desktop', 'mobile', 'tablet'
    os?: string;          // e.g., 'Windows 10', 'macOS', 'iOS', 'Android'
    browser?: string;     // e.g., 'Chrome 120', 'Firefox', 'Safari'
    isTrusted?: boolean;  // If the device is marked as trusted
  };
  mfaVerified: boolean;     // Indicates if MFA was successfully verified for this session
}

interface TokenRefreshRequest {
  refreshToken: string;
}

interface TokenRefreshResponse {
  accessToken: string;
  refreshToken: string; // Optionally, a new refresh token might be issued (e.g., for refresh token rotation)
  expiresIn: number;
  tokenType: 'Bearer';
}
```

### Authentication Endpoints

#### Login Endpoint
- **Endpoint:** `POST /api/v1/auth/login`
- **Description:** Authenticates a user and returns access/refresh tokens along with user and session information.
- **Request Body:** `AuthenticationRequest`
  ```json
  // Example Request
  {
    "email": "user@torvan.com",
    "password": "SecurePassword123!",
    "remember": false,
    "deviceFingerprint": "abc123def456xyz"
  }
  ```
- **Success Response (200 OK):** `AuthenticationResponse`
  ```json
  // Example Success Response
  {
    "accessToken": "eyJhbGciOiJSUzI1NiIs...",
    "refreshToken": "def456ghi789jkl...", // Note: This might be set as an HttpOnly cookie instead of in body
    "expiresIn": 3600, // e.g., 1 hour
    "tokenType": "Bearer",
    "user": {
      "userId": "550e8400-e29b-41d4-a716-446655440000",
      "username": "johndoe",
      "email": "user@torvan.com",
      "fullName": "John Doe",
      "role": "ProductionCoordinator",
      "isActive": true,
      "mfaEnabled": true,
      "lastLogin": "2024-12-01T10:30:00Z",
      "createdAt": "2023-01-15T09:00:00Z",
      "updatedAt": "2024-11-20T14:00:00Z"
    },
    "permissions": [
      { "resource": "orders", "actions": ["create", "read", "update"] },
      { "resource": "boms", "actions": ["generate", "read"] }
    ],
    "sessionInfo": {
      "sessionId": "sess_abc123xyz789",
      "userId": "550e8400-e29b-41d4-a716-446655440000",
      "createdAt": "2024-12-01T10:30:00Z",
      "expiresAt": "2024-12-01T18:30:00Z", // 8-hour absolute timeout
      "idleExpiresAt": "2024-12-01T12:30:00Z", // 2-hour idle timeout
      "ipAddress": "192.168.1.100",
      "userAgent": "Chrome/120.0.0.0",
      "mfaVerified": true
    }
  }
  ```
- **Error Responses:**
    - `400 Bad Request`: Invalid request format (e.g., missing fields, invalid email).
      ```json
      {
        "error": {
          "code": "VALIDATION_ERROR",
          "message": "Email and password are required.",
          "details": [{"field": "email", "issue": "cannot be empty"}]
        }
      }
      ```
    - `401 Unauthorized`: Invalid credentials (wrong email/password), or MFA required but not provided/invalid.
      ```json
      // Invalid Credentials
      {
        "error": {
          "code": "INVALID_CREDENTIALS",
          "message": "Invalid email or password."
        }
      }
      // MFA Required
      {
        "error": {
          "code": "MFA_REQUIRED",
          "message": "Multi-factor authentication is required for this account."
        }
      }
      // MFA Code Invalid
      {
        "error": {
          "code": "MFA_INVALID_CODE",
          "message": "The MFA code provided is invalid or expired."
        }
      }
      ```
    - `429 Too Many Requests`: Account locked due to too many failed login attempts.
      ```json
      {
        "error": {
          "code": "ACCOUNT_LOCKED",
          "message": "Account locked due to too many failed login attempts. Try again in 15 minutes or reset your password.",
          "retryAfter": 900 // seconds
        }
      }
      ```

#### Token Refresh Endpoint
- **Endpoint:** `POST /api/v1/auth/refresh`
- **Description:** Obtains a new access token (and potentially a new refresh token) using a valid refresh token.
- **Request Body:** `TokenRefreshRequest`
  ```json
  // Example Request
  {
    "refreshToken": "def456ghi789jkl..." // Typically sent via HttpOnly cookie, or in body if not possible
  }
  ```
- **Success Response (200 OK):** `TokenRefreshResponse`
  ```json
  // Example Success Response
  {
    "accessToken": "eyJhbGciOiJSUzI1NiIs...",
    "refreshToken": "newRefreshTokenIfRotated...", // Only if refresh token rotation is enabled
    "expiresIn": 3600,
    "tokenType": "Bearer"
  }
  ```
- **Error Responses:**
    - `401 Unauthorized`: Invalid or expired refresh token.
      ```json
      {
        "error": {
          "code": "INVALID_REFRESH_TOKEN",
          "message": "Refresh token is invalid or expired. Please log in again."
        }
      }
      ```

#### Logout Endpoint
- **Endpoint:** `POST /api/v1/auth/logout`
- **Description:** Invalidates the user's current session (identified by the refresh token or access token).
- **Request Body:** (Optional)
  ```json
  // Optional: Can send refresh token to ensure specific session invalidation
  {
    "refreshToken": "def456ghi789jkl..."
  }
  ```
- **Success Response (204 No Content or 200 OK):**
  ```json
  // Example 200 OK Response (if a body is returned)
  {
    "message": "Logout successful."
  }
  ```
- **Error Responses:**
    - `401 Unauthorized`: If no valid session/token is provided to identify the session to logout.

#### Get Session Info Endpoint
- **Endpoint:** `GET /api/v1/auth/session`
- **Description:** Retrieves information about the current authenticated user's session. Requires a valid access token.
- **Request Body:** None
- **Success Response (200 OK):** `AuthenticationResponse` (or a subset focusing on `user`, `permissions`, and `sessionInfo`)
  ```json
  // Example: Similar to AuthenticationResponse but without new tokens
  {
    "user": { /* ... UserProfile ... */ },
    "permissions": [ /* ... Permission[] ... */ ],
    "sessionInfo": { /* ... SessionInfo ... */ }
  }
  ```
- **Error Responses:**
    - `401 Unauthorized`: If no valid access token is provided.

#### Password Reset Endpoints
Details for password reset (requesting a token, submitting new password) would typically involve:
1.  `POST /api/v1/auth/request-password-reset`
    - Request: `{ "email": "user@example.com" }`
    - Action: Generates a secure, time-limited token and sends it to the user's email.
2.  `POST /api/v1/auth/reset-password`
    - Request: `{ "resetToken": "someSecureToken", "newPassword": "NewSecurePassword123!" }`
    - Action: Validates the token and updates the user's password if the token is valid and the new password meets policy.

(Detailed specification for these can be added if this initial structure is approved).

#### Token Verification Endpoint
- **Endpoint:** `GET /api/v1/auth/verify` (or `POST` if token is sent in body)
- **Description:** Allows a client or another service to verify the validity of an access token.
- **Request:** `Authorization: Bearer <accessToken>` (if GET) or `{ "token": "<accessToken>" }` (if POST)
- **Success Response (200 OK):**
  ```json
  {
    "active": true,
    "payload": { /* ... Decoded JWTPayload ... */ }
    // or simply a 200 OK for active, 401 for inactive/invalid
  }
  ```
- **Error Responses:**
    - `401 Unauthorized`: Token is invalid, expired, or malformed.

---

## Security Framework

### Transport Security
```typescript
interface TransportSecurity {
  encryption: 'TLS 1.3 minimum';
  certificates: 'Extended Validation SSL certificates';
  hsts: 'Strict-Transport-Security header with 1 year max-age';
  certificatePinning: 'HTTP Public Key Pinning for mobile apps';
  ocspStapling: 'Online Certificate Status Protocol stapling';
}
```

### API Security Headers
```typescript
interface SecurityHeaders {
  'Strict-Transport-Security': 'max-age=31536000; includeSubDomains; preload';
  'Content-Security-Policy': "default-src 'self'; script-src 'self' 'unsafe-inline'";
  'X-Content-Type-Options': 'nosniff';
  'X-Frame-Options': 'DENY';
  'X-XSS-Protection': '1; mode=block';
  'Referrer-Policy': 'strict-origin-when-cross-origin';
  'Permissions-Policy': 'camera=(), microphone=(), geolocation=()';
}
```

### Input Validation & Sanitization
```typescript
interface ValidationRules {
  email: {
    pattern: /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    maxLength: 254;
    sanitize: 'trim, lowercase';
  };
  
  password: {
    minLength: 12;
    pattern: /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]/;
    sanitize: 'none';  // Never log or expose passwords
  };
  
  poNumber: {
    pattern: /^[A-Z0-9\-_]{1,50}$/;
    sanitize: 'trim, uppercase';
  };
  
  buildNumber: {
    pattern: /^[A-Z0-9\-]{1,20}$/;
    sanitize: 'trim, uppercase';
  };
  
  notes: {
    maxLength: 2000;
    sanitize: 'trim, html_escape';
    allowedTags: [];  // No HTML tags allowed
  };
}
```

### Rate Limiting Strategy
```typescript
interface RateLimitConfiguration {
  global: {
    windowMs: 900000;      // 15 minutes
    max: 1000;             // Limit each IP to 1000 requests per windowMs
    message: 'Too many requests from this IP';
    standardHeaders: true;  // Return rate limit info in headers
    legacyHeaders: false;
  };
  
  authenticated: {
    windowMs: 3600000;     // 1 hour
    max: 5000;             // 5000 requests per hour for authenticated users
    keyGenerator: (req) => req.user.userId;
  };
  
  admin: {
    windowMs: 3600000;     // 1 hour
    max: 10000;            // 10000 requests per hour for admins
    keyGenerator: (req) => req.user.userId;
  };
  
  login: {
    windowMs: 900000;      // 15 minutes
    max: 5;                // Limit login attempts
    keyGenerator: (req) => req.ip + ':' + req.body.email;
    skipSuccessfulRequests: true;
  };
  
  passwordReset: {
    windowMs: 3600000;     // 1 hour
    max: 3;                // 3 password reset attempts per hour
    keyGenerator: (req) => req.body.email;
  };
}
```

---

## Core Data Models

### User Management Models
```typescript
interface User {
  userId: string;          // UUID
  email: string;           // Unique email address
  fullName: string;        // Full display name
  role: UserRole;          // Primary role
  department: string;      // Department/division
  isActive: boolean;       // Account status
  createdAt: Date;         // Account creation
  updatedAt: Date;         // Last profile update
  lastLogin?: Date;        // Last successful login
  emailVerified: boolean;  // Email verification status
  mfaEnabled: boolean;     // Multi-factor authentication
  preferences: UserPreferences;
  metadata: UserMetadata;
}

interface UserPreferences {
  language: 'en' | 'es' | 'fr';
  timezone: string;        // IANA timezone identifier
  dateFormat: 'MM/DD/YYYY' | 'DD/MM/YYYY' | 'YYYY-MM-DD';
  timeFormat: '12h' | '24h';
  notifications: NotificationPreferences;
  dashboard: DashboardPreferences;
}

interface NotificationPreferences {
  email: {
    orderUpdates: boolean;
    qcAlerts: boolean;
    systemMaintenance: boolean;
    weeklyReports: boolean;
  };
  inApp: {
    taskAssignments: boolean;
    approvalRequests: boolean;
    urgentAlerts: boolean;
    mentions: boolean;
  };
  frequency: 'immediate' | 'hourly' | 'daily';
}

interface UserMetadata {
  employeeId?: string;     // Employee identifier
  phoneNumber?: string;    // Contact number
  workLocation: string;    // Physical work location
  supervisor?: string;     // Supervisor user ID
  skills: string[];        // User skills/certifications
  trainingCompleted: string[]; // Completed training modules
  lastPasswordChange: Date;
  loginHistory: LoginHistoryEntry[];
}

interface LoginHistoryEntry {
  sessionId: string;
  loginTime: Date;
  logoutTime?: Date;
  ipAddress: string;
  userAgent: string;
  deviceFingerprint?: string;
  location?: GeoLocation;
}

#### Submit BOM Review Action (Approve/Reject/Modify)
- **Endpoint:** `POST /boms/{bomId}/review` (This could also be PUT actions on specific sub-resources like `/approve`, `/reject` as listed in the initial structure. Consolidating into one for now, or can be split.)
- **Description:** Allows a reviewer (e.g., Procurement Specialist) to submit their review decision for a BOM (Approve, Reject, or Request Modification).
- **Authorization:** Requires roles like `ProcurementSpecialist` or `Admin` with `boms:review_submit` permission.
- **Path Parameters:**
    - `bomId: string` (UUID) - The ID of the BOM being reviewed.
- **Request Body:** `BOMReviewActionPayload` (Derived from `BOMReviewRequest` in Data Models)
  ```typescript
  interface BOMReviewActionPayload {
    action: BOMReviewStatus.APPROVED | BOMReviewStatus.REJECTED | BOMReviewStatus.REQUIRES_MODIFICATION;
    notes?: string; // Required for REJECTED or REQUIRES_MODIFICATION
    modificationsSuggested?: any; // JSON defining suggested changes if action is REQUIRES_MODIFICATION
                                // (e.g., [{ itemId: string, field: 'quantity', newValue: 10, reason: 'Stock issue' }])
  }
  ```
  ```json
  // Example Request Body (Approve)
  {
    "action": "Approved",
    "notes": "All parts verified and in stock."
  }
  // Example Request Body (Reject)
  {
    "action": "Rejected",
    "notes": "Critical component XYZ is discontinued and no alternative found."
  }
  // Example Request Body (Requires Modification)
  {
    "action": "RequiresModification",
    "notes": "Part ABC quantity needs adjustment based on current project scope. See attached suggestions.",
    "modificationsSuggested": [ { "itemId": "part_uuid_123", "field": "quantity", "newValue": 8, "reason": "Scope changed" } ]
  }
  ```
- **Success Response (200 OK):** `BOMStructure` (The BOM with its updated status and review details, or `BOMReview_DB` model from Data Models showing the review record)
  ```json
  // Example Success Response (Returning the BOMReview_DB record)
  {
    "reviewId": "review_uuid_abc",
    "bomId": "bom_uuid_xyz",
    "reviewerId": "user_uuid_procure",
    "reviewStatus": "Approved", // or Rejected, RequiresModification
    "reviewNotes": "All parts verified and in stock.",
    "reviewedAt": "2024-11-06T14:00:00Z"
    // modifications field would be populated if status was RequiresModification
  }
  ```
- **Error Responses:** `400 Bad Request` (invalid action or missing notes for rejection/modification), `401 Unauthorized`, `403 Forbidden`, `404 Not Found` (BOM not found), `409 Conflict` (BOM not in a state that allows review).

_The other specific PUT endpoints like `/boms/{bomId}/approve`, `/reject`, `/modify` from the initial list can be implemented as convenience wrappers or alternatives to the consolidated `/review` endpoint if desired. Their request bodies would be simpler, often just notes._