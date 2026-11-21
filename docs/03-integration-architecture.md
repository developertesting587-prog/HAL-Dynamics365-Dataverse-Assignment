# 3. Integration Architecture

## 3.1 Overview

The proposed approach keeps the current customer-facing channels working. During the transition, the existing CRM API acts as a compatibility facade for consumers that already depend on it. The CRM API maps requests to Dataverse Web API operations and maps Dataverse responses back to the existing contract.

New or migrated consumers may call the Dataverse Web API through a secured integration layer. Avoid allowing every external system to write directly to Dataverse without a clear API/security boundary.

## 3.2 Communication matrix

| From | To | Protocol / mechanism | Data exchanged | Pattern |
|---|---|---|---|---|
| Magento | Profile Service | HTTPS REST API | Registration, customer/company details, onboarding request | Synchronous request |
| Profile Service | CRM API | HTTPS REST API | Company/contact create or update, buyer/seller role, onboarding status | Synchronous request |
| CRM API | Dataverse | Dataverse Web API (HTTPS/OData) | Account, Contact, Company Role, KYB Case, status | Synchronous request |
| Admin Backoffice | CRM API / Dataverse-backed API | HTTPS REST / Dataverse Web API | KYB review, approval/rejection, comments, audit view | Synchronous request |
| Dataverse integration publisher | Azure Service Bus | Azure Service Bus SDK / supported Dataverse endpoint integration | Company/KYB/financing domain events | Asynchronous |
| Azure Service Bus | Notification service | Service Bus subscription | Event details for email/SMS/notification | Event-driven |
| Financing Portal | Financing integration/API | HTTPS REST API | Financing application and applicant/company references | Synchronous submission |
| Financing integration/API | Dataverse | Dataverse Web API | Financing Application and partner tracking records | Synchronous write or queued processing |
| Financing integration/API | Lending Partners | Partner-specific HTTPS APIs | Application data, documents references, status requests | Synchronous or asynchronous depending on bank API |
| Dataverse / integration service | Document Service | HTTPS REST API | Secure upload/download request and document metadata/reference | Synchronous request |
| Profile Service / integration service | Auth0 | Auth0-supported HTTPS/OAuth flows | Customer identity and authentication operations | Synchronous identity flow |
| Staff users | Admin Backoffice | Entra ID / OpenID Connect | Staff authentication and authorization claims | Synchronous identity flow |
| Magento | Dataverse catalog copy | Through integration service / CRM API | Product ID, SKU, name, description, status, selected price fields | Event-driven or scheduled sync |

The exact partner-bank protocols and Magento event capabilities must be confirmed during discovery. Where a partner only supports batch files or polling, use that supported mechanism instead of assuming REST/webhooks.

## 3.3 Synchronous vs asynchronous

### Synchronous
Use synchronous calls when the calling application needs an immediate result, for example:
- Create/update a company profile.
- Submit an onboarding request and receive a tracking reference.
- Submit a financing application to the platform and receive an accepted/rejected request response.

The API should validate the request, persist the CRM record, and return a stable business identifier. Long-running KYB checks and bank decisions should not hold the HTTP request open.

### Asynchronous
Use events for work that can continue after the original request:
- KYB decision recorded.
- Company onboarding status changed.
- Catalog item changed.
- Financing application status changed.
- Notification requested.

Azure Service Bus provides decoupling between the publisher and consumers. Consumers should be able to retry safely.

## 3.4 Event publishing recommendation

For Dataverse-originated business changes, use a supported asynchronous integration pattern, such as an asynchronous plugin registered to a Dataverse Azure Service Bus endpoint or an integration/outbox component that publishes to Service Bus. Select the exact mechanism based on operational requirements and the team's existing Azure setup.

Do not make a slow external HTTP call from a synchronous plugin transaction. A synchronous plugin should focus on Dataverse validation and data changes.

If an outbox pattern is selected, write an integration message/outbox record in the same Dataverse transaction as the business update, then let an asynchronous processor publish it. The processor must mark success/failure and support retry without publishing duplicate business effects.

## 3.5 Reliability and security

- Use OAuth 2.0 / Microsoft Entra application authentication for service-to-service access to Dataverse where applicable.
- Keep Auth0 for customer identity if that is the existing customer identity provider; do not copy passwords or tokens into Dataverse.
- Use Entra ID for staff authentication as specified in the scenario.
- Apply least-privilege Dataverse application users and security roles.
- Keep secrets in a secure secrets store, not in source code or ordinary table columns.
- Include a unique message/event ID and source record ID in each event.
- Make event consumers idempotent; processing the same event twice must not create duplicate business actions.
- Use retry with backoff for transient failures and dead-letter handling for messages that cannot be processed.
- Log correlation IDs across API calls and Service Bus messages.
- Avoid putting unnecessary personal or sensitive KYB data in event payloads. Send identifiers and the minimum data consumers need.
