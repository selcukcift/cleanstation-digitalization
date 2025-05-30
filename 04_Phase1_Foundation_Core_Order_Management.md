# Phase 1: Foundation & Core Order Management
## Torvan Medical CleanStation Production Workflow Digitalization

**Version:** 1.0  
**Date:** December 2024  
**Document Type:** Phase Implementation Specification  
**Timeline:** Weeks 1-8  
**Dependencies:** Database Design, Client Clarifications

---

## Executive Summary

Phase 1 establishes the foundational infrastructure for the CleanStation digitalization system, focusing on user authentication, core order management, and the comprehensive 5-step order creation wizard for MDRD sinks. This phase creates the essential backbone that all subsequent phases will build upon.

### Key Deliverables
- **User Authentication & Role Management** - Complete auth system with 6 role types
- **5-Step Order Creation Wizard** - Full MDRD sink configuration workflow
- **Basic BOM Generation** - Automated Bill of Materials creation engine
- **Order Management Dashboard** - View, search, filter, and track orders
- **Database Foundation** - Core schema implementation
- **UI Framework** - ShadCN UI component library setup

### Success Criteria
- Production Coordinators can create complete MDRD orders through the wizard
- System generates accurate BOMs based on sink configurations
- Users can view order details and track current status
- Admin can manage users and basic system data
- All user roles have appropriate access controls and dashboards

---

## Scope & Objectives

### In Scope
- **User Management:** Authentication, authorization, role-based access
- **Order Creation:** Complete 5-step wizard for MDRD sink family
- **BOM Generation:** Automated BOM creation based on configurations
- **Order Tracking:** Status management and order history
- **Basic Dashboards:** Role-specific views and navigation
- **Data Import:** Migration of categories, parts, and assemblies from JSON files

### Out of Scope (Deferred to Later Phases)
- Endoscope CleanStation and InstroSink families (placeholder pages only)
- Quality control workflows and digital forms
- Assembly task management and guidance
- Service department functionality
- Advanced reporting and analytics
- Document upload and management beyond basic order files

### Objectives
1. **Establish Foundation:** Create robust authentication and data architecture
2. **Enable Order Creation:** Full workflow for creating MDRD orders
3. **Automate BOM Generation:** Reduce manual work and errors
4. **Provide Visibility:** Order tracking and status management
5. **Ensure Usability:** Intuitive interfaces for all user roles
6. **Set Standards:** UI/UX patterns and code architecture for future phases

---

## Technical Requirements

### Frontend Architecture

#### Technology Stack
```typescript
// Core Framework
- Next.js 14+ with App Router
- TypeScript for type safety
- React 18+ with Server Components

// UI & Styling
- ShadCN UI component library
- Tailwind CSS for styling
- Framer Motion for animations
- Lucide React for icons

// State Management
- Zustand with Immer for global state
- React Hook Form for form management
- Zod for schema validation

// Development Tools
- ESLint + Prettier for code quality
- Husky for git hooks
- TypeScript strict mode
```

#### Component Architecture
```
src/
├── app/
│   ├── (auth)/
│   │   ├── login/
│   │   └── register/
│   ├── dashboard/
│   │   ├── production-coordinator/
│   │   ├── admin/
│   │   └── [role]/
│   ├── orders/
│   │   ├── create/
│   │   ├── [orderId]/
│   │   └── list/
│   └── api/
├── components/
│   ├── ui/ (ShadCN components)
│   ├── auth/
│   ├── orders/
│   ├── dashboard/
│   └── shared/
├── lib/
│   ├── auth/
│   ├── database/
│   ├── validations/
│   └── utils/
└── stores/
    ├── auth-store.ts
    ├── order-store.ts
    └── ui-store.ts
```

### Backend Architecture

#### API Design
```typescript
// Authentication Endpoints
POST /api/auth/login
POST /api/auth/logout
GET  /api/auth/session
POST /api/auth/refresh

// User Management
GET    /api/users
POST   /api/users
GET    /api/users/[id]
PUT    /api/users/[id]
DELETE /api/users/[id]

// Order Management
GET    /api/orders
POST   /api/orders
GET    /api/orders/[id]
PUT    /api/orders/[id]
DELETE /api/orders/[id]

// BOM Generation
POST   /api/orders/[id]/generate-bom
GET    /api/orders/[id]/bom

// Catalog Data
GET    /api/categories
GET    /api/parts
GET    /api/assemblies
GET    /api/accessories
```

