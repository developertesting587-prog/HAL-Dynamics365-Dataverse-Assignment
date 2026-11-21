# 2. Dataverse Entity Design

## 2.1 Standard tables

### Account (Company)
Use the standard Account table to represent a registered business.

Suggested columns:
- Account Name
- External Company ID (text; alternate key if source ID is stable and unique)
- Registration Number
- Tax/VAT Number (only if required and appropriately protected)
- Country/Region
- Registered Address
- Company Status
- Onboarding Status
- Magento Customer/Company Reference
- Primary Contact (lookup to Contact)

### Contact (Person)
Use the standard Contact table for individual users associated with a business.

Suggested columns:
- First Name, Last Name
- Email
- Mobile Phone
- Auth0 User ID (external identifier; do not store credentials or tokens)
- Contact Status
- Preferred Language (if required)

A Contact can be associated with an Account using the standard parent customer relationship. If a person can belong to multiple companies, use the Company Contact Role relationship table described below rather than relying only on the single parent customer field.

## 2.2 Custom tables

### Company Role
Purpose: allow one company to act as a Buyer, Seller, or both.

Suggested columns:
- Company (lookup to Account)
- Role Type (Buyer / Seller)
- Status (Pending / Active / Suspended / Inactive)
- Effective From
- Effective To

Relationship: Account 1:N Company Role.

A separate row for each role avoids a single multi-select field and supports role-specific status and dates.

### KYB Case
Purpose: track a company's KYB verification and review lifecycle.

Suggested columns:
- Case Number (autonumber)
- Company (lookup to Account)
- Submitted By (lookup to Contact)
- Case Status (Draft / Submitted / In Review / More Information Required / Approved / Rejected)
- Submission Date
- Decision Date
- Assigned Reviewer (lookup to System User or Team, based on assignment approach)
- Reviewer Comments
- Rejection Reason
- External Reference

Relationship: Account 1:N KYB Case.

Business rule: only an authorized reviewer can record a final approval/rejection. Enforce important state and data validations server-side with a plugin, not only with form JavaScript.

### KYB Document
Purpose: maintain document metadata and its relationship to a KYB case.

Suggested columns:
- KYB Case (lookup to KYB Case)
- Document Type
- Document Name
- Document Status (Uploaded / Pending Review / Accepted / Rejected / Expired)
- Document Service Reference (text)
- File Name
- Uploaded On
- Expiry Date
- Reviewer Comments

Relationship: KYB Case 1:N KYB Document.

Document binary: store it in the existing Document Service or its secure object storage. Dataverse stores metadata and a secure reference. The reference must not be a public unauthenticated URL.

### Onboarding Step
Purpose: track configurable steps in a company's onboarding journey.

Suggested columns:
- Company (lookup to Account)
- KYB Case (optional lookup)
- Step Name
- Step Status (Not Started / In Progress / Completed / Blocked)
- Sequence Number
- Started On
- Completed On
- Failure/Block Reason

Relationship: Account 1:N Onboarding Step; optionally KYB Case 1:N Onboarding Step.

### Product Catalog Item
Purpose: hold the CRM-facing copy of selected Magento catalog data.

Suggested columns:
- External Product ID (alternate key if stable and unique)
- Product Name
- SKU
- Description
- Product Status
- Currency
- Price (only if CRM users need it)
- Last Synced On
- Source System

Magento remains the commerce catalog master unless the business explicitly changes that ownership. Dataverse contains a synchronized reference copy, not a competing catalog master.

### Lending Partner
Purpose: store the list of external banks/financing partners.

Suggested columns:
- Partner Name
- External Partner Code
- Status
- Integration Type / Configuration Reference

Do not store client secrets or API credentials in ordinary Dataverse text fields. Use an appropriate secrets store and managed identity/service authentication where supported.

### Financing Application
Purpose: track a business's financing or account-opening request.

Suggested columns:
- Application Number (autonumber)
- Company (lookup to Account)
- Applicant Contact (lookup to Contact)
- Application Type (Loan / Account Opening)
- Requested Amount
- Currency
- Application Status (Draft / Submitted / Under Review / Approved / Rejected / Withdrawn)
- Submitted On
- External Portal Reference

Relationship: Account 1:N Financing Application.

### Financing Application Partner
Purpose: track the submission and outcome for each lending partner when one application is sent to multiple partners.

Suggested columns:
- Financing Application (lookup)
- Lending Partner (lookup)
- Partner Application Reference
- Submission Status
- Decision Status
- Decision Date
- Response Summary / Error Message

Relationships: Financing Application 1:N Financing Application Partner; Lending Partner 1:N Financing Application Partner.

This junction table supports one application being considered by multiple partners.

### Integration Message (optional operational table)
Purpose: provide application-level tracking for messages when the integration design requires a Dataverse-visible record.

Suggested columns:
- Message ID (unique external identifier)
- Message Type
- Direction
- Related Record Reference
- Processing Status
- Retry Count
- Last Error
- Created On / Processed On

Do not use this table as a replacement for Azure Service Bus. It is optional operational tracking and should be added only if the support team needs it.

## 2.3 Company-contact relationship when multiple companies are allowed

If one Contact may represent more than one Company, add a custom `Company Contact Role` table:
- Company (Account lookup)
- Contact (Contact lookup)
- Role/Job Title
- Is Primary Contact
- Status

This provides an explicit many-to-many relationship with additional attributes.

## 2.4 Relationships summary

| Parent | Relationship | Child |
|---|---|---|
| Account | 1:N | Company Role |
| Account | 1:N | KYB Case |
| KYB Case | 1:N | KYB Document |
| Account | 1:N | Onboarding Step |
| Account | 1:N | Financing Application |
| Financing Application | 1:N | Financing Application Partner |
| Lending Partner | 1:N | Financing Application Partner |
| Account | 1:N | Company Contact Role (if required) |
| Contact | 1:N | Company Contact Role (if required) |

## 2.5 Dataverse configuration and development

- Use solution-aware tables, columns, relationships, forms, views, and security roles.
- Use business rules for simple form-level requirements.
- Use JavaScript only for user-experience behavior; do not rely on it for security or critical validation.
- Use synchronous Pre-Operation plugins for validations or field updates that must happen within the Dataverse transaction.
- Use asynchronous processing for non-blocking downstream work.
- Enable Dataverse auditing for important business tables and sensitive fields according to retention and compliance policy.
- Use teams/security roles to restrict KYB and financing data to authorized staff.
