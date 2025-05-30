# Database Design & Architecture Specification
## Torvan Medical CleanStation Production Workflow Digitalization

**Version:** 1.0  
**Date:** December 2024  
**Document Type:** Technical Architecture  
**Database:** PostgreSQL 15+

---

## Executive Summary

This document defines the complete database architecture for the CleanStation Production Workflow Digitalization system. The design supports hierarchical data structures, complex BOM relationships, workflow status tracking, quality control processes, and ISO 13485:2016 compliance requirements.

### Key Architecture Features
- **PostgreSQL 15+** with JSON/JSONB support for complex data
- **Hierarchical Data Models** for categories, assemblies, and BOMs
- **Audit Logging** for all critical operations
- **Referential Integrity** with cascading rules
- **Performance Optimization** with strategic indexing
- **Compliance Support** for medical device manufacturing

---

## Database Architecture Overview

### Technology Stack
- **Primary Database:** PostgreSQL 15+
- **Connection Pooling:** PgBouncer
- **ORM/Query Builder:** Prisma
- **Migration Management:** Prisma Migrate
- **Backup Strategy:** Automated daily backups with point-in-time recovery
- **Monitoring:** PostgreSQL monitoring with query performance tracking

### Design Principles
1. **Data Integrity:** Strong foreign key relationships with appropriate cascade rules
2. **Audit Trail:** Complete audit logging for compliance requirements
3. **Performance:** Optimized indexing for common query patterns
4. **Scalability:** Design to support growing data volumes and user base
5. **Flexibility:** JSON fields for extensible configuration data
6. **Compliance:** Support for ISO 13485:2016 traceability requirements

---

## Core Data Model

### 1. User Management Schema

```sql
-- Users table for authentication and role management
CREATE TABLE users (
    user_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash TEXT NOT NULL,
    full_name VARCHAR(255) NOT NULL,
    initials VARCHAR(10), -- For QC/Assembly sign-offs
    role user_role_enum NOT NULL,
    is_active BOOLEAN DEFAULT true,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    last_login TIMESTAMP WITH TIME ZONE
);

-- User roles enumeration
CREATE TYPE user_role_enum AS ENUM (
    'ProductionCoordinator',
    'Admin',
    'ProcurementSpecialist', 
    'QCPerson',
    'Assembler',
    'ServiceDepartment'
);

-- User sessions for security tracking
CREATE TABLE user_sessions (
    session_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(user_id) ON DELETE CASCADE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    expires_at TIMESTAMP WITH TIME ZONE NOT NULL,
    ip_address INET,
    user_agent TEXT
);
```

### 2. Inventory & Catalog Schema

```sql
-- Categories for organizational hierarchy
CREATE TABLE categories (
    category_id VARCHAR(10) PRIMARY KEY, -- e.g., "718"
    name VARCHAR(255) NOT NULL,
    description TEXT,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Subcategories for detailed organization
CREATE TABLE subcategories (
    subcategory_id VARCHAR(20) PRIMARY KEY, -- e.g., "718.001"
    parent_category_id VARCHAR(10) NOT NULL REFERENCES categories(category_id),
    name VARCHAR(255) NOT NULL,
    description TEXT,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Parts catalog - individual components
CREATE TABLE parts (
    part_id VARCHAR(50) PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    manufacturer_part_number VARCHAR(100),
    manufacturer_info VARCHAR(255),
    type part_type_enum NOT NULL,
    status part_status_enum DEFAULT 'ACTIVE',
    photo_url TEXT,
    technical_drawing_url TEXT,
    description TEXT,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Part types and status enums
CREATE TYPE part_type_enum AS ENUM ('COMPONENT', 'MATERIAL');
CREATE TYPE part_status_enum AS ENUM ('ACTIVE', 'INACTIVE', 'DISCONTINUED');

-- Assemblies - buildable products and sub-assemblies
CREATE TABLE assemblies (
    assembly_id VARCHAR(50) PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    type assembly_type_enum NOT NULL,
    category_code VARCHAR(10) REFERENCES categories(category_id),
    subcategory_code VARCHAR(20) REFERENCES subcategories(subcategory_id),
    can_order BOOLEAN DEFAULT false,
    is_kit BOOLEAN DEFAULT false,
    status assembly_status_enum DEFAULT 'ACTIVE',
    photo_url TEXT,
    technical_drawing_url TEXT,
    qr_data TEXT, -- For QR code generation
    kit_components JSONB, -- For kit-specific component details
    work_instruction_id UUID, -- FK to work_instructions
    description TEXT,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Assembly types and status enums
CREATE TYPE assembly_type_enum AS ENUM ('SIMPLE', 'COMPLEX', 'SERVICE_PART', 'KIT');
CREATE TYPE assembly_status_enum AS ENUM ('ACTIVE', 'INACTIVE', 'DISCONTINUED');

-- Assembly component relationships (BOM structure)
CREATE TABLE assembly_components (
    component_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    parent_assembly_id VARCHAR(50) NOT NULL REFERENCES assemblies(assembly_id),
    child_part_id VARCHAR(50), -- Can reference parts OR assemblies
    child_assembly_id VARCHAR(50),
    quantity INTEGER NOT NULL DEFAULT 1,
    notes TEXT,
    is_optional BOOLEAN DEFAULT false,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    
    -- Ensure either part_id OR assembly_id is specified, not both
    CONSTRAINT check_child_reference CHECK (
        (child_part_id IS NOT NULL AND child_assembly_id IS NULL) OR
        (child_part_id IS NULL AND child_assembly_id IS NOT NULL)
    )
);

-- Add foreign key constraints for assembly components
ALTER TABLE assembly_components 
ADD CONSTRAINT fk_child_part FOREIGN KEY (child_part_id) REFERENCES parts(part_id);

ALTER TABLE assembly_components 
ADD CONSTRAINT fk_child_assembly FOREIGN KEY (child_assembly_id) REFERENCES assemblies(assembly_id);
```

### 3. Orders & Production Schema

```sql
-- Order status enumeration
CREATE TYPE order_status_enum AS ENUM (
    'OrderCreated',
    'PartsSent',
    'ReadyForPreQC',
    'ReadyForProduction',
    'TestingComplete',
    'PackagingComplete',
    'ReadyForFinalQC',
    'ReadyForShip',
    'Shipped'
);

-- Language enumeration for manuals
CREATE TYPE language_enum AS ENUM ('EN', 'FR', 'SP');

-- Sink family enumeration
CREATE TYPE sink_family_enum AS ENUM ('MDRD', 'Endoscope', 'InstroSink');

-- Main orders table
CREATE TABLE orders (
    order_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    po_number VARCHAR(100) NOT NULL,
    build_number VARCHAR(100) NOT NULL,
    customer_name VARCHAR(255) NOT NULL,
    project_name VARCHAR(255),
    sales_person VARCHAR(255) NOT NULL,
    want_date DATE NOT NULL,
    order_language language_enum DEFAULT 'EN',
    
    -- Sink configuration
    sink_family sink_family_enum NOT NULL,
    sink_model VARCHAR(50),
    sink_dimensions JSONB, -- {width: number, length: number}
    legs_type VARCHAR(100),
    legs_model VARCHAR(100),
    feet_type VARCHAR(100),
    
    -- Pegboard configuration
    has_pegboard BOOLEAN DEFAULT false,
    pegboard_color VARCHAR(50),
    pegboard_type VARCHAR(50),
    pegboard_size JSONB, -- {width: number, length: number} or "SameAsSink"
    
    workflow_direction VARCHAR(50),
    
    -- Configuration arrays (stored as JSONB)
    basin_configurations JSONB, -- Array of basin config objects
    faucet_configurations JSONB, -- Array of faucet config objects
    sprayer_configurations JSONB, -- Array of sprayer config objects
    selected_accessories JSONB, -- Array of {accessoryId, quantity}
    
    -- Status and assignment
    status order_status_enum DEFAULT 'OrderCreated',
    current_assignee UUID REFERENCES users(user_id),
    
    -- BOM reference
    generated_bom_id UUID, -- FK to boms table
    
    -- Audit fields
    created_by UUID NOT NULL REFERENCES users(user_id),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    
    -- Unique constraint for PO + Build Number combination
    UNIQUE(po_number, build_number)
);

-- Order history for audit trail
CREATE TABLE order_history (
    history_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    order_id UUID NOT NULL REFERENCES orders(order_id) ON DELETE CASCADE,
    user_id UUID NOT NULL REFERENCES users(user_id),
    action VARCHAR(100) NOT NULL,
    old_status order_status_enum,
    new_status order_status_enum,
    notes TEXT,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Order documents association
CREATE TABLE order_documents (
    document_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    order_id UUID NOT NULL REFERENCES orders(order_id) ON DELETE CASCADE,
    filename VARCHAR(255) NOT NULL,
    file_url TEXT NOT NULL,
    file_type VARCHAR(50),
    file_size INTEGER,
    uploaded_by UUID NOT NULL REFERENCES users(user_id),
    uploaded_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    document_type VARCHAR(50) -- 'PO', 'TechnicalDrawing', 'QCForm', etc.
);
```