#### Database Schema (Phase 1 Focus)
```sql
-- Core tables for Phase 1
- users (authentication and roles)
- user_sessions (security tracking)
- categories (organizational hierarchy)
- subcategories (detailed organization)
- parts (individual components)
- assemblies (buildable products)
- assembly_components (BOM relationships)
- orders (production orders)
- order_history (audit trail)
- boms (generated BOMs)
```

---

## User Stories & Features

### 1. User Authentication & Role Management

#### UC 1.1: User Login & Session Management
```typescript
interface LoginCredentials {
  username: string;
  password: string;
}

interface UserSession {
  userId: string;
  username: string;
  fullName: string;
  role: UserRole;
  permissions: Permission[];
  sessionExpiry: Date;
}
```

**Acceptance Criteria:**
- Users can log in with username/password
- Sessions expire after configurable timeout (default: 8 hours)
- Role-based redirect to appropriate dashboard
- Secure session management with HTTP-only cookies
- Automatic logout on session expiry

#### UC 1.2: Role-Based Access Control
```typescript
enum UserRole {
  ProductionCoordinator = 'ProductionCoordinator',
  Admin = 'Admin',
  ProcurementSpecialist = 'ProcurementSpecialist',
  QCPerson = 'QCPerson',
  Assembler = 'Assembler',
  ServiceDepartment = 'ServiceDepartment'
}

interface RolePermissions {
  [UserRole.ProductionCoordinator]: [
    'orders:create',
    'orders:read',
    'orders:update_status',
    'boms:read'
  ];
  [UserRole.Admin]: ['*']; // All permissions
  // ... other role permissions
}
```

**Acceptance Criteria:**
- Access control enforced at both frontend and backend
- Users only see features appropriate to their role
- API endpoints validate user permissions
- Clear error messages for unauthorized access

### 2. Order Creation Wizard

#### UC 2.1: Step 1 - Customer & Order Information
```typescript
interface OrderStepOne {
  poNumber: string; // min 3 chars, unique
  customerName: string; // min 3 chars
  projectName?: string; // optional, min 3 chars if provided
  salesPerson: string; // min 3 chars
  wantDate: Date; // must be future date
  poDocument?: File; // PDF upload
  notes?: string;
  documentLanguage: 'EN' | 'FR' | 'SP';
}
```

**UI Components:**
- Form with validation feedback
- Date picker with future date constraint
- File upload with drag-and-drop
- Language selector with flags
- Progress indicator (Step 1 of 5)

**Validation Rules:**
- PO Number uniqueness check (real-time)
- Future date validation for want date
- File type validation (PDF only)
- Required field validation with clear messages

#### UC 2.2: Step 2 - Sink Selection & Quantity
```typescript
interface OrderStepTwo {
  sinkFamily: 'MDRD'; // Only MDRD in Phase 1
  quantity: number; // positive integer
  buildNumbers: BuildNumber[]; // array of unique build numbers
}

interface BuildNumber {
  id: string;
  buildNumber: string; // alphanumeric, unique
  isValid: boolean;
}
```

**UI Components:**
- Sink family selector (MDRD only, others disabled with "Coming Soon")
- Quantity input with increment/decrement controls
- Dynamic build number input fields
- Real-time uniqueness validation
- Build number generation suggestion

**Business Logic:**
- Auto-generate suggested build numbers
- Validate uniqueness across all orders
- Support for multiple sinks per order
- Clear indication of which families are available

