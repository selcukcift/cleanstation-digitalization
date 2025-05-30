# Phase 2: Procurement & Pre-QC Workflow
## Torvan Medical CleanStation Production Workflow Digitalization

**Version:** 1.0  
**Date:** December 2024  
**Document Type:** Phase Implementation Specification  
**Timeline:** Weeks 9-14 (6 weeks)  
**Dependencies:** Phase 1 Completion, Client QC Clarifications

---

## Executive Summary

Phase 2 implements the procurement workflow and digital Pre-QC processes, bridging the gap between order creation and production. This phase enables Procurement Specialists to review and approve BOMs, manage parts outsourcing, and provides QC Personnel with digital tools for Pre-QC inspections based on the CLP.T2.001.V01 checklist standards.

### Key Deliverables
- **Procurement Specialist Dashboard** - BOM review, approval, and outsourcing management
- **Digital Pre-QC Forms** - CLP.T2.001.V01 Section 1 implementation
- **BOM Review & Approval System** - Multi-stage approval workflow
- **Parts Outsourcing Management** - Tracking and status management
- **QC Person Dashboard** - Pre-QC queue and form management
- **Document Management System** - Upload, association, and verification
- **Status Tracking & Notifications** - Automated workflow progression

### Success Criteria
- Procurement specialists can review and approve BOMs efficiently
- QC personnel can perform digital Pre-QC checks with full compliance
- Documents are properly associated and verified with orders
- Status updates flow correctly through the procurement and QC pipeline
- System maintains full audit trail for ISO 13485:2016 compliance

---

## Scope & Objectives

### In Scope
- **BOM Review Workflow** - Complete approval and modification process
- **Pre-QC Digital Forms** - Based on CLP.T2.001.V01 Section 1
- **Parts Outsourcing** - Identification, tracking, and status management
- **Document Management** - Upload, verification, and association
- **QC Dashboard** - Task management and form completion
- **Status Notifications** - Automated updates and alerts
- **Audit Logging** - Complete traceability for compliance

### Out of Scope (Deferred to Later Phases)
- Assembly guidance and production checklists
- Final QC workflows
- Service department functionality
- Advanced reporting beyond basic audit trails
- Integration with external procurement systems

### Objectives
1. **Streamline BOM Approval** - Efficient review and approval process
2. **Digitize Pre-QC** - Replace manual checklists with digital forms
3. **Enable Outsourcing Management** - Track external processing requirements
4. **Ensure Compliance** - Maintain ISO 13485:2016 audit trails
5. **Improve Communication** - Clear status updates and notifications
6. **Reduce Errors** - Validation and verification at each step

---

## Technical Requirements

### Database Schema Extensions

