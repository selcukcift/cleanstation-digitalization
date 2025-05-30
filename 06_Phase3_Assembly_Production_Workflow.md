# Phase 3: Assembly & Production Workflow
## Torvan Medical CleanStation Production Workflow Digitalization

**Version:** 1.0  
**Date:** December 2024  
**Document Type:** Phase Implementation Specification  
**Timeline:** Weeks 15-22 (8 weeks)  
**Dependencies:** Phase 2 Completion, Pre-QC Forms Validation

---

## Executive Summary

Phase 3 implements the core production workflow, transforming the assembly process with guided task management, integrated quality checks, and comprehensive tracking. This phase creates a digital assembly line that ensures consistent quality, reduces errors, and provides complete traceability for each production step.

### Key Deliverables
- **Assembler Dashboard & Task Assignment** - Personalized work queues and guided assembly
- **Dynamic Task List Generation** - Configuration-based assembly instructions
- **Production Checklist Integration** - CLP.T2.001.V01 Sections 2 & 3 implementation
- **Assembly Progress Tracking** - Real-time status and completion monitoring
- **Testing Forms & Results** - Digital testing protocols and result recording
- **Packaging Management System** - Packaging requirements and verification
- **Ready for Final QC Workflow** - Transition to final quality control

### Success Criteria
- Assemblers receive tailored, step-by-step task lists based on order configurations
- Production quality checks are seamlessly integrated into assembly workflows
- Testing procedures are digitized with automated result validation
- Packaging requirements are clearly defined and tracked
- Complete assembly audit trail is maintained for compliance
- System ensures readiness for Final QC with all requirements met

---

## Scope & Objectives

### In Scope
- **Assembly Task Management** - Dynamic task generation and assignment
- **Production Checklists** - CLP.T2.001.V01 Sections 2 & 3 digitization
- **Work Instructions** - Digital work instruction delivery and tracking
- **Testing Integration** - Testing forms, procedures, and result recording
- **Assembly Progress Tracking** - Real-time progress monitoring and reporting
- **Packaging Management** - Packaging verification and documentation
- **Quality Integration** - In-process quality checks and validation
- **Handoff to Final QC** - Complete readiness verification

### Out of Scope (Deferred to Later Phases)
- Final QC workflows (Section 4 of CLP.T2.001.V01)
- Service department functionality
- Inventory management beyond assembly tracking
- Advanced analytics and reporting
- External system integrations

### Objectives
1. **Standardize Assembly Process** - Consistent, repeatable assembly procedures
2. **Reduce Assembly Errors** - Guided instructions and verification steps
3. **Ensure Quality Integration** - Quality checks embedded in assembly workflow
4. **Provide Real-time Visibility** - Progress tracking and status updates
5. **Maintain Compliance** - Complete audit trail for medical device standards
6. **Optimize Workflow** - Efficient task sequencing and resource utilization

---

## Technical Requirements

### Database Schema Extensions