### 4. Bill of Materials (BOM) Schema

```sql
-- Generated BOMs for orders
CREATE TABLE boms (
    bom_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    order_id UUID NOT NULL REFERENCES orders(order_id),
    generated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    generated_by UUID NOT NULL REFERENCES users(user_id),
    status VARCHAR(50) DEFAULT 'Generated', -- 'Generated', 'Approved', 'Rejected'
    approved_by UUID REFERENCES users(user_id),
    approved_at TIMESTAMP WITH TIME ZONE,
    
    -- Hierarchical BOM data stored as JSONB
    bom_items JSONB NOT NULL, -- Complete BOM structure with quantities
    
    -- Metadata
    total_unique_parts INTEGER,
    total_assemblies INTEGER,
    version INTEGER DEFAULT 1
);

-- BOM approval history
CREATE TABLE bom_approvals (
    approval_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    bom_id UUID NOT NULL REFERENCES boms(bom_id) ON DELETE CASCADE,
    user_id UUID NOT NULL REFERENCES users(user_id),
    action VARCHAR(50) NOT NULL, -- 'Approved', 'Rejected', 'Modified'
    notes TEXT,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

### 5. Quality Control Schema

```sql
-- QC form templates
CREATE TABLE qc_form_templates (
    template_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    form_name VARCHAR(255) NOT NULL,
    form_type qc_form_type_enum NOT NULL,
    version INTEGER DEFAULT 1,
    is_active BOOLEAN DEFAULT true,
    
    -- Checklist structure stored as JSONB
    checklist_items JSONB NOT NULL,
    
    created_by UUID NOT NULL REFERENCES users(user_id),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- QC form types
CREATE TYPE qc_form_type_enum AS ENUM ('PreQC', 'FinalQC', 'InProcessAssembly');

-- QC results for completed forms
CREATE TABLE qc_results (
    result_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    order_id UUID NOT NULL REFERENCES orders(order_id),
    build_number VARCHAR(100),
    template_id UUID NOT NULL REFERENCES qc_form_templates(template_id),
    qc_type qc_form_type_enum NOT NULL,
    
    -- QC performer information
    performed_by UUID NOT NULL REFERENCES users(user_id),
    job_id VARCHAR(100), -- As per checklist requirements
    number_of_basins INTEGER,
    
    -- Results
    overall_status qc_status_enum NOT NULL,
    item_results JSONB NOT NULL, -- Results for each checklist item
    digital_signature TEXT, -- User initials + timestamp or crypto signature
    
    performed_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    notes TEXT
);

-- QC status enumeration
CREATE TYPE qc_status_enum AS ENUM ('Pass', 'Fail', 'Incomplete', 'InProgress');
```

### 6. Work Instructions & Tasks Schema

```sql
-- Work instructions library
CREATE TABLE work_instructions (
    instruction_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    title VARCHAR(255) NOT NULL,
    associated_assembly_id VARCHAR(50) REFERENCES assemblies(assembly_id),
    
    -- Instruction steps stored as JSONB array
    steps JSONB NOT NULL, -- [{stepNumber, description, visualUrl, safetyNotes}]
    
    version INTEGER DEFAULT 1,
    is_active BOOLEAN DEFAULT true,
    created_by UUID NOT NULL REFERENCES users(user_id),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Tools catalog
CREATE TABLE tools (
    tool_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(255) NOT NULL,
    description TEXT,
    image_url TEXT,
    is_active BOOLEAN DEFAULT true,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Task lists for assembly guidance
CREATE TABLE task_lists (
    task_list_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    order_id UUID REFERENCES orders(order_id), -- Can be order-specific
    assembly_type VARCHAR(100), -- Or generic for assembly type
    
    -- Tasks stored as JSONB array
    tasks JSONB NOT NULL, -- [{sequence, description, workInstructionId, toolIds, partIds, checklistItemId}]
    
    created_by UUID NOT NULL REFERENCES users(user_id),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Task completion tracking
CREATE TABLE task_completions (
    completion_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    task_list_id UUID NOT NULL REFERENCES task_lists(task_list_id),
    order_id UUID NOT NULL REFERENCES orders(order_id),
    task_sequence INTEGER NOT NULL,
    
    completed_by UUID NOT NULL REFERENCES users(user_id),
    completed_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    notes TEXT,
    
    UNIQUE(task_list_id, order_id, task_sequence)
);
```

### 7. Testing & Results Schema

```sql
-- Testing form templates
CREATE TABLE testing_form_templates (
    template_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    form_name VARCHAR(255) NOT NULL,
    test_type VARCHAR(100) NOT NULL,
    version INTEGER DEFAULT 1,
    is_active BOOLEAN DEFAULT true,
    
    -- Test items structure
    test_items JSONB NOT NULL,
    
    created_by UUID NOT NULL REFERENCES users(user_id),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Testing results
CREATE TABLE testing_results (
    result_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    order_id UUID NOT NULL REFERENCES orders(order_id),
    template_id UUID NOT NULL REFERENCES testing_form_templates(template_id),
    
    performed_by UUID NOT NULL REFERENCES users(user_id),
    overall_status test_status_enum NOT NULL,
    test_results JSONB NOT NULL, -- Results for each test item
    
    performed_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    notes TEXT
);

-- Test status enumeration
CREATE TYPE test_status_enum AS ENUM ('Pass', 'Fail', 'Incomplete');
```

### 8. Service Orders Schema

```sql
-- Service part orders from service department
CREATE TABLE service_orders (
    service_order_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    requested_by UUID NOT NULL REFERENCES users(user_id),
    request_timestamp TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    
    status service_order_status_enum DEFAULT 'Pending',
    
    -- Requested items stored as JSONB
    items JSONB NOT NULL, -- [{partId, quantity}]
    
    processed_by UUID REFERENCES users(user_id),
    processed_timestamp TIMESTAMP WITH TIME ZONE,
    processing_notes TEXT
);

-- Service order status enumeration
CREATE TYPE service_order_status_enum AS ENUM ('Pending', 'Approved', 'Rejected', 'Fulfilled');
```

---

## Indexing Strategy

### Primary Indexes

```sql
-- Performance indexes for common queries
CREATE INDEX idx_orders_status ON orders(status);
CREATE INDEX idx_orders_po_number ON orders(po_number);
CREATE INDEX idx_orders_created_at ON orders(created_at);
CREATE INDEX idx_orders_assignee ON orders(current_assignee);

-- QC and task tracking indexes
CREATE INDEX idx_qc_results_order ON qc_results(order_id);
CREATE INDEX idx_qc_results_performer ON qc_results(performed_by);
CREATE INDEX idx_task_completions_order ON task_completions(order_id);

-- Assembly and parts indexes
CREATE INDEX idx_assembly_components_parent ON assembly_components(parent_assembly_id);
CREATE INDEX idx_assembly_components_child_part ON assembly_components(child_part_id);
CREATE INDEX idx_assembly_components_child_assembly ON assembly_components(child_assembly_id);

-- Document and history indexes
CREATE INDEX idx_order_documents_order ON order_documents(order_id);
CREATE INDEX idx_order_history_order ON order_history(order_id);
CREATE INDEX idx_order_history_timestamp ON order_history(created_at);

-- User and session indexes
CREATE INDEX idx_user_sessions_user ON user_sessions(user_id);
CREATE INDEX idx_user_sessions_expires ON user_sessions(expires_at);
```

### JSONB Indexes for Configuration Data

```sql
-- GIN indexes for JSONB fields
CREATE INDEX idx_orders_basin_configs ON orders USING GIN (basin_configurations);
CREATE INDEX idx_orders_faucet_configs ON orders USING GIN (faucet_configurations);
CREATE INDEX idx_boms_items ON boms USING GIN (bom_items);
CREATE INDEX idx_qc_results_items ON qc_results USING GIN (item_results);
```

---

## Data Validation & Constraints

### Business Rules Implementation

```sql
-- Ensure work instruction reference in assemblies
ALTER TABLE assemblies 
ADD CONSTRAINT fk_work_instruction 
FOREIGN KEY (work_instruction_id) REFERENCES work_instructions(instruction_id);

-- Ensure BOM reference in orders
ALTER TABLE orders 
ADD CONSTRAINT fk_generated_bom 
FOREIGN KEY (generated_bom_id) REFERENCES boms(bom_id);

-- Ensure want_date is in the future for new orders
ALTER TABLE orders 
ADD CONSTRAINT check_want_date_future 
CHECK (want_date >= CURRENT_DATE);

-- Ensure quantity in assembly components is positive
ALTER TABLE assembly_components 
ADD CONSTRAINT check_positive_quantity 
CHECK (quantity > 0);

-- Ensure QC results have valid status progression
-- (This would be enforced in application logic as well)

-- Ensure user initials are provided for QC/Assembler roles
-- (This constraint would be conditional and enforced in application)
```

### Audit Triggers

```sql
-- Automatic updated_at timestamp trigger
CREATE OR REPLACE FUNCTION update_updated_at_column()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = NOW();
    RETURN NEW;
END;
$$ language 'plpgsql';

-- Apply to relevant tables
CREATE TRIGGER update_users_updated_at BEFORE UPDATE ON users
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

CREATE TRIGGER update_orders_updated_at BEFORE UPDATE ON orders
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

CREATE TRIGGER update_assemblies_updated_at BEFORE UPDATE ON assemblies
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

-- Add similar triggers for other tables with updated_at columns
```

---

## Migration Strategy

### Phase 1 Migrations
1. **User Management Schema** - Users, roles, sessions
2. **Core Inventory Schema** - Categories, subcategories, parts, assemblies
3. **Basic Orders Schema** - Orders table with essential fields
4. **Initial Data Import** - Categories, parts, and assemblies from JSON files

### Phase 2 Migrations
1. **QC Schema** - QC form templates and results
2. **Document Management** - Order documents association
3. **BOM Schema** - BOM generation and approval tracking

### Phase 3 Migrations
1. **Task Management Schema** - Work instructions, tools, task lists
2. **Testing Schema** - Testing templates and results
3. **Task Completion Tracking**

### Phase 4 Migrations
1. **Service Orders Schema** - Service department functionality
2. **Performance Optimizations** - Additional indexes and constraints
3. **Audit Enhancements** - Additional logging and tracking

---

## Performance Considerations

### Query Optimization
- **Connection Pooling:** PgBouncer for efficient connection management
- **Query Planning:** Regular ANALYZE for optimal query plans
- **Materialized Views:** For complex reporting queries
- **Partitioning:** Consider partitioning large tables by date if needed

### Scalability Planning
- **Horizontal Scaling:** Read replicas for reporting queries
- **Archival Strategy:** Archive old orders and results to maintain performance
- **Monitoring:** Query performance monitoring and slow query identification

### Backup & Recovery
- **Daily Backups:** Automated full database backups
- **Point-in-time Recovery:** Transaction log archival
- **Testing:** Regular backup restoration testing
- **Disaster Recovery:** Multi-region backup storage

---

## Security Considerations

### Data Protection
- **Encryption at Rest:** Database-level encryption
- **Encryption in Transit:** SSL/TLS for all connections
- **Access Control:** Role-based database access
- **Audit Logging:** Database-level audit trail

### Compliance Support
- **Data Retention:** Configurable retention policies
- **Audit Trails:** Complete change tracking for compliance
- **Data Integrity:** Checksums and validation
- **Access Logging:** User activity tracking

---

## Next Steps

1. **Environment Setup:** Configure PostgreSQL instances for development, staging, and production
2. **Prisma Configuration:** Set up Prisma schema and initial migrations
3. **Data Migration Scripts:** Prepare scripts for importing existing data
4. **Testing Setup:** Configure database testing environment
5. **Monitoring Setup:** Implement database monitoring and alerting

This database design provides a solid foundation for the CleanStation digitalization project while maintaining flexibility for future enhancements and ensuring compliance with medical device manufacturing requirements. 