```sql
-- Procurement-specific tables
CREATE TABLE bom_reviews (
    review_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    bom_id UUID NOT NULL REFERENCES boms(bom_id),
    reviewer_id UUID NOT NULL REFERENCES users(user_id),
    review_status bom_review_status_enum NOT NULL,
    review_notes TEXT,
    modifications JSONB, -- Changes made to BOM
    reviewed_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    approved_at TIMESTAMP WITH TIME ZONE,
    rejected_at TIMESTAMP WITH TIME ZONE
);

CREATE TYPE bom_review_status_enum AS ENUM (
    'Pending',
    'InReview', 
    'Approved',
    'Rejected',
    'RequiresModification'
);

-- Parts outsourcing management
CREATE TABLE outsourced_parts (
    outsourcing_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    bom_id UUID NOT NULL REFERENCES boms(bom_id),
    part_id VARCHAR(50) NOT NULL REFERENCES parts(part_id),
    assembly_id VARCHAR(50) REFERENCES assemblies(assembly_id),
    quantity INTEGER NOT NULL,
    vendor_name VARCHAR(255),
    expected_delivery DATE,
    status outsourcing_status_enum DEFAULT 'Identified',
    tracking_number VARCHAR(100),
    notes TEXT,
    created_by UUID NOT NULL REFERENCES users(user_id),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE TYPE outsourcing_status_enum AS ENUM (
    'Identified',
    'Quoted',
    'Ordered',
    'InProduction',
    'Shipped',
    'Received',
    'QCFailed',
    'Completed'
);

-- Pre-QC form completions
CREATE TABLE preqc_completions (
    completion_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    order_id UUID NOT NULL REFERENCES orders(order_id),
    qc_template_id UUID NOT NULL REFERENCES qc_form_templates(template_id),
    performed_by UUID NOT NULL REFERENCES users(user_id),
    job_id VARCHAR(100),
    number_of_basins INTEGER,
    overall_result preqc_result_enum NOT NULL,
    checklist_results JSONB NOT NULL, -- Individual item results
    digital_signature TEXT NOT NULL,
    attachments JSONB, -- Associated documents/photos
    completed_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    notes TEXT
);

CREATE TYPE preqc_result_enum AS ENUM ('Pass', 'Fail', 'ConditionalPass');

-- Notification system
CREATE TABLE notifications (
    notification_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(user_id),
    type notification_type_enum NOT NULL,
    title VARCHAR(255) NOT NULL,
    message TEXT NOT NULL,
    related_order_id UUID REFERENCES orders(order_id),
    related_bom_id UUID REFERENCES boms(bom_id),
    is_read BOOLEAN DEFAULT false,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    expires_at TIMESTAMP WITH TIME ZONE
);

CREATE TYPE notification_type_enum AS ENUM (
    'BOMReviewRequired',
    'BOMApproved',
    'BOMRejected', 
    'PreQCRequired',
    'PreQCCompleted',
    'OrderStatusChanged',
    'DocumentRequired',
    'OutsourcingUpdate'
);
```

### API Endpoints

```typescript
// BOM Review Endpoints
GET    /api/boms/pending-review        // Get BOMs awaiting review
POST   /api/boms/[bomId]/review        // Submit BOM review
PUT    /api/boms/[bomId]/approve       // Approve BOM
PUT    /api/boms/[bomId]/reject        // Reject BOM
PUT    /api/boms/[bomId]/modify        // Request modifications

// Outsourcing Management
GET    /api/outsourcing/[bomId]        // Get outsourcing for BOM
POST   /api/outsourcing/identify       // Identify parts for outsourcing
PUT    /api/outsourcing/[id]/status    // Update outsourcing status
GET    /api/outsourcing/dashboard      // Outsourcing dashboard data

// Pre-QC Management
GET    /api/preqc/templates            // Get Pre-QC form templates
GET    /api/preqc/pending              // Get orders pending Pre-QC
POST   /api/preqc/[orderId]/complete   // Complete Pre-QC form
GET    /api/preqc/[orderId]/results    // Get Pre-QC results

// Document Management
POST   /api/documents/upload           // Upload documents
GET    /api/documents/[orderId]        // Get order documents
POST   /api/documents/verify           // Verify document approval
DELETE /api/documents/[documentId]     // Delete document

// Notifications
GET    /api/notifications              // Get user notifications
PUT    /api/notifications/[id]/read    // Mark notification as read
POST   /api/notifications/send         // Send notification (system)
```

---

## User Stories & Features

### 1. Procurement Specialist Workflows

#### UC 1.1: BOM Review Dashboard
```typescript
interface BOMReviewDashboard {
  pendingReviews: BOMReviewItem[];
  inProgressReviews: BOMReviewItem[];
  completedReviews: BOMReviewItem[];
  outsourcingRequests: OutsourcingItem[];
}

interface BOMReviewItem {
  bomId: string;
  orderId: string;
  poNumber: string;
  customerName: string;
  totalParts: number;
  totalAssemblies: number;
  priority: 'High' | 'Medium' | 'Low';
  submittedAt: Date;
  dueDate: Date;
  complexity: 'Simple' | 'Moderate' | 'Complex';
}
```

**UI Components:**
- Priority-based task queue with color coding
- BOM summary cards with key metrics
- Quick action buttons (Approve, Reject, Modify)
- Filter and search capabilities
- Workload indicator and capacity planning