```sql
-- Assembly task management
CREATE TABLE assembly_tasks (
    task_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    order_id UUID NOT NULL REFERENCES orders(order_id),
    build_number VARCHAR(100) NOT NULL,
    task_type task_type_enum NOT NULL,
    sequence_number INTEGER NOT NULL,
    title VARCHAR(255) NOT NULL,
    description TEXT,
    
    -- Task requirements
    required_parts JSONB, -- [{partId, quantity, verified}]
    required_tools JSONB, -- [{toolId, name}]
    work_instruction_id UUID REFERENCES work_instructions(instruction_id),
    
    -- Quality integration
    quality_checks JSONB, -- [{checkId, description, required, result}]
    
    -- Status and assignment
    status task_status_enum DEFAULT 'Pending',
    assigned_to UUID REFERENCES users(user_id),
    estimated_duration INTEGER, -- minutes
    
    -- Completion tracking
    started_at TIMESTAMP WITH TIME ZONE,
    completed_at TIMESTAMP WITH TIME ZONE,
    completed_by UUID REFERENCES users(user_id),
    completion_notes TEXT,
    
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    
    UNIQUE(order_id, build_number, sequence_number)
);

CREATE TYPE task_type_enum AS ENUM (
    'Assembly',
    'QualityCheck',
    'Testing',
    'Packaging',
    'Documentation',
    'Verification'
);

CREATE TYPE task_status_enum AS ENUM (
    'Pending',
    'Ready',
    'InProgress',
    'Blocked',
    'Completed',
    'Failed',
    'Skipped'
);

-- Production checklist completions (Sections 2 & 3)
CREATE TABLE production_checklist_completions (
    completion_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    order_id UUID NOT NULL REFERENCES orders(order_id),
    build_number VARCHAR(100) NOT NULL,
    section production_section_enum NOT NULL,
    template_id UUID NOT NULL REFERENCES qc_form_templates(template_id),
    
    performed_by UUID NOT NULL REFERENCES users(user_id),
    job_id VARCHAR(100),
    
    -- Section-specific data
    checklist_results JSONB NOT NULL, -- Individual item results
    measurements JSONB, -- Recorded measurements
    digital_signature TEXT NOT NULL,
    
    overall_result production_result_enum NOT NULL,
    completed_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    notes TEXT
);

CREATE TYPE production_section_enum AS ENUM ('Section2_Assembly', 'Section3_Installation');
CREATE TYPE production_result_enum AS ENUM ('Pass', 'Fail', 'Rework');

-- Testing procedures and results
CREATE TABLE testing_procedures (
    procedure_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    order_id UUID NOT NULL REFERENCES orders(order_id),
    build_number VARCHAR(100) NOT NULL,
    test_type test_type_enum NOT NULL,
    
    test_parameters JSONB NOT NULL, -- Test-specific parameters
    test_results JSONB NOT NULL, -- Recorded results
    
    performed_by UUID NOT NULL REFERENCES users(user_id),
    testing_equipment JSONB, -- Equipment used
    environmental_conditions JSONB, -- Temperature, humidity, etc.
    
    result test_result_enum NOT NULL,
    performed_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    verified_by UUID REFERENCES users(user_id),
    verification_notes TEXT
);

CREATE TYPE test_type_enum AS ENUM (
    'PressureTest',
    'LeakTest', 
    'FunctionalTest',
    'SafetyTest',
    'PerformanceTest'
);

CREATE TYPE test_result_enum AS ENUM ('Pass', 'Fail', 'Conditional', 'Retest');

-- Packaging management
CREATE TABLE packaging_requirements (
    requirement_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    order_id UUID NOT NULL REFERENCES orders(order_id),
    build_number VARCHAR(100) NOT NULL,
    
    packaging_type packaging_type_enum NOT NULL,
    requirements JSONB NOT NULL, -- Specific packaging requirements
    
    completed BOOLEAN DEFAULT false,
    completed_by UUID REFERENCES users(user_id),
    completed_at TIMESTAMP WITH TIME ZONE,
    verification_photos JSONB, -- Photo documentation
    notes TEXT
);

CREATE TYPE packaging_type_enum AS ENUM (
    'StandardPackaging',
    'CustomPackaging', 
    'ShippingPackaging',
    'ProtectivePackaging'
);

-- Assembly progress tracking
CREATE TABLE assembly_progress (
    progress_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    order_id UUID NOT NULL REFERENCES orders(order_id),
    build_number VARCHAR(100) NOT NULL,
    
    total_tasks INTEGER NOT NULL,
    completed_tasks INTEGER DEFAULT 0,
    failed_tasks INTEGER DEFAULT 0,
    blocked_tasks INTEGER DEFAULT 0,
    
    estimated_completion TIMESTAMP WITH TIME ZONE,
    actual_start TIMESTAMP WITH TIME ZONE,
    actual_completion TIMESTAMP WITH TIME ZONE,
    
    current_phase assembly_phase_enum NOT NULL,
    overall_status assembly_status_enum DEFAULT 'NotStarted',
    
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    
    UNIQUE(order_id, build_number)
);

CREATE TYPE assembly_phase_enum AS ENUM (
    'PreAssembly',
    'MainAssembly',
    'QualityChecks',
    'Testing',
    'Packaging',
    'ReadyForFinalQC'
);

CREATE TYPE assembly_status_enum AS ENUM (
    'NotStarted',
    'InProgress',
    'OnHold',
    'Completed',
    'Failed'
);
```

### API Endpoints

```typescript
// Assembly Task Management
GET    /api/assembly/tasks/assigned        // Get tasks assigned to current user
GET    /api/assembly/tasks/available       // Get available tasks for assignment
POST   /api/assembly/tasks/[taskId]/start  // Start a task
POST   /api/assembly/tasks/[taskId]/complete // Complete a task
PUT    /api/assembly/tasks/[taskId]/status // Update task status
GET    /api/assembly/orders/[orderId]/tasks // Get all tasks for order

// Production Checklists
GET    /api/production/checklists/templates     // Get checklist templates
POST   /api/production/checklists/complete      // Complete checklist section
GET    /api/production/checklists/[orderId]     // Get checklist status

// Testing Management
GET    /api/testing/procedures/[orderId]        // Get testing procedures
POST   /api/testing/procedures/execute          // Execute test procedure
PUT    /api/testing/results/[procedureId]       // Update test results
GET    /api/testing/equipment                   // Get available equipment

// Packaging Management
GET    /api/packaging/requirements/[orderId]    // Get packaging requirements
POST   /api/packaging/complete                  // Complete packaging step
POST   /api/packaging/photos/upload             // Upload packaging photos

// Progress Tracking
GET    /api/assembly/progress/[orderId]         // Get assembly progress
GET    /api/assembly/dashboard                  // Get assembler dashboard data
POST   /api/assembly/handoff-to-qc             // Mark ready for Final QC
```

---

## User Stories & Features

### 1. Assembler Dashboard & Task Management