#### UC 2.3: Step 3 - Sink Configuration (Per Build Number)
```typescript
interface SinkConfiguration {
  buildNumber: string;
  
  // Sink Body Configuration
  sinkModel: 'T2-B1' | 'T2-B2' | 'T2-B3'; // determines basin count
  sinkDimensions: {
    width: number; // inches
    length: number; // inches
  };
  legsType: 'HeightAdjustable' | 'FixedHeight';
  legsModel: 'DL27' | 'DL14' | 'LC1'; // based on legs type
  feetType: 'LockLevelingCasters' | 'SSAdjustableSeismicFeet';
  
  // Pegboard Configuration
  pegboard: PegboardConfig | null;
  
  workflowDirection: 'LeftToRight' | 'RightToLeft';
  
  // Basin Configurations (array based on sink model)
  basins: BasinConfiguration[];
  
  // Faucet & Sprayer Configurations
  faucets: FaucetConfiguration[];
  sprayers: SprayerConfiguration[];
}

interface PegboardConfig {
  color: 'Green' | 'Black' | 'Yellow' | 'Grey' | 'Red' | 'Blue' | 'Orange' | 'White';
  type: 'Perforated' | 'Solid';
  size: {
    type: 'SameAsSink' | 'Custom';
    width?: number; // if custom
    length?: number; // if custom
  };
}

interface BasinConfiguration {
  index: number;
  type: 'E-Sink' | 'E-Sink DI' | 'E-Drain';
  size: StandardBasinSize | CustomBasinSize;
  addons: BasinAddon[];
}

interface FaucetConfiguration {
  type: string; // from predefined list
  quantity: number; // max based on basin count
  placement: string; // based on basin count
}

interface SprayerConfiguration {
  type: string; // from predefined list
  quantity: 1 | 2;
  location: 'LeftSide' | 'RightSide';
}
```

**UI Components:**
- Multi-step configuration wizard for each build number
- Conditional form fields based on selections
- Real-time validation and preview
- Visual sink configuration builder
- Smart defaults and suggestions

**Business Logic:**
- Auto-determine sink body assembly based on dimensions
- Conditional field display based on selections
- Part number generation for custom items
- Configuration validation across all components

#### UC 2.4: Step 4 - Add-on Accessories
```typescript
interface AccessorySelection {
  accessoryId: string;
  name: string;
  category: string;
  quantity: number;
  canOrder: boolean;
}

interface AccessoryLibrary {
  categories: AccessoryCategory[];
  assemblies: Assembly[];
}
```

**UI Components:**
- Categorized accessory browser
- Search and filter functionality
- Add to order interface with quantity selection
- Selected accessories review panel
- Category-based organization

**Features:**
- Browse accessories from categories.json
- Filter by availability and category
- Add multiple quantities
- Remove or modify selections
- Real-time order total calculation (without pricing)

#### UC 2.5: Step 5 - Review and Submit
```typescript
interface OrderReview {
  customerInfo: OrderStepOne;
  sinkSelection: OrderStepTwo;
  configurations: SinkConfiguration[];
  accessories: AccessorySelection[];
  preliminaryBOMs: BOMPreview[];
}

interface BOMPreview {
  buildNumber: string;
  totalParts: number;
  totalAssemblies: number;
  majorComponents: BOMItem[];
}
```

**UI Components:**
- Comprehensive order summary
- Preliminary BOM display for each build number
- Edit buttons for each step
- Final validation and submission
- Order confirmation and next steps

**Features:**
- Complete order validation
- Preliminary BOM generation preview
- Edit capability for any previous step
- Clear submission process
- Order creation confirmation

### 3. BOM Generation Engine

#### UC 3.1: Configuration-Based BOM Generation
```typescript
interface BOMGenerator {
  generateBOM(configuration: SinkConfiguration): BOMStructure;
  calculateQuantities(items: BOMItem[]): BOMItem[];
  resolveAssemblyComponents(assemblyId: string): BOMItem[];
}

interface BOMStructure {
  buildNumber: string;
  items: BOMItem[];
  totalUniqueparts: number;
  totalAssemblies: number;
  generatedAt: Date;
}

interface BOMItem {
  id: string;
  type: 'PART' | 'ASSEMBLY';
  name: string;
  partNumber: string;
  quantity: number;
  level: number; // hierarchy level
  parent?: string; // parent assembly ID
  children?: BOMItem[]; // child components
  notes?: string;
}
```

**Business Logic:**
- Map sink configurations to specific assemblies and parts
- Handle custom part number generation (e.g., 720.215.001 pattern)
- Resolve assembly hierarchies and component relationships
- Calculate total quantities including duplicates
- Include mandatory components (manuals, standard items)