**Features:**
- Auto-prioritization based on want date and complexity
- Batch review capabilities for similar orders
- Review time tracking and performance metrics
- Escalation alerts for overdue reviews

#### UC 1.2: Detailed BOM Review Interface
```typescript
interface BOMReviewInterface {
  bomDetails: BOMStructure;
  reviewChecklist: ReviewChecklistItem[];
  previousReviews: BOMReview[];
  suggestedModifications: Modification[];
  outsourcingRecommendations: OutsourcingRecommendation[];
}

interface ReviewChecklistItem {
  id: string;
  category: string;
  description: string;
  status: 'NotReviewed' | 'Pass' | 'Fail' | 'Question';
  notes?: string;
  severity: 'Critical' | 'Major' | 'Minor';
}
```

**UI Components:**
- Hierarchical BOM viewer with expand/collapse
- Side-by-side comparison with previous versions
- Interactive checklist with status indicators
- Annotation tools for notes and modifications
- Real-time collaboration features

**Business Logic:**
- Automated validation checks (part availability, quantities)
- Comparison with similar approved BOMs
- Cost impact analysis for modifications
- Lead time calculations including outsourcing

#### UC 1.3: Parts Outsourcing Management
```typescript
interface OutsourcingManagement {
  identificationEngine: OutsourcingIdentifier;
  vendorManagement: VendorManager;
  trackingSystem: OutsourcingTracker;
  deliveryCoordination: DeliveryCoordinator;
}

interface OutsourcingItem {
  id: string;
  partId: string;
  partName: string;
  quantity: number;
  estimatedCost?: number;
  preferredVendor?: string;
  leadTime: number; // days
  criticality: 'Critical' | 'Important' | 'Standard';
  reason: string; // Why outsourcing is needed
}
```

**Features:**
- Smart outsourcing identification based on part characteristics
- Vendor recommendation engine
- Automated quote request generation
- Delivery tracking integration
- Cost and timeline impact analysis

### 2. Quality Control Workflows

#### UC 2.1: QC Person Dashboard
```typescript
interface QCDashboard {
  pendingPreQC: PreQCTask[];
  inProgressTasks: QCTask[];
  completedTasks: QCTask[];
  upcomingSchedule: QCScheduleItem[];
  performanceMetrics: QCMetrics;
}

interface PreQCTask {
  orderId: string;
  poNumber: string;
  buildNumbers: string[];
  sinkType: string;
  arrivalDate: Date;
  priority: 'Urgent' | 'High' | 'Normal';
  requiredDocuments: string[];
  assignedTo?: string;
}
```

**UI Components:**
- Task queue with priority indicators
- Quick assignment and claim functionality
- Progress tracking for multi-step inspections
- Performance dashboard with KPIs
- Calendar integration for scheduling

#### UC 2.2: Digital Pre-QC Forms (CLP.T2.001.V01 Section 1)
```typescript
interface PreQCForm {
  // Based on CLP.T2.001.V01 Section 1: Pre-Production Check
  jobId: string;
  numberOfBasins: number;
  performerInitials: string;
  
  dimensionChecks: {
    finalSinkDimensions: DimensionCheck;
    basinDimensions: DimensionCheck[];
    pegboardInstallation?: PegboardCheck;
    pegboardDimensions?: DimensionCheck;
  };
  
  documentVerification: {
    finalApprovedDrawing: DocumentCheck;
    paperworkAttached: DocumentCheck;
  };
  
  mountingVerification: {
    sinkFaucetHoles: LocationCheck[];
    mountingHoles: LocationCheck[];
    feetType: ComponentCheck;
  };
  
  basinSpecificChecks: BasinCheck[];
  
  overallResult: 'Pass' | 'Fail' | 'ConditionalPass';
  notes: string;
  digitalSignature: string;
}

interface DimensionCheck {
  required: { width: number; length: number; depth?: number };
  actual: { width: number; length: number; depth?: number };
  tolerance: number;
  status: 'Pass' | 'Fail' | 'Within Tolerance';
  notes?: string;
}

interface BasinCheck {
  basinIndex: number;
  basinType: 'E-Sink' | 'E-Sink DI' | 'E-Drain';
  bottomFillHole: ComponentCheck;
  drainButton?: ComponentCheck;
  basinLight?: ComponentCheck;
  drainLocation: LocationCheck;
  faucetLocation: LocationCheck;
}
```