#### UC 1.1: Personalized Assembler Dashboard
```typescript
interface AssemblerDashboard {
  assignedTasks: AssemblyTask[];
  availableTasks: AssemblyTask[];
  completedToday: CompletedTask[];
  currentWorkload: WorkloadMetrics;
  upcomingDeadlines: TaskDeadline[];
  skillBasedRecommendations: TaskRecommendation[];
}

interface AssemblyTask {
  taskId: string;
  orderId: string;
  buildNumber: string;
  title: string;
  description: string;
  priority: 'High' | 'Medium' | 'Low';
  estimatedDuration: number; // minutes
  requiredSkills: string[];
  requiredTools: Tool[];
  workInstructionId?: string;
  qualityChecks: QualityCheck[];
  status: TaskStatus;
  dueDate: Date;
}
```

**UI Components:**
- Personal task queue with priority sorting
- Skill-based task recommendations
- Workload visualization and capacity planning
- Quick task actions (Start, Pause, Complete)
- Real-time progress indicators

**Features:**
- Auto-assignment based on skills and availability
- Task dependency management
- Break and shift scheduling integration
- Performance metrics and feedback

#### UC 1.2: Task Execution Interface
```typescript
interface TaskExecutionInterface {
  taskDetails: AssemblyTask;
  workInstructions: WorkInstruction[];
  requiredParts: PartRequirement[];
  qualityChecks: QualityCheck[];
  progressTracker: TaskProgress;
  helpResources: HelpResource[];
}

interface WorkInstruction {
  stepNumber: number;
  title: string;
  description: string;
  visualAids: string[]; // URLs to images/videos
  safetyNotes: string[];
  estimatedTime: number;
  verificationRequired: boolean;
}

interface PartRequirement {
  partId: string;
  partName: string;
  quantity: number;
  verified: boolean;
  location?: string; // Where to find the part
  alternativeParts?: string[]; // Acceptable substitutes
}
```

**UI Components:**
- Step-by-step instruction viewer
- Part verification checklist
- Integrated timer and progress tracking
- Photo capture for documentation
- Help and escalation buttons

**Features:**
- Interactive work instructions
- Real-time verification and validation
- Integration with quality check procedures
- Automatic time tracking
- Context-sensitive help system

### 2. Dynamic Task List Generation

#### UC 2.1: Configuration-Based Task Generation
```typescript
interface TaskGenerator {
  generateTasksForOrder(orderId: string, buildNumber: string): Promise<AssemblyTask[]>;
  calculateTaskDependencies(tasks: AssemblyTask[]): TaskDependencyGraph;
  optimizeTaskSequencing(tasks: AssemblyTask[], constraints: Constraint[]): AssemblyTask[];
  assignTasksToAssemblers(tasks: AssemblyTask[], assemblers: Assembler[]): TaskAssignment[];
}

interface TaskGenerationConfig {
  sinkConfiguration: SinkConfiguration;
  productionRequirements: ProductionRequirement[];
  qualityStandards: QualityStandard[];
  availableResources: Resource[];
}

interface TaskDependencyGraph {
  nodes: TaskNode[];
  edges: TaskDependency[];
  criticalPath: string[]; // Task IDs in critical path
  parallelPaths: string[][]; // Tasks that can be done in parallel
}
```

**Business Logic:**
- Analyze order configuration to determine required assembly steps
- Generate tasks based on sink model, basins, accessories
- Calculate dependencies and optimal sequencing
- Consider available skills and resources
- Integrate quality check requirements

#### UC 2.2: Smart Task Sequencing
```typescript
interface TaskSequencer {
  calculateOptimalSequence(tasks: AssemblyTask[]): SequencedTask[];
  identifyBottlenecks(sequence: SequencedTask[]): Bottleneck[];
  balanceWorkload(tasks: SequencedTask[], assemblers: Assembler[]): WorkloadPlan;
  handleTaskBlocking(blockedTask: AssemblyTask): ResolutionPlan;
}

interface SequencedTask extends AssemblyTask {
  sequenceNumber: number;
  predecessors: string[]; // Task IDs that must complete first
  successors: string[]; // Task IDs that depend on this task
  canStartEarly: boolean;
  criticalPathTask: boolean;
}
```

**Features:**
- Critical path analysis for optimal scheduling
- Resource conflict resolution
- Parallel task identification
- Bottleneck prediction and mitigation
- Dynamic re-sequencing based on progress

### 3. Production Checklist Integration (CLP.T2.001.V01 Sections 2 & 3)

#### UC 3.1: Section 2 - Assembly Checklist Integration
```typescript
interface AssemblyChecklist {
  // Based on CLP.T2.001.V01 Section 2: Assembly
  jobId: string;
  buildNumber: string;
  assemblerInitials: string;
  
  structuralAssembly: {
    frameAssembly: ComponentCheck;
    legInstallation: ComponentCheck;
    feetAttachment: ComponentCheck;
    stabilityVerification: StabilityCheck;
  };
  
  basinInstallation: {
    basinMounting: BasinMountCheck[];
    sealingVerification: SealCheck[];
    drainInstallation: DrainCheck[];
    levelingCheck: LevelCheck[];
  };
  
  plumbingAssembly: {
    faucetInstallation: FaucetCheck[];
    sprayerInstallation: SprayerCheck[];
    plumbingConnections: PlumbingCheck[];
    pressureTesting: PressureCheck;
  };
  
  electricalAssembly: {
    wiringInstallation: WiringCheck[];
    componentConnections: ElectricalCheck[];
    groundingVerification: GroundingCheck;
    functionalTesting: ElectricalTest[];
  };
  
  finalAssemblyChecks: {
    overallAlignment: AlignmentCheck;
    operationalTest: OperationalTest;
    safetyVerification: SafetyCheck;
  };
  
  overallResult: 'Pass' | 'Fail' | 'Rework';
  digitalSignature: string;
  completedAt: Date;
}
```

