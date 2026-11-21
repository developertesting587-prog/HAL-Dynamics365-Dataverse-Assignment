# 5. Practical Implementation and Testing Plan

## 5.1 Implementation sequence

1. Confirm the data ownership, company/contact rules, KYB lifecycle, financing lifecycle, and integration contracts.
2. Create a Dataverse solution and configure Account/Contact extensions and custom tables.
3. Create relationships, alternate keys for stable external IDs, forms, views, and status choices.
4. Configure security roles and teams for onboarding staff, KYB reviewers, finance staff, and administrators.
5. Enable auditing for selected tables and sensitive fields based on client policy.
6. Implement server-side plugin validations for critical rules, such as preventing an invalid KYB state transition or duplicate active role where the business rule requires it.
7. Configure Power Automate for appropriate notifications and approval tasks. Keep long-running or high-volume integration processing in a dedicated integration component where needed.
8. Integrate CRM API with Dataverse Web API and preserve existing request/response contracts during transition.
9. Configure event publishing and Service Bus consumers with retry, dead-letter, idempotency, and correlation.
10. Integrate Document Service and financing partner interfaces.
11. Test, deploy through DEV → TEST/UAT → PROD, and monitor after release.

## 5.2 Example business validations

- A company cannot be marked KYB Approved unless required documents have been accepted and mandatory company data is present.
- Only authorized reviewers can approve or reject a KYB case.
- A rejected case must have a rejection reason.
- A company must have an active Buyer or Seller role before the corresponding marketplace capability is enabled.
- An external company/product/application ID must not create duplicate records when the same integration request is retried.
- A financing partner response must update the matching Financing Application Partner row, not create a new application.

The exact rules must be confirmed with the business before implementation.

## 5.3 Test scenarios

| # | Scenario | Expected result |
|---|---|---|
| 1 | Create a company from Profile Service | One Account is created and external ID is stored |
| 2 | Retry the same company request | Existing Account is updated or returned; no duplicate |
| 3 | Assign Buyer and Seller roles | Two Company Role records can exist for one company |
| 4 | Submit KYB case without required data | Submission is blocked with a clear validation message |
| 5 | Reviewer approves a complete KYB case | Status changes to Approved and decision details are recorded |
| 6 | Unauthorized user attempts approval | Operation is denied |
| 7 | Upload KYB document | Document binary is stored in Document Service; metadata/reference is saved in Dataverse |
| 8 | Magento catalog update arrives twice | Catalog copy remains consistent without duplicate product rows |
| 9 | Submit financing application to two partners | One Financing Application and two partner tracking records are created |
| 10 | Partner response arrives | Correct partner tracking row and application summary are updated |
| 11 | Service Bus consumer temporarily fails | Message is retried and eventually processed or dead-lettered |
| 12 | Same event is delivered twice | Consumer detects duplicate event ID and avoids duplicate business effects |
| 13 | Staff user opens KYB record | Access is limited according to assigned role/team |
| 14 | Audit review | Authorized staff can review relevant Dataverse audit history |

## 5.4 Deployment and support

Use solution-aware development and managed deployments across environments. Keep environment-specific URLs and configuration outside code. Use Azure DevOps or the client's approved pipeline for deployment, and validate plugin steps, connection references, environment variables, security roles, and integration endpoints after each import.

Monitor API failures, plugin exceptions, flow failures, Service Bus dead-letter messages, and integration latency. Provide a correlation ID so support teams can trace a business operation across services.