**UI Components:**
- Progressive form with section-based navigation
- Real-time validation and calculation
- Photo capture integration for documentation
- Digital signature pad
- Offline capability for shop floor use

**Features:**
- Auto-population from order configuration
- Measurement tools integration (if available)
- Pass/fail status calculation
- Automatic report generation
- Integration with order status updates

#### UC 2.3: Document Verification System
```typescript
interface DocumentVerification {
  requiredDocuments: RequiredDocument[];
  uploadedDocuments: UploadedDocument[];
  verificationResults: VerificationResult[];
  approvalWorkflow: ApprovalStep[];
}

interface RequiredDocument {
  type: 'TechnicalDrawing' | 'PO' | 'QCCertificate' | 'ManufacturingSpec';
  name: string;
  mandatory: boolean;
  version?: string;
  approvalRequired: boolean;
}

interface VerificationResult {
  documentId: string;
  verifiedBy: string;
  verificationStatus: 'Approved' | 'Rejected' | 'RequiresUpdate';
  verificationDate: Date;
  notes?: string;
}
```

**Features:**
- Document checklist based on order type
- Version control and approval tracking
- Digital signature verification
- Automated notifications for missing documents
- Integration with external document systems

### 3. Status Management & Notifications

#### UC 3.1: Automated Status Progression
```typescript
interface StatusManager {
  validateTransition(currentStatus: OrderStatus, newStatus: OrderStatus): boolean;
  executeTransition(orderId: string, newStatus: OrderStatus, userId: string): Promise<void>;
  triggerNotifications(orderId: string, statusChange: StatusChange): Promise<void>;
  updateDependentTasks(orderId: string, newStatus: OrderStatus): Promise<void>;
}

interface StatusTransitionRule {
  from: OrderStatus;
  to: OrderStatus;
  requiredConditions: TransitionCondition[];
  automaticTriggers: AutomatedTrigger[];
  notifications: NotificationConfig[];
}
```

**Business Logic:**
- Validation of status progression rules
- Automatic status updates based on form completion
- Dependency management between orders
- Escalation handling for stuck orders

#### UC 3.2: Notification System
```typescript
interface NotificationSystem {
  sendNotification(notification: NotificationRequest): Promise<void>;
  subscribeToEvents(userId: string, eventTypes: NotificationType[]): void;
  getNotifications(userId: string, filters?: NotificationFilters): Promise<Notification[]>;
  markAsRead(notificationIds: string[]): Promise<void>;
}

interface NotificationRequest {
  recipients: string[];
  type: NotificationType;
  title: string;
  message: string;
  urgency: 'Low' | 'Medium' | 'High' | 'Critical';
  relatedOrderId?: string;
  actionRequired?: boolean;
  expiresAt?: Date;
}
```

**Features:**
- Real-time in-app notifications
- Email notifications for critical events
- Mobile push notifications (future)
- Notification preferences per user
- Automatic escalation for unread critical notifications

---

## UI/UX Design Specifications

### Procurement Specialist Interface