**UI Integration:**
- Checklist items embedded within relevant assembly tasks
- Real-time validation and verification
- Photo documentation for critical checks
- Automatic pass/fail determination
- Integration with assembly progress tracking

#### UC 3.2: Section 3 - Installation & Final Assembly
```typescript
interface InstallationChecklist {
  // Based on CLP.T2.001.V01 Section 3: Installation & Final Assembly
  jobId: string;
  buildNumber: string;
  installerInitials: string;
  
  accessoryInstallation: {
    pegboardInstallation?: PegboardInstallationCheck;
    accessoryMounting: AccessoryMountCheck[];
    accessoryFunctioning: AccessoryFunctionCheck[];
  };
  
  finalConnections: {
    finalPlumbingConnections: PlumbingConnectionCheck[];
    finalElectricalConnections: ElectricalConnectionCheck[];
    systemIntegrationTest: SystemTest;
  };
  
  qualityVerification: {
    visualInspection: VisualInspectionCheck;
    functionalTesting: FunctionalTestSuite;
    performanceVerification: PerformanceCheck;
    safetyCompliance: SafetyComplianceCheck;
  };
  
  documentation: {
    serialNumberApplication: SerialNumberCheck;
    labelApplication: LabelCheck[];
    documentationPackaging: DocumentationCheck;
  };
  
  overallResult: 'Pass' | 'Fail' | 'Rework';
  digitalSignature: string;
  completedAt: Date;
}
```

**Features:**
- Guided installation procedures
- Automated test procedure execution
- Real-time compliance verification
- Digital documentation and labeling
- Seamless transition to packaging

### 4. Testing Integration & Management

#### UC 4.1: Digital Testing Procedures
```typescript
interface TestingProcedure {
  procedureId: string;
  testType: TestType;
  requiredEquipment: TestEquipment[];
  environmentalRequirements: EnvironmentalRequirement[];
  testSteps: TestStep[];
  passCriteria: PassCriteria[];
  failureActions: FailureAction[];
}

interface TestStep {
  stepNumber: number;
  instruction: string;
  expectedResult: string;
  measurementType: 'Visual' | 'Measurement' | 'Functional' | 'Performance';
  tolerances?: Tolerance[];
  recordingRequired: boolean;
  verificationPhotos: boolean;
}

interface TestResult {
  stepNumber: number;
  actualResult: any;
  measurementValue?: number;
  measurementUnit?: string;
  status: 'Pass' | 'Fail' | 'Warning';
  notes?: string;
  photos?: string[]; // URLs to captured photos
}
```

**Testing Types Supported:**
- Pressure testing with automated data recording
- Leak detection and verification
- Functional testing of all components
- Performance validation against specifications
- Safety compliance verification

#### UC 4.2: Test Equipment Integration
```typescript
interface TestEquipmentIntegration {
  equipmentId: string;
  equipmentType: string;
  calibrationStatus: CalibrationStatus;
  dataInterface: DataInterface;
  automatedDataCapture: boolean;
}

interface DataInterface {
  connectionType: 'USB' | 'Bluetooth' | 'WiFi' | 'Manual';
  dataFormat: string;
  realTimeCapture: boolean;
  dataValidation: ValidationRule[];
}
```

**Features:**
- Automated data capture from test equipment
- Real-time measurement validation
- Equipment calibration tracking
- Test result trending and analysis
- Integration with assembly task completion

### 5. Assembly Progress Tracking

#### UC 5.1: Real-Time Progress Monitoring
```typescript
interface ProgressMonitor {
  trackTaskCompletion(taskId: string, completionData: TaskCompletion): void;
  calculateOverallProgress(orderId: string, buildNumber: string): ProgressMetrics;
  identifyDelays(orderId: string): DelayAnalysis[];
  predictCompletionTime(orderId: string): CompletionPrediction;
}

interface ProgressMetrics {
  overallCompletion: number; // percentage
  taskBreakdown: {
    total: number;
    completed: number;
    inProgress: number;
    blocked: number;
    failed: number;
  };
  timeMetrics: {
    estimatedTotal: number; // hours
    actualSpent: number; // hours
    remainingEstimate: number; // hours
  };
  qualityMetrics: {
    passRate: number; // percentage
    reworkRequired: number;
    criticalIssues: number;
  };
}
```

**UI Components:**
- Real-time progress dashboards
- Gantt chart visualization
- Bottleneck identification
- Resource utilization tracking
- Completion time predictions