#### UC 3.2: BOM Validation & Storage
```typescript
interface BOMValidation {
  validateBOM(bom: BOMStructure): ValidationResult;
  checkPartAvailability(partIds: string[]): AvailabilityResult;
  saveBOM(orderId: string, bom: BOMStructure): Promise<string>;
}

interface ValidationResult {
  isValid: boolean;
  errors: BOMError[];
  warnings: BOMWarning[];
}
```

**Features:**
- Validate all parts exist in catalog
- Check for circular references in assemblies
- Verify quantity calculations
- Store BOM with order reference
- Support BOM versioning

### 4. Order Management Dashboard

#### UC 4.1: Order List & Filtering
```typescript
interface OrderFilters {
  status?: OrderStatus[];
  dateRange?: {
    start: Date;
    end: Date;
  };
  customer?: string;
  poNumber?: string;
  salesPerson?: string;
}

interface OrderListItem {
  orderId: string;
  poNumber: string;
  buildNumbers: string[];
  customerName: string;
  status: OrderStatus;
  wantDate: Date;
  createdAt: Date;
  assignedTo?: string;
}
```

**UI Components:**
- Data table with sorting and pagination
- Advanced filtering sidebar
- Search functionality
- Status indicators with color coding
- Bulk operations (if applicable)

**Features:**
- Real-time order status updates
- Responsive table design
- Export functionality (CSV/PDF)
- Quick status filters
- Advanced search capabilities

#### UC 4.2: Order Details View
```typescript
interface OrderDetails {
  orderInfo: OrderInfo;
  configurations: SinkConfiguration[];
  bom: BOMStructure;
  history: OrderHistoryItem[];
  documents: OrderDocument[];
  currentStatus: OrderStatus;
  nextActions: NextAction[];
}

interface OrderHistoryItem {
  timestamp: Date;
  user: string;
  action: string;
  oldStatus?: OrderStatus;
  newStatus?: OrderStatus;
  notes?: string;
}
```

**UI Components:**
- Tabbed interface for different sections
- Configuration viewer with technical details
- Interactive BOM browser
- Timeline view for order history
- Document preview and download

**Features:**
- Complete order information display
- BOM viewer with expand/collapse functionality
- Order history audit trail
- Document management
- Status change tracking

---

## UI/UX Design Specifications

### Design System

#### Color Palette
```css
/* Primary Colors */
--primary-50: #eff6ff;
--primary-500: #3b82f6;
--primary-600: #2563eb;
--primary-700: #1d4ed8;

/* Neutral Colors */
--neutral-50: #f9fafb;
--neutral-100: #f3f4f6;
--neutral-500: #6b7280;
--neutral-900: #111827;

/* Status Colors */
--success-500: #10b981;
--warning-500: #f59e0b;
--error-500: #ef4444;
--info-500: #3b82f6;
```

#### Typography
```css
/* Font Families */
font-family: 'Inter', sans-serif;

/* Font Sizes */
--text-xs: 0.75rem;
--text-sm: 0.875rem;
--text-base: 1rem;
--text-lg: 1.125rem;
--text-xl: 1.25rem;
--text-2xl: 1.5rem;

/* Font Weights */
--font-normal: 400;
--font-medium: 500;
--font-semibold: 600;
--font-bold: 700;
```

#### Component Patterns

##### Dashboard Layout
```tsx
<div className="min-h-screen bg-neutral-50">
  <Sidebar />
  <div className="pl-64">
    <Header />
    <main className="p-6">
      <Breadcrumbs />
      <PageContent />
    </main>
  </div>
</div>
```

##### Form Layout
```tsx
<Card className="max-w-2xl mx-auto">
  <CardHeader>
    <CardTitle>Step {currentStep} of 5</CardTitle>
    <ProgressBar value={currentStep} max={5} />
  </CardHeader>
  <CardContent>
    <Form>
      <FormFields />
    </Form>
  </CardContent>
  <CardFooter>
    <Button variant="outline" onClick={onPrevious}>
      Previous
    </Button>
    <Button onClick={onNext}>
      {isLastStep ? 'Submit' : 'Next'}
    </Button>
  </CardFooter>
</Card>
```