#### BOM Review Interface Design
```tsx
// Main BOM Review Component
const BOMReviewInterface = () => {
  return (
    <div className="grid grid-cols-12 gap-6 h-screen">
      {/* BOM Structure - Left Panel */}
      <div className="col-span-5 border-r">
        <BOMHierarchyViewer
          bom={bomData}
          onItemSelect={handleItemSelect}
          expandedLevels={3}
          searchable
        />
      </div>
      
      {/* Review Panel - Right Panel */}
      <div className="col-span-7">
        <Tabs>
          <TabsList>
            <TabsTrigger value="checklist">Review Checklist</TabsTrigger>
            <TabsTrigger value="modifications">Modifications</TabsTrigger>
            <TabsTrigger value="outsourcing">Outsourcing</TabsTrigger>
            <TabsTrigger value="history">History</TabsTrigger>
          </TabsList>
          
          <TabsContent value="checklist">
            <ReviewChecklist
              items={checklistItems}
              onItemUpdate={handleChecklistUpdate}
            />
          </TabsContent>
          
          <TabsContent value="modifications">
            <ModificationPanel
              bom={bomData}
              onModification={handleModification}
            />
          </TabsContent>
          
          <TabsContent value="outsourcing">
            <OutsourcingPanel
              bomItems={bomData.items}
              onOutsourceSelect={handleOutsourcing}
            />
          </TabsContent>
        </Tabs>
        
        {/* Action Bar */}
        <div className="fixed bottom-0 right-0 p-6 bg-white border-t">
          <div className="flex gap-3">
            <Button variant="outline" onClick={saveDraft}>
              Save Draft
            </Button>
            <Button variant="destructive" onClick={rejectBOM}>
              Reject
            </Button>
            <Button onClick={approveBOM}>
              Approve BOM
            </Button>
          </div>
        </div>
      </div>
    </div>
  );
};
```

### QC Interface Design

#### Pre-QC Form Layout
```tsx
const PreQCForm = () => {
  return (
    <Card className="max-w-4xl mx-auto">
      <CardHeader>
        <div className="flex justify-between items-center">
          <div>
            <CardTitle>Pre-Production Quality Check</CardTitle>
            <CardDescription>
              Order: {poNumber} | Build: {buildNumber}
            </CardDescription>
          </div>
          <Badge variant={getStatusColor(currentStatus)}>
            {currentStatus}
          </Badge>
        </div>
        <Progress value={completionPercentage} className="mt-4" />
      </CardHeader>
      
      <CardContent>
        <Tabs value={currentSection} onValueChange={setCurrentSection}>
          <TabsList className="grid w-full grid-cols-4">
            <TabsTrigger value="basic-info">Basic Info</TabsTrigger>
            <TabsTrigger value="dimensions">Dimensions</TabsTrigger>
            <TabsTrigger value="components">Components</TabsTrigger>
            <TabsTrigger value="review">Review</TabsTrigger>
          </TabsList>
          
          <TabsContent value="basic-info" className="space-y-6">
            <BasicInfoSection
              jobId={formData.jobId}
              numberOfBasins={formData.numberOfBasins}
              onChange={handleBasicInfoChange}
            />
          </TabsContent>
          
          <TabsContent value="dimensions" className="space-y-6">
            <DimensionCheckSection
              requiredDimensions={orderConfig.dimensions}
              actualDimensions={formData.actualDimensions}
              onChange={handleDimensionChange}
            />
          </TabsContent>
          
          <TabsContent value="components" className="space-y-6">
            <ComponentCheckSection
              basins={orderConfig.basins}
              checks={formData.componentChecks}
              onChange={handleComponentChange}
            />
          </TabsContent>
          
          <TabsContent value="review" className="space-y-6">
            <PreQCReviewSection
              formData={formData}
              onSubmit={handleSubmit}
            />
          </TabsContent>
        </Tabs>
      </CardContent>
    </Card>
  );
};
```

### Mobile Responsiveness

```css
/* Mobile-first responsive design for QC forms */
@media (max-width: 768px) {
  .preqc-form {
    @apply px-4;
  }
  
  .bom-review-grid {
    @apply grid-cols-1;
  }
  
  .action-bar {
    @apply flex-col gap-2;
  }
  
  .notification-panel {
    @apply fixed inset-x-0 bottom-0 h-1/2;
  }
}
```

---

## Implementation Plan

### Week 9-10: Procurement Infrastructure
**Tasks:**
- [ ] Database schema extensions for BOM review and outsourcing
- [ ] Procurement Specialist dashboard framework
- [ ] BOM review interface development
- [ ] Basic outsourcing identification system
- [ ] Notification system foundation