#### UC 5.2: Exception Handling & Escalation
```typescript
interface ExceptionManager {
  detectExceptions(orderId: string): Exception[];
  categorizeException(exception: Exception): ExceptionCategory;
  triggerEscalation(exception: Exception): EscalationAction[];
  resolveException(exceptionId: string, resolution: Resolution): void;
}

interface Exception {
  exceptionId: string;
  type: 'TaskBlocked' | 'QualityIssue' | 'ResourceUnavailable' | 'EquipmentFailure';
  severity: 'Low' | 'Medium' | 'High' | 'Critical';
  affectedTasks: string[];
  estimatedDelay: number; // hours
  autoResolvable: boolean;
}
```

**Features:**
- Automatic exception detection
- Smart escalation routing
- Resolution tracking and learning
- Impact analysis and mitigation
- Communication with stakeholders

### 6. Packaging Management System

#### UC 6.1: Packaging Requirements Management
```typescript
interface PackagingManager {
  generatePackagingRequirements(orderId: string): PackagingRequirement[];
  validatePackagingCompletion(orderId: string): PackagingValidation;
  documentPackaging(orderId: string, documentation: PackagingDocumentation): void;
  verifyShippingReadiness(orderId: string): ShippingReadiness;
}

interface PackagingRequirement {
  requirementId: string;
  type: PackagingType;
  description: string;
  requiredMaterials: PackagingMaterial[];
  specialInstructions: string[];
  verificationCriteria: VerificationCriterion[];
  photoDocumentationRequired: boolean;
}

interface PackagingDocumentation {
  packagedBy: string;
  packagedAt: Date;
  materialsUsed: string[];
  verificationPhotos: string[];
  shippingLabel: boolean;
  qualitySeals: boolean;
  notes?: string;
}
```

**Features:**
- Configuration-based packaging requirements
- Step-by-step packaging instructions
- Photo documentation and verification
- Shipping label generation
- Quality seal application tracking

---

## UI/UX Design Specifications

### Assembler Interface Design

#### Task Execution Interface
```tsx
const TaskExecutionInterface = () => {
  return (
    <div className="grid grid-cols-12 gap-6 h-screen p-6">
      {/* Task Overview - Top */}
      <div className="col-span-12">
        <Card>
          <CardHeader className="pb-3">
            <div className="flex justify-between items-start">
              <div>
                <CardTitle className="text-xl">
                  {currentTask.title}
                </CardTitle>
                <CardDescription>
                  Order {poNumber} | Build {buildNumber} | Step {stepNumber} of {totalSteps}
                </CardDescription>
              </div>
              <div className="flex gap-2">
                <Badge variant={getPriorityColor(currentTask.priority)}>
                  {currentTask.priority}
                </Badge>
                <Badge variant={getStatusColor(currentTask.status)}>
                  {currentTask.status}
                </Badge>
              </div>
            </div>
            <Progress value={(stepNumber / totalSteps) * 100} className="mt-3" />
          </CardHeader>
        </Card>
      </div>
      
      {/* Work Instructions - Left Panel */}
      <div className="col-span-8">
        <Card className="h-full">
          <CardHeader>
            <CardTitle className="flex items-center gap-2">
              <FileText className="h-5 w-5" />
              Work Instructions
            </CardTitle>
          </CardHeader>
          <CardContent className="h-full overflow-auto">
            <WorkInstructionViewer
              instructions={workInstructions}
              currentStep={currentInstructionStep}
              onStepComplete={handleInstructionStepComplete}
            />
          </CardContent>
        </Card>
      </div>
      
      {/* Task Controls - Right Panel */}
      <div className="col-span-4 space-y-6">
        {/* Parts Verification */}
        <Card>
          <CardHeader>
            <CardTitle className="text-sm">Required Parts</CardTitle>
          </CardHeader>
          <CardContent>
            <PartsChecklist
              requiredParts={currentTask.requiredParts}
              onPartVerify={handlePartVerification}
            />
          </CardContent>
        </Card>
        
        {/* Quality Checks */}
        <Card>
          <CardHeader>
            <CardTitle className="text-sm">Quality Checks</CardTitle>
          </CardHeader>
          <CardContent>
            <QualityCheckList
              checks={currentTask.qualityChecks}
              onCheckComplete={handleQualityCheck}
            />
          </CardContent>
        </Card>
        
        {/* Task Actions */}
        <Card>
          <CardContent className="pt-6">
            <div className="space-y-3">
              <Button 
                className="w-full" 
                onClick={handleTaskComplete}
                disabled={!canCompleteTask}
              >
                Complete Task
              </Button>
              <Button 
                variant="outline" 
                className="w-full"
                onClick={handlePauseTask}
              >
                Pause Task
              </Button>
              <Button 
                variant="destructive" 
                className="w-full"
                onClick={handleReportIssue}
              >
                Report Issue
              </Button>
            </div>
          </CardContent>
        </Card>
      </div>
    </div>
  );
};
```

