# HAL – Dynamics 365 & Dataverse Solution Design

**Version:** 1.0  
**Role perspective:** Dynamics 365 CRM / Dataverse Developer  
**Scope:** Entity design and integration architecture

## 1. Assignment understanding

HAL is a UAE-based B2B marketplace and financing platform. Businesses register as buyers and/or sellers, complete KYB onboarding, trade through Magento, and may apply for financing from partner banks.

This repository proposes:
1. A Dataverse data model for companies, contacts, onboarding/KYB, catalog items, and financing applications.
2. An integration architecture that allows the existing customer-facing applications and services to continue working while Dataverse becomes the CRM data platform.

## 2. Repository contents

| Path | Description |
|---|---|
| `docs/01-solution-overview.md` | Scope, design principles, and component responsibilities |
| `docs/02-dataverse-entity-design.md` | Tables, columns, relationships, and ownership |
| `docs/03-integration-architecture.md` | Protocols, data exchanged, and communication patterns |
| `docs/04-assumptions-and-decisions.md` | Explicit assumptions, decisions, and open questions |
| `docs/05-implementation-and-testing.md` | Practical implementation sequence and test scenarios |
| `diagrams/entity-relationship.mmd` | Mermaid entity relationship diagram |
| `diagrams/integration-architecture.mmd` | Mermaid integration architecture diagram |
| `examples/kyb-decision-recorded.json` | Example event payload |

## 3. Design approach

The design uses standard Dataverse capabilities where practical:
- Account for a business/company.
- Contact for a person associated with a company.
- Custom tables for company roles, KYB cases, KYB documents, onboarding steps, catalog items, and financing applications.
- Dataverse security roles, business units/teams, auditing, forms, views, and business process flows where appropriate.
- C# plugins for transactional validations that must run consistently.
- Power Automate for suitable workflow orchestration and notifications.
- Dataverse Web API for synchronous application operations.
- Azure Service Bus for asynchronous integration events.

This is a proposed design, not a claim that the solution has already been built or deployed.

## 4. How to view the diagrams

GitHub renders Mermaid diagrams in Markdown. Open the `.mmd` files or copy their contents into a Markdown file to view them. The diagrams are editable source files.

## 5. Important implementation boundary

The assignment asks for design and diagrams; it does not explicitly request a complete deployable codebase. Therefore, this repository contains design artifacts and an illustrative event payload, not production-ready plugins, flows, or Azure services.

## 6. AI assistance

AI-assisted tools were used to help organize the documentation and refine wording/diagram drafts. The candidate reviewed the proposed design and is responsible for understanding and explaining the entity model, integration choices, assumptions, and trade-offs. Keep this statement only if it accurately describes how the work was prepared.