**Deliverables:**
- Procurement dashboard with pending BOMs
- Basic BOM review interface
- Database schema for procurement workflows
- Notification system architecture

### Week 11-12: BOM Review & Approval System
**Tasks:**
- [ ] Complete BOM review workflow implementation
- [ ] Modification tracking and versioning
- [ ] Approval/rejection workflow with reasons
- [ ] Outsourcing management interface
- [ ] Integration with order status system

**Deliverables:**
- Full BOM review and approval workflow
- Outsourcing identification and tracking
- Order status integration
- Audit trail implementation

### Week 13-14: QC Dashboard & Pre-QC Forms
**Tasks:**
- [ ] QC Person dashboard development
- [ ] Digital Pre-QC form implementation (CLP.T2.001.V01 Section 1)
- [ ] Document verification system
- [ ] Digital signature integration
- [ ] Mobile-responsive QC interface

**Deliverables:**
- QC dashboard with task management
- Complete digital Pre-QC forms
- Document verification workflow
- Mobile QC interface

---

## Testing Strategy

### Integration Testing
```typescript
describe('BOM Review Workflow', () => {
  test('procurement specialist can review and approve BOM', async () => {
    // Create test BOM
    const bom = await createTestBOM();
    
    // Login as procurement specialist
    const session = await loginUser('procurement-specialist');
    
    // Navigate to BOM review
    const reviewPage = await goToBOMReview(bom.id);
    
    // Perform review steps
    await reviewPage.checkAllItems();
    await reviewPage.addNotes('BOM approved - all items verified');
    await reviewPage.clickApprove();
    
    // Verify status change
    const updatedOrder = await getOrder(bom.orderId);
    expect(updatedOrder.status).toBe('PartsSent');
  });
});

describe('Pre-QC Digital Forms', () => {
  test('QC person can complete Pre-QC check', async () => {
    // Setup test order with arrived status
    const order = await createTestOrder({ status: 'ReadyForPreQC' });
    
    // Login as QC person
    const session = await loginUser('qc-person');
    
    // Complete Pre-QC form
    const qcForm = await openPreQCForm(order.id);
    await qcForm.fillBasicInfo();
    await qcForm.performDimensionChecks();
    await qcForm.verifyComponents();
    await qcForm.submitWithSignature();
    
    // Verify completion
    const result = await getPreQCResult(order.id);
    expect(result.overallStatus).toBe('Pass');
  });
});
```

### Performance Testing
```typescript
describe('Performance Requirements', () => {
  test('BOM review loads within 2 seconds', async () => {
    const startTime = Date.now();
    await loadBOMReview(largeBOMId);
    const loadTime = Date.now() - startTime;
    
    expect(loadTime).toBeLessThan(2000);
  });
  
  test('Pre-QC form saves data within 1 second', async () => {
    const formData = generateLargeFormData();
    
    const startTime = Date.now();
    await savePreQCForm(formData);
    const saveTime = Date.now() - startTime;
    
    expect(saveTime).toBeLessThan(1000);
  });
});
```

---

## Security & Compliance

### Data Validation
```typescript
// BOM Review Validation
const bomReviewSchema = z.object({
  reviewStatus: z.enum(['Approved', 'Rejected', 'RequiresModification']),
  reviewNotes: z.string().min(10).max(1000),
  modifications: z.array(z.object({
    itemId: z.string(),
    changeType: z.enum(['Add', 'Remove', 'Modify', 'Replace']),
    reason: z.string().min(5)
  })).optional(),
  outsourcingItems: z.array(z.object({
    partId: z.string(),
    reason: z.string(),
    estimatedLeadTime: z.number().min(1)
  })).optional()
});

// Pre-QC Form Validation
const preQCSchema = z.object({
  jobId: z.string().min(1),
  numberOfBasins: z.number().min(1).max(3),
  dimensionChecks: z.object({
    finalSinkDimensions: dimensionCheckSchema,
    basinDimensions: z.array(dimensionCheckSchema)
  }),
  digitalSignature: z.string().min(1),
  overallResult: z.enum(['Pass', 'Fail', 'ConditionalPass'])
});
```

