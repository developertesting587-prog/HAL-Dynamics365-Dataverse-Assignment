# 4. Assumptions and Design Decisions

The assignment asks the candidate to make and state assumptions. The following assumptions are used for this proposal.

1. Dataverse becomes the system of record for CRM-owned company, contact, onboarding, KYB, and financing tracking data.
2. Magento remains the master for commerce catalog and storefront operations. Dataverse holds a synchronized copy of selected catalog fields for CRM visibility.
3. A company can be a Buyer, a Seller, or both. Company Role is therefore a separate child table.
4. A company can have multiple contacts. If one contact can work with multiple companies, Company Contact Role provides the many-to-many association.
5. A company may have more than one KYB case over time to support resubmission, renewal, or periodic review.
6. KYB documents are stored in the existing Document Service or secure object storage. Dataverse stores document metadata and secure references.
7. A financing application may be submitted to more than one lending partner. Financing Application Partner tracks each partner submission and result.
8. Auth0 remains the customer identity provider and Entra ID remains the staff identity provider, as stated in the assignment.
9. Existing customer-facing applications must continue to work during migration. The existing CRM API is retained initially as a compatibility facade.
10. Partner bank APIs and Magento event capabilities are not specified. Their exact protocols, authentication, rate limits, and callback/polling options must be confirmed.
11. Data residency, retention, KYB regulatory requirements, and UAE privacy/compliance requirements must be validated with the client before implementation.
12. Dataverse licensing, API limits, storage capacity, and expected transaction volumes must be assessed before final sizing.

## Key decisions

| Decision | Reason |
|---|---|
| Use Account and Contact standard tables | Fits company/person CRM concepts and standard Dataverse functionality |
| Use a Company Role child table | Supports Buyer and Seller roles independently, including separate status |
| Keep document binary outside Dataverse by default | Avoids unnecessary Dataverse file storage and reuses the existing Document Service |
| Use a junction table for application-partner submissions | Supports multiple lending partners per financing application |
| Retain CRM API during transition | Avoids breaking existing consumers and allows gradual migration |
| Use Service Bus for asynchronous events | Decouples services and supports retries/subscriptions |
| Keep critical validation in server-side plugins | Applies validation regardless of whether the change comes from a form or API |

## Questions to confirm with the client

- Is the CRM API expected to be retired, or will it remain the long-term integration facade?
- Which system owns company master data during and after migration?
- Can a contact be associated with multiple companies?
- What are the KYB document types, expiry rules, and reviewer approval levels?
- Can a financing application be routed to multiple banks simultaneously or sequentially?
- Which Magento version and event/webhook mechanisms are available?
- Which APIs or message formats are supported by each lending partner?
- What are expected volumes, availability targets, retention periods, and audit requirements?