#### Mobile-Optimized Assembly Interface
```tsx
const MobileAssemblyInterface = () => {
  return (
    <div className="min-h-screen bg-background">
      <div className="sticky top-0 z-50 bg-background border-b">
        <div className="p-4">
          <div className="flex items-center justify-between mb-3">
            <h1 className="font-semibold truncate">{currentTask.title}</h1>
            <Badge variant={getStatusColor(currentTask.status)}>
              {currentTask.status}
            </Badge>
          </div>
          <Progress value={(stepNumber / totalSteps) * 100} />
          <p className="text-sm text-muted-foreground mt-2">
            Step {stepNumber} of {totalSteps}
          </p>
        </div>
      </div>
      
      <div className="p-4 space-y-6">
        <Tabs value={activeTab} onValueChange={setActiveTab}>
          <TabsList className="grid w-full grid-cols-3">
            <TabsTrigger value="instructions">Instructions</TabsTrigger>
            <TabsTrigger value="parts">Parts</TabsTrigger>
            <TabsTrigger value="quality">Quality</TabsTrigger>
          </TabsList>
          
          <TabsContent value="instructions" className="mt-6">
            <MobileWorkInstructions
              instructions={workInstructions}
              currentStep={currentInstructionStep}
            />
          </TabsContent>
          
          <TabsContent value="parts" className="mt-6">
            <MobilePartsChecklist
              requiredParts={currentTask.requiredParts}
              onPartVerify={handlePartVerification}
            />
          </TabsContent>
          
          <TabsContent value="quality" className="mt-6">
            <MobileQualityChecks
              checks={currentTask.qualityChecks}
              onCheckComplete={handleQualityCheck}
            />
          </TabsContent>
        </Tabs>
      </div>
      
      {/* Fixed Action Bar */}
      <div className="fixed bottom-0 left-0 right-0 bg-background border-t p-4">
        <Button 
          className="w-full" 
          onClick={handleTaskComplete}
          disabled={!canCompleteTask}
        >
          Complete Task
        </Button>
      </div>
    </div>
  );
};
```

### Progress Visualization

#### Assembly Progress Dashboard
```tsx
const AssemblyProgressDashboard = () => {
  return (
    <div className="space-y-6">
      {/* Overall Progress */}
      <Card>
        <CardHeader>
          <CardTitle>Assembly Progress Overview</CardTitle>
        </CardHeader>
        <CardContent>
          <div className="grid grid-cols-1 md:grid-cols-4 gap-6">
            <div className="space-y-2">
              <p className="text-sm font-medium">Overall Completion</p>
              <div className="flex items-center space-x-2">
                <Progress value={progressMetrics.overallCompletion} className="flex-1" />
                <span className="text-sm font-medium">
                  {progressMetrics.overallCompletion}%
                </span>
              </div>
            </div>
            
            <div className="space-y-2">
              <p className="text-sm font-medium">Time Progress</p>
              <div className="flex items-center space-x-2">
                <Progress 
                  value={(progressMetrics.timeMetrics.actualSpent / 
                           progressMetrics.timeMetrics.estimatedTotal) * 100} 
                  className="flex-1" 
                />
                <span className="text-sm">
                  {progressMetrics.timeMetrics.actualSpent}h / {progressMetrics.timeMetrics.estimatedTotal}h
                </span>
              </div>
            </div>
            
            <div className="space-y-2">
              <p className="text-sm font-medium">Quality Score</p>
              <div className="flex items-center space-x-2">
                <Progress value={progressMetrics.qualityMetrics.passRate} className="flex-1" />
                <span className="text-sm font-medium">
                  {progressMetrics.qualityMetrics.passRate}%
                </span>
              </div>
            </div>
            
            <div className="space-y-2">
              <p className="text-sm font-medium">On-Time Delivery</p>
              <div className="flex items-center space-x-2">
                <Progress value={onTimePercentage} className="flex-1" />
                <span className="text-sm font-medium">
                  {onTimePercentage}%
                </span>
              </div>
            </div>
          </div>
        </CardContent>
      </Card>
      
      {/* Task Breakdown */}
      <Card>
        <CardHeader>
          <CardTitle>Task Status Breakdown</CardTitle>
        </CardHeader>
        <CardContent>
          <div className="grid grid-cols-2 md:grid-cols-5 gap-4">
            <div className="text-center">
              <div className="text-2xl font-bold text-blue-600">
                {progressMetrics.taskBreakdown.total}
              </div>
              <div className="text-sm text-muted-foreground">Total Tasks</div>
            </div>
            <div className="text-center">
              <div className="text-2xl font-bold text-green-600">
                {progressMetrics.taskBreakdown.completed}
              </div>
              <div className="text-sm text-muted-foreground">Completed</div>
            </div>
            <div className="text-center">
              <div className="text-2xl font-bold text-yellow-600">
                {progressMetrics.taskBreakdown.inProgress}
              </div>
              <div className="text-sm text-muted-foreground">In Progress</div>
            </div>
            <div className="text-center">
              <div className="text-2xl font-bold text-orange-600">
                {progressMetrics.taskBreakdown.blocked}
              </div>
              <div className="text-sm text-muted-foreground">Blocked</div>
            </div>
            <div className="text-center">
              <div className="text-2xl font-bold text-red-600">
                {progressMetrics.taskBreakdown.failed}
              </div>
              <div className="text-sm text-muted-foreground">Failed</div>
            </div>
          </div>
        </CardContent>
      </Card>
    </div>
  );
};
```