### Audit Logging
```typescript
interface AuditLogEntry {
  eventType: 'BOMReview' | 'PreQCComplete' | 'StatusChange' | 'DocumentUpload';
  userId: string;
  orderId: string;
  timestamp: Date;
  details: Record<string, any>;
  ipAddress: string;
  userAgent: string;
}

// Example audit log implementation
const logBOMReview = async (review: BOMReview) => {
  await auditLog.create({
    eventType: 'BOMReview',
    userId: review.reviewerId,
    orderId: review.bom.orderId,
    details: {
      bomId: review.bomId,
      status: review.status,
      modificationsCount: review.modifications?.length || 0,
      outsourcingItemsCount: review.outsourcingItems?.length || 0
    }
  });
};
```

### ISO 13485:2016 Compliance Features
- Complete audit trail for all QC activities
- Digital signature validation and storage
- Document version control and approval tracking
- Non-repudiation through cryptographic signatures
- Data integrity validation and checksums
- Regular compliance reporting capabilities

---

## Risk Mitigation

### Technical Risks
1. **Complex BOM Review Logic**
   - *Risk:* Performance issues with large BOMs
   - *Mitigation:* Lazy loading, pagination, caching strategies
   - *Fallback:* Simplified view mode for large BOMs

2. **Digital Signature Compliance**
   - *Risk:* Legal validity of digital signatures
   - *Mitigation:* Cryptographic signing with timestamps
   - *Fallback:* Hybrid digital/physical signature process

3. **Mobile QC Interface**
   - *Risk:* Poor performance on shop floor devices
   - *Mitigation:* Progressive Web App with offline capabilities
   - *Fallback:* Tablet-specific interface optimization

### Business Risks
1. **User Training Requirements**
   - *Risk:* Complexity of new digital processes
   - *Mitigation:* Comprehensive training program and documentation
   - *Fallback:* Gradual rollout with parallel manual processes

2. **Compliance Validation**
   - *Risk:* Digital processes not meeting audit requirements
   - *Mitigation:* Regular compliance reviews and updates
   - *Fallback:* Manual backup processes for critical compliance items

---

## Acceptance Criteria

### Functional Acceptance
- [ ] Procurement specialists can review BOMs with complete part visibility
- [ ] BOM approval/rejection workflow functions correctly with audit trail
- [ ] Parts outsourcing identification and tracking works accurately
- [ ] QC personnel can complete Pre-QC forms on mobile devices
- [ ] Digital signatures are captured and stored securely
- [ ] Document verification workflow ensures required documents are present
- [ ] Status notifications are sent to appropriate users automatically
- [ ] All data changes are logged for compliance auditing

### Performance Acceptance
- [ ] BOM review interface loads within 2 seconds for BOMs up to 500 items
- [ ] Pre-QC form saves data within 1 second on mobile connections
- [ ] Notification delivery occurs within 30 seconds of trigger events
- [ ] Document upload completes within 5 seconds for files up to 10MB

### Compliance Acceptance
- [ ] All QC activities maintain complete audit trail
- [ ] Digital signatures meet legal requirements for medical device manufacturing
- [ ] Document version control prevents use of outdated specifications
- [ ] Data integrity is maintained throughout all workflow stages

---

## Next Steps

Upon completion of Phase 2:

1. **User Acceptance Testing:** Validate procurement and QC workflows with actual users
2. **Compliance Verification:** Confirm all ISO 13485:2016 requirements are met
3. **Performance Optimization:** Address any identified bottlenecks
4. **Training Material Development:** Create user guides and training videos
5. **Phase 3 Planning:** Begin detailed planning for Assembly & Production workflows

Phase 2 establishes the critical bridge between order creation and production, ensuring quality control and proper procurement processes are in place before assembly begins. 