# Client Questions & Clarifications Required
## Torvan Medical CleanStation Production Workflow Digitalization

**Version:** 1.0  
**Date:** December 2024  
**Document Type:** Requirements Clarification  
**Priority:** CRITICAL - Required before Phase 1 implementation

---

## Executive Summary

This document outlines critical questions and clarifications required from Torvan Medical before proceeding with the implementation of the CleanStation Production Workflow Digitalization project. These clarifications will ensure accurate requirements interpretation, proper compliance implementation, and successful system delivery.

**Action Required:** Client response needed within 2 weeks to maintain project timeline.

---

## Critical Priority Questions

### 1. QC Checklist Documentation (HIGH PRIORITY)

**Question:** Is CLP.T2.001.V01 - T2SinkProduction.docx the definitive and complete source for all Pre-QC, Production, and Packaging checklists, or is it an example document?

**Context:** The PRD references this document as the basis for digital checklists across multiple phases.

**Clarifications Needed:**
- Are there other versions of this document (V02, V03, etc.)?
- Are there separate documents for different sink families or models?
- Who maintains and updates this document?
- What is the approval process for checklist changes?
- How frequently are these checklists updated?

**Impact:** This affects the entire QC workflow design and data model structure.

---

### 2. Digital Signatures & Compliance (HIGH PRIORITY)

**Question:** How should "INITIALS" and "Signature" fields from the production checklists be handled digitally for traceability and ISO 13485:2016 compliance?

**Context:** Current checklists require physical signatures for compliance and audit trails.

**Options for Consideration:**
- User's logged-in identity + timestamp
- Digital signature with cryptographic verification
- Electronic signature capture
- Simple username/initials with audit logging

**Clarifications Needed:**
- What level of digital signature is acceptable for compliance?
- Are there specific ISO 13485:2016 requirements for digital signatures?
- Do auditors have specific expectations for electronic records?
- Is a simple username + timestamp sufficient, or is cryptographic signing required?

**Impact:** Affects user authentication design, audit logging, and compliance architecture.

---

### 3. N/A Options in Digital Forms (MEDIUM PRIORITY)

**Question:** How should "N/A" options in the production checklists be handled in digital forms?

**Context:** Physical checklists often have items marked as "N/A" based on configuration.

**Options for Consideration:**
- Separate N/A checkbox for each item
- Automatic N/A assignment based on configuration
- Hide non-applicable items entirely
- Require justification for N/A selections

**Clarifications Needed:**
- Should N/A be a user choice or system-determined?
- Are there items that should never be N/A?
- Is justification required for N/A selections?
- How are N/A items handled in current compliance audits?

**Impact:** Affects form design, validation logic, and compliance reporting.

---

### 4. Document Verification Process (MEDIUM PRIORITY)

**Question:** For checklist items like "Attach the final approved drawing and paperwork," how will this be verified digitally?

**Context:** Current process involves physical document verification.

**Clarifications Needed:**
- What constitutes "approved" documentation in the digital system?
- Who has authority to approve documents?
- Should the system enforce document approval before proceeding?
- Are there specific document formats required?
- How are document revisions handled?

**Impact:** Affects document management system design and workflow validation.

---

## Operational Questions

### 5. Parts Outsourcing Identification (MEDIUM PRIORITY)

**Question:** How are "Parts for Outsourcing" currently identified? Should the system automate this identification or rely on manual flagging by the Procurement Specialist?

**Current Understanding:** Procurement Specialist manually identifies outsourcing needs.

**Clarifications Needed:**
- Are there specific part types always outsourced?
- Are there vendor relationships that determine outsourcing?
- Should the system suggest outsourcing based on part characteristics?
- How is outsourcing status tracked currently?

**Impact:** Affects BOM processing logic and procurement workflow design.

---

### 6. QR Code Content Specification (LOW PRIORITY)

**Question:** What specific data should be encoded in QR codes for assemblies/sub-assemblies?

**Options for Consideration:**
- Link to assembly details page
- Assembly ID and basic information
- Serial number or lot tracking
- Manufacturing date and location

**Clarifications Needed:**
- What information is most valuable in the field?
- Are there size constraints for QR codes?
- Will QR codes be printed on physical labels?
- Should QR codes link to external URLs or contain data directly?