---

## Implementation Plan

### Week 15-16: Assembly Infrastructure & Task Management
**Tasks:**
- [ ] Database schema implementation for assembly tasks and progress
- [ ] Task generation engine development
- [ ] Basic assembler dashboard framework
- [ ] Task assignment and status management
- [ ] Work instruction integration

**Deliverables:**
- Assembly task management system
- Basic task generation for MDRD configurations
- Assembler dashboard with task queue
- Task status tracking infrastructure

### Week 17-18: Production Checklist Integration (Section 2)
**Tasks:**
- [ ] Digital production checklist implementation (CLP.T2.001.V01 Section 2)
- [ ] Quality check integration within assembly tasks
- [ ] Real-time validation and verification system
- [ ] Photo documentation capabilities
- [ ] Assembly progress tracking

**Deliverables:**
- Complete Section 2 digital checklist
- Integrated quality verification
- Photo documentation system
- Real-time progress monitoring

### Week 19-20: Testing Integration & Management
**Tasks:**
- [ ] Testing procedure digitization
- [ ] Test equipment integration framework
- [ ] Automated data capture implementation
- [ ] Test result validation and recording
- [ ] Performance and safety testing protocols

**Deliverables:**
- Digital testing procedures
- Test equipment integration
- Automated result recording
- Testing compliance verification

### Week 21-22: Final Assembly & Packaging (Section 3)
**Tasks:**
- [ ] Installation checklist implementation (CLP.T2.001.V01 Section 3)
- [ ] Packaging requirements management
- [ ] Final quality verification system
- [ ] Handoff to Final QC workflow
- [ ] Mobile interface optimization

**Deliverables:**
- Complete Section 3 digital checklist
- Packaging management system
- Final QC handoff workflow
- Mobile-optimized interface

---

## Testing Strategy

### Unit Testing
```typescript
describe('Task Generation Engine', () => {
  test('generates correct tasks for T2-B2 configuration', () => {
    const config = createTestConfiguration('T2-B2');
    const tasks = taskGenerator.generateTasksForOrder(orderId, config);
    
    expect(tasks).toHaveLength(expectedTaskCount);
    expect(tasks.filter(t => t.type === 'Assembly')).toHaveLength(assemblyTaskCount);
    expect(tasks.filter(t => t.type === 'QualityCheck')).toHaveLength(qcTaskCount);
  });
  
  test('calculates correct task dependencies', () => {
    const tasks = generateTestTasks();
    const dependencies = taskGenerator.calculateTaskDependencies(tasks);
    
    expect(dependencies.criticalPath).toContain('frame-assembly');
    expect(dependencies.criticalPath).toContain('basin-installation');
  });
});

describe('Production Checklist Integration', () => {
  test('validates Section 2 checklist completion', () => {
    const checklistData = createSection2TestData();
    const validation = validateProductionChecklist(checklistData);
    
    expect(validation.isValid).toBe(true);
    expect(validation.completionPercentage).toBe(100);
  });
  
  test('handles quality check failures correctly', () => {
    const checklistData = createFailedQualityCheck();
    const result = processQualityCheck(checklistData);
    
    expect(result.overallResult).toBe('Fail');
    expect(result.reworkRequired).toBe(true);
  });
});
```

### Integration Testing
```typescript
describe('Assembly Workflow Integration', () => {
  test('complete assembly workflow from start to Final QC handoff', async () => {
    // Setup order ready for production
    const order = await createTestOrder({ status: 'ReadyForProduction' });
    
    // Generate assembly tasks
    const tasks = await generateAssemblyTasks(order.id);
    expect(tasks.length).toBeGreaterThan(0);
    
    // Simulate task completion
    for (const task of tasks) {
      await completeTask(task.id, createTestTaskCompletion());
    }
    
    // Verify progress tracking
    const progress = await getAssemblyProgress(order.id);
    expect(progress.overallCompletion).toBe(100);
    
    // Verify ready for Final QC
    const updatedOrder = await getOrder(order.id);
    expect(updatedOrder.status).toBe('ReadyForFinalQC');
  });
});
```

### Performance Testing
```typescript
describe('Performance Requirements', () => {
  test('task generation completes within 5 seconds for complex orders', async () => {
    const complexOrder = createComplexTestOrder();
    
    const startTime = Date.now();
    await generateAssemblyTasks(complexOrder.id);
    const generationTime = Date.now() - startTime;
    
    expect(generationTime).toBeLessThan(5000);
  });
  
  test('progress tracking updates within 1 second', async () => {
    const taskCompletion = createTestTaskCompletion();
    
    const startTime = Date.now();
    await updateTaskProgress(taskId, taskCompletion);
    const updateTime = Date.now() - startTime;
    
    expect(updateTime).toBeLessThan(1000);
  });
});
```

---

## Security & Compliance

### Quality Control Validation
```typescript
// Production checklist validation
const productionChecklistSchema = z.object({
  section: z.enum(['Section2_Assembly', 'Section3_Installation']),
  jobId: z.string().min(1),
  buildNumber: z.string().min(1),
  checklistResults: z.record(z.object({
    result: z.enum(['Pass', 'Fail', 'NA']),
    notes: z.string().optional(),
    measurementValue: z.number().optional(),
    verificationPhoto: z.string().url().optional()
  })),
  digitalSignature: z.string().min(1),
  overallResult: z.enum(['Pass', 'Fail', 'Rework'])
});

// Testing procedure validation
const testingProcedureSchema = z.object({
  testType: z.enum(['PressureTest', 'LeakTest', 'FunctionalTest', 'SafetyTest']),
  testParameters: z.record(z.any()),
  testResults: z.record(z.object({
    value: z.number(),
    unit: z.string(),
    withinTolerance: z.boolean(),
    notes: z.string().optional()
  })),
  result: z.enum(['Pass', 'Fail', 'Conditional', 'Retest']),
  equipmentCalibrated: z.boolean()
});
```

### Audit Trail Implementation
```typescript
interface AssemblyAuditEntry {
  eventType: 'TaskStarted' | 'TaskCompleted' | 'QualityCheckPerformed' | 'TestExecuted';
  orderId: string;
  buildNumber: string;
  taskId?: string;
  userId: string;
  timestamp: Date;
  details: {
    taskType?: TaskType;
    qualityResult?: QualityResult;
    testResult?: TestResult;
    duration?: number;
    notes?: string;
  };
  location?: string; // Work station or area
  equipment?: string[]; // Equipment used
}

// Example audit logging
const logTaskCompletion = async (completion: TaskCompletion) => {
  await auditLog.create({
    eventType: 'TaskCompleted',
    orderId: completion.orderId,
    buildNumber: completion.buildNumber,
    taskId: completion.taskId,
    userId: completion.completedBy,
    details: {
      taskType: completion.taskType,
      duration: completion.actualDuration,
      qualityResult: completion.qualityCheckResults,
      notes: completion.completionNotes
    },
    location: completion.workStation,
    equipment: completion.equipmentUsed
  });
};
```

### ISO 13485:2016 Compliance Features
- Complete traceability of all assembly activities
- Digital signature verification for quality checks
- Equipment calibration status tracking
- Environmental condition recording
- Non-conformance tracking and resolution
- Change control for work instructions

---

## Risk Mitigation

### Technical Risks
1. **Complex Task Generation Logic**
   - *Risk:* Incorrect task sequences causing assembly errors
   - *Mitigation:* Extensive testing with all sink configurations
   - *Fallback:* Manual task override capabilities

2. **Mobile Performance Issues**
   - *Risk:* Poor performance on shop floor devices
   - *Mitigation:* Progressive Web App with offline capabilities
   - *Fallback:* Tablet-specific interface optimization

3. **Integration with Testing Equipment**
   - *Risk:* Data capture failures or incompatible interfaces
   - *Mitigation:* Multiple interface options and manual fallback
   - *Fallback:* Manual data entry with validation

### Operational Risks
1. **Assembler Training Requirements**
   - *Risk:* Complex new digital processes
   - *Mitigation:* Comprehensive training and gradual rollout
   - *Fallback:* Parallel manual processes during transition

2. **Quality Control Compliance**
   - *Risk:* Digital checklists not meeting audit requirements
   - *Mitigation:* Regular compliance validation and audits
   - *Fallback:* Hybrid digital/manual quality processes

---

## Acceptance Criteria

### Functional Acceptance
- [ ] Assemblers receive accurate, configuration-based task lists
- [ ] Production checklists (Sections 2 & 3) are fully digitized and functional
- [ ] Quality checks are seamlessly integrated into assembly workflow
- [ ] Testing procedures execute correctly with automated data capture
- [ ] Assembly progress is tracked in real-time with accurate metrics
- [ ] Packaging requirements are generated and verified correctly
- [ ] Handoff to Final QC includes all required verification
- [ ] Mobile interface supports all assembly activities effectively

### Performance Acceptance
- [ ] Task generation completes within 5 seconds for complex orders
- [ ] Progress updates occur within 1 second of task completion
- [ ] Mobile interface loads within 3 seconds on shop floor devices
- [ ] Testing data capture completes within 2 seconds of test completion

### Compliance Acceptance
- [ ] All assembly activities maintain complete audit trail
- [ ] Digital signatures meet medical device manufacturing requirements
- [ ] Quality checks enforce compliance with CLP.T2.001.V01 standards
- [ ] Equipment calibration status is verified before testing
- [ ] Non-conformance tracking captures all quality issues

---

## Next Steps

Upon completion of Phase 3:

1. **Assembly Process Validation:** Conduct full assembly trials with digital system
2. **Quality Control Verification:** Validate compliance with ISO 13485:2016 requirements
3. **Performance Optimization:** Address any identified bottlenecks in assembly workflow
4. **Training Program Execution:** Train assembly team on new digital processes
5. **Phase 4 Planning:** Begin detailed planning for Final QC & Service Department functionality

Phase 3 creates the digital backbone of the production process, ensuring consistent quality, comprehensive tracking, and seamless progression toward final quality control. 