##### Data Table Pattern
```tsx
<div className="space-y-4">
  <div className="flex justify-between items-center">
    <Input placeholder="Search orders..." />
    <FilterPopover />
  </div>
  <DataTable
    columns={orderColumns}
    data={orders}
    pagination
    sorting
  />
</div>
```

### Responsive Design

#### Breakpoints
```css
/* Mobile First Approach */
@media (min-width: 640px) { /* sm */ }
@media (min-width: 768px) { /* md */ }
@media (min-width: 1024px) { /* lg */ }
@media (min-width: 1280px) { /* xl */ }
```

#### Mobile Considerations
- Collapsible sidebar navigation
- Touch-friendly button sizes (min 44px)
- Optimized form layouts for small screens
- Swipe gestures for table navigation
- Modal alternatives for mobile forms

---

## Implementation Plan

### Week 1-2: Project Setup & Authentication
**Tasks:**
- [ ] Next.js project initialization with TypeScript
- [ ] ShadCN UI component library setup
- [ ] Database schema implementation (Phase 1 tables)
- [ ] Authentication system with NextAuth.js
- [ ] Role-based access control middleware
- [ ] Basic user management interface

**Deliverables:**
- Working authentication system
- User registration and login flows
- Role-based dashboard routing
- Basic admin user management

### Week 3-4: Data Import & Core Infrastructure
**Tasks:**
- [ ] JSON data migration scripts (categories, parts, assemblies)
- [ ] Database seeding with initial data
- [ ] API endpoints for catalog data
- [ ] Basic admin interface for data management
- [ ] Global state management setup

**Deliverables:**
- Complete catalog data imported
- Admin interface for basic CRUD operations
- API endpoints for frontend consumption
- Data validation and integrity checks

### Week 5-6: Order Creation Wizard (Steps 1-3)
**Tasks:**
- [ ] Step 1: Customer & Order Information form
- [ ] Step 2: Sink Selection & Quantity interface
- [ ] Step 3: Sink Configuration wizard
- [ ] Form validation and error handling
- [ ] Wizard navigation and state management

**Deliverables:**
- Working first 3 steps of order creation
- Form validation and user feedback
- Configuration preview functionality
- Data persistence between steps

### Week 7-8: Order Creation Completion & BOM Generation
**Tasks:**
- [ ] Step 4: Accessory selection interface
- [ ] Step 5: Review and submission flow
- [ ] BOM generation engine implementation
- [ ] Order storage and retrieval
- [ ] Order management dashboard

**Deliverables:**
- Complete 5-step order creation wizard
- Functional BOM generation
- Order list and details views
- Basic order status tracking

---

## Testing Strategy

### Unit Testing
```typescript
// Example test structure
describe('BOMGenerator', () => {
  test('generates correct BOM for T2-B2 configuration', () => {
    const config = createTestConfiguration();
    const bom = bomGenerator.generateBOM(config);
    
    expect(bom.items).toHaveLength(expectedCount);
    expect(bom.totalUniqueparts).toBe(expectedParts);
  });
});

describe('OrderValidation', () => {
  test('validates required fields correctly', () => {
    const invalidOrder = { poNumber: 'AB' }; // too short
    const result = validateOrder(invalidOrder);
    
    expect(result.isValid).toBe(false);
    expect(result.errors).toContain('PO Number must be at least 3 characters');
  });
});
```

### Integration Testing
- API endpoint testing with mock data
- Database transaction testing
- Authentication flow testing
- BOM generation accuracy testing

### End-to-End Testing
```typescript
// Example E2E test with Playwright
test('Production Coordinator can create complete order', async ({ page }) => {
  await page.goto('/login');
  await page.fill('[data-testid=username]', 'prod-coordinator');
  await page.fill('[data-testid=password]', 'password');
  await page.click('[data-testid=login-button]');
  
  // Navigate to order creation
  await page.click('[data-testid=create-order]');
  
  // Step 1: Customer Information
  await page.fill('[data-testid=po-number]', 'TEST-PO-001');
  await page.fill('[data-testid=customer-name]', 'Test Customer');
  // ... continue through all steps
  
  await page.click('[data-testid=submit-order]');
  await expect(page.locator('[data-testid=success-message]')).toBeVisible();
});
```

### Acceptance Testing
- User role access verification
- Order creation workflow validation
- BOM accuracy verification
- Performance benchmarking

---

## Security Considerations

### Authentication Security
- Secure password hashing with bcrypt
- Session management with secure cookies
- CSRF protection on state-changing operations
- Rate limiting on authentication endpoints

### Authorization Security
- Role-based access control at API level
- Input validation on all endpoints
- SQL injection prevention with parameterized queries
- XSS protection with content security policy

### Data Security
- Sensitive data encryption at rest
- Secure file upload handling
- Audit logging for compliance
- Regular security updates and patches

---

## Performance Requirements

### Response Time Targets
- Page load times: < 2 seconds
- API response times: < 500ms
- BOM generation: < 5 seconds
- Order search: < 1 second

### Scalability Targets
- Support 50 concurrent users
- Handle 1000+ orders per month
- Database queries optimized for growth
- Efficient caching strategies

---

## Deployment Strategy

### Environment Setup
```yaml
# Docker configuration
services:
  web:
    build: .
    ports:
      - "3000:3000"
    environment:
      - DATABASE_URL=${DATABASE_URL}
      - NEXTAUTH_SECRET=${NEXTAUTH_SECRET}
  
  db:
    image: postgres:15
    environment:
      - POSTGRES_DB=cleanstation
      - POSTGRES_USER=${DB_USER}
      - POSTGRES_PASSWORD=${DB_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data
```

### CI/CD Pipeline
1. **Build Stage:** TypeScript compilation, linting, testing
2. **Test Stage:** Unit tests, integration tests, E2E tests
3. **Deploy Stage:** Docker image build and deployment
4. **Monitoring:** Application monitoring and alerting

---

## Acceptance Criteria

### Functional Acceptance
- [ ] All user roles can successfully log in and access appropriate dashboards
- [ ] Production Coordinators can create complete MDRD orders through the 5-step wizard
- [ ] System generates accurate BOMs based on sink configurations
- [ ] Order list displays all orders with correct filtering and search
- [ ] Order details show complete configuration and BOM information
- [ ] Admin users can manage other users and basic system data

### Technical Acceptance
- [ ] All API endpoints respond within performance targets
- [ ] Database schema supports all required data relationships
- [ ] Frontend components follow design system standards
- [ ] Authentication and authorization work correctly
- [ ] Form validation provides clear user feedback
- [ ] System handles errors gracefully with user-friendly messages

### Quality Acceptance
- [ ] Code coverage > 80% for critical business logic
- [ ] All security requirements implemented
- [ ] Performance targets met under normal load
- [ ] Accessibility guidelines followed (WCAG 2.1)
- [ ] Cross-browser compatibility verified
- [ ] Mobile responsiveness tested and working

---

## Risk Mitigation

### Technical Risks
1. **BOM Generation Complexity**
   - *Mitigation:* Extensive testing with known configurations
   - *Fallback:* Manual BOM review and correction interface

2. **Data Migration Issues**
   - *Mitigation:* Thorough data validation and testing
   - *Fallback:* Rollback procedures and data restoration

3. **Performance Issues**
   - *Mitigation:* Load testing and optimization
   - *Fallback:* Caching strategies and database optimization

### Business Risks
1. **User Adoption Challenges**
   - *Mitigation:* User training and gradual rollout
   - *Fallback:* Enhanced user support and documentation

2. **Requirements Changes**
   - *Mitigation:* Regular client communication and validation
   - *Fallback:* Flexible architecture for modifications

---

## Next Steps

Upon completion of Phase 1:

1. **User Acceptance Testing:** Conduct thorough testing with actual users
2. **Performance Optimization:** Address any performance bottlenecks
3. **Documentation:** Complete user guides and technical documentation
4. **Phase 2 Planning:** Begin detailed planning for Procurement & Pre-QC workflows
5. **Training Preparation:** Prepare training materials for end users

Phase 1 provides the essential foundation for the entire CleanStation digitalization project, establishing the architecture, patterns, and core functionality that will support all subsequent phases. 