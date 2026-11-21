# 1. Solution Overview

## 1.1 Business context

HAL operates a B2B marketplace and financing platform. A business can register as a buyer, a seller, or both. The business completes KYB (Know Your Business) checks before being enabled for relevant marketplace activities. Businesses can trade through Magento and apply for financing through lending partners.

## 1.2 Proposed target state

Dataverse becomes the CRM system of record for:
- Company and contact profile information.
- Buyer/seller roles.
- KYB cases, review status, and onboarding progress.
- A reference copy of Magento product/catalog information needed by CRM users.
- Financing application tracking and lending partner outcomes.

Existing customer-facing channels remain operational. They integrate with Dataverse through the existing CRM API during transition or through the Dataverse Web API where direct integration is appropriate.

## 1.3 Main components

| Component | Responsibility in the proposed design |
|---|---|
| Dataverse / model-driven app | CRM data, internal forms/views, security, business rules, plugin validations, audit |
| CRM API | Compatibility facade for existing consumers during migration; validates/maps requests and calls Dataverse |
| Profile Service | Registration and onboarding orchestration for Magento/customer-facing journeys |
| Magento | Storefront, commerce catalog, and trading experience |
| Admin Backoffice | Internal KYB review experience; can continue using CRM API or move to Dataverse-backed APIs |
| Financing Portal | Captures financing/account-opening applications and displays partner status |
| Document Service | Stores document binaries and provides secure document access |
| Notification / OTP services | Sends email/SMS/OTP using existing services |
| Azure Service Bus | Asynchronous event transport between services |
| Entra ID | Staff identity |
| Auth0 | Customer identity |

## 1.4 Design principles

1. Keep customer-facing channels working during migration.
2. Use Dataverse as the authoritative CRM record for the agreed CRM domain.
3. Do not store large document binaries in Dataverse unless there is a specific requirement; store secure document references and metadata.
4. Use synchronous calls when the user needs an immediate response.
5. Use asynchronous events for downstream notifications and updates that do not need to block the user.
6. Make integrations secure, traceable, retryable, and idempotent.
7. Keep business validations in Dataverse when they must apply regardless of which application creates or updates the record.