**Impact:** Affects QR code generation system and mobile interface design.

---

### 7. Failure and Rejection Workflows (MEDIUM PRIORITY)

**Question:** What are the specific workflows for handling rejected QC steps or failed tests?

**Context:** PRD doesn't specify what happens when QC fails or tests don't pass.

**Clarifications Needed:**
- Who gets notified when QC fails?
- Can orders be reworked, or must they be scrapped?
- Are there different severity levels for QC failures?
- How is rework tracked and documented?
- Are there approval requirements for continuing after failures?

**Impact:** Affects status tracking, notification system, and workflow logic.

---

## Technical & Integration Questions

### 8. Existing Digital Assets (LOW PRIORITY)

**Question:** Are there existing digital assets (images, technical drawings, work instruction documents) that need to be migrated or linked to the new system?

**Clarifications Needed:**
- Where are current technical drawings stored?
- Are there existing work instruction documents?
- What file formats are used for technical documents?
- Are there existing part photographs?
- Is there a current document management system to integrate with?

**Impact:** Affects data migration planning and storage requirements.

---

### 9. Custom Dimension Validation (LOW PRIORITY)

**Question:** What are the specific validation requirements for custom dimensions beyond basic type checks (e.g., min/max values for custom sink or basin dimensions)?

**Clarifications Needed:**
- What are the physical constraints for custom dimensions?
- Are there manufacturing limitations that should be enforced?
- Should the system warn about non-standard dimensions?
- Are there cost implications for custom dimensions that should be flagged?

**Impact:** Affects form validation and configuration logic.

---

### 10. Notification System Requirements (LOW PRIORITY)

**Question:** What are the specific triggers and content requirements for user notifications?

**Clarifications Needed:**
- Should notifications be in-app only or include email?
- What events require immediate notification?
- Are there escalation procedures for unaddressed notifications?
- Should there be notification preferences per user?
- Are there notification requirements for management oversight?

**Impact:** Affects notification system design and user experience.

---

## Data & System Configuration

### 11. User Account Management (LOW PRIORITY)

**Question:** What are the requirements for user account provisioning, deactivation, and role changes?

**Clarifications Needed:**
- Who has authority to create/modify user accounts?
- Are there approval workflows for user role changes?
- How should inactive users be handled?
- Are there temporary access requirements (contractors, etc.)?
- Should the system track user activity for audit purposes?

**Impact:** Affects user management interface and security design.

---

### 12. System Integration Requirements (MEDIUM PRIORITY)

**Question:** Are there plans to integrate with any existing systems (ERP, accounting, inventory management)?

**Clarifications Needed:**
- Are there existing systems that should share data?
- Are there export requirements for external systems?
- Should the system provide APIs for future integrations?
- Are there specific data formats required for external reporting?

**Impact:** Affects API design and data export capabilities.

---

## Response Requirements

### For Each Question, Please Provide:

1. **Direct Answer:** Clear response to the specific question
2. **Context:** Any additional background information
3. **Constraints:** Technical, regulatory, or business limitations
4. **Preferences:** If multiple options exist, indicate preference
5. **Timeline:** Any urgency or scheduling considerations
6. **Contacts:** Subject matter experts for follow-up questions

### Response Timeline

- **Critical Priority Questions (1-4):** Response within 1 week
- **Medium Priority Questions (5-7, 12):** Response within 2 weeks  
- **Low Priority Questions (8-11):** Response within 3 weeks

### Meeting Request

We recommend scheduling a 2-hour clarification meeting to discuss these questions in detail, particularly the critical priority items related to compliance and QC workflows.

**Suggested Participants:**
- Project stakeholder (Sal)
- QC personnel
- Procurement team member
- Production team representative
- IT/Systems representative (if applicable)

---

## Next Steps

1. **Client Review:** Review all questions and gather internal responses
2. **Clarification Meeting:** Schedule and conduct detailed discussion
3. **Documentation Update:** Update requirements based on responses
4. **Implementation Planning:** Finalize Phase 1 specifications
5. **Development Kickoff:** Begin implementation with confirmed requirements

The quality and completeness of these responses will directly impact the success of the project and the accuracy of the delivered system. Please don't hesitate to reach out for clarification on any of these questions. 