# HAL | Dynamics 365 and Dataverse Architecture

**Assignment:** Principal / Lead Developer (Dynamics 365 & Dataverse)  
**Candidate:** Majid Asgari  
**Status:** Architecture proposal for review

## 1. My understanding of the problem

HAL already operates a B2B marketplace and financing platform on Azure. Businesses register, complete KYB onboarding as buyers and/or sellers, trade through Magento, and can apply for financing through partner banks.

The goal is to adopt Dynamics 365 / Dataverse as the CRM platform without interrupting the existing customer-facing channels. I have therefore designed this as an incremental transition, not a replacement of every existing application.

My proposal puts Dataverse at the centre of CRM and KYB information. Existing channels and specialist services continue to do the jobs they already do. An Azure integration layer protects their contracts and handles communication with Dataverse.

## 2. Proposed architecture at a glance

- **Dataverse:** CRM accounts, contacts, marketplace roles, KYB applications and reviews, and CRM-facing financing summaries.
- **Magento:** Continues to own the storefront and product catalogue.
- **Auth0:** Continues to authenticate external customers.
- **Microsoft Entra ID:** Continues to authenticate internal staff.
- **Profile Service:** Continues to orchestrate registration and onboarding for existing channels.
- **Financing Portal and partner banks:** Continue to own financing application processing and lending decisions.
- **Azure integration services:** API Management, Azure Functions and Service Bus provide controlled API access, transformation and asynchronous messaging.
- **Document service / secure object storage:** Retains document binaries; Dataverse stores metadata and secure references.

See [Integration architecture](diagrams/integration-architecture.mmd) and [Entity relationship model](diagrams/entity-relationship.mmd).

## 3. Design principles

1. Keep existing customer-facing channels working during the transition.
2. Give each business data set one authoritative owner.
3. Keep authentication identities separate from CRM records.
4. Use synchronous APIs only when the caller needs an immediate answer.
5. Use asynchronous messaging for business events and work that can complete later.
6. Make event handling idempotent and observable.
7. Store sensitive document binaries outside Dataverse unless a confirmed requirement justifies otherwise.
8. Migrate in stages and avoid uncontrolled dual-writing.

## 4. Documents

| Document | Purpose |
|---|---|
| [Solution overview](docs/01-solution-overview.md) | Scope, architecture and migration approach |
| [Dataverse data model](docs/02-dataverse-data-model.md) | Tables, relationships and modelling decisions |
| [Integration architecture](docs/03-integration-architecture.md) | Components, protocols and communication patterns |
| [Integration flows](docs/04-integration-flows.md) | Registration, KYB, catalogue and financing flows |
| [Assumptions and decisions](docs/05-assumptions-and-decisions.md) | Explicit assumptions and trade-offs |
| [Security and operations](docs/06-security-and-operations.md) | Identity, access, reliability and monitoring |

## 5. Important scope boundary

This repository is an architecture and data-model proposal. It does not claim to contain a deployed Dataverse solution, working plugins, Azure Functions, API definitions or infrastructure-as-code. Those would be implementation deliverables after the design is agreed.

## 6. Review points

Before implementation, I would validate the current CRM API contracts, the exact Magento integration capabilities, the existing document service, the financing portal's data contract, data volumes, regulatory retention requirements and the required availability / recovery objectives.

## 7. Diagram rendering

The diagrams are written in Mermaid. GitHub can render Mermaid diagrams directly in Markdown when embedded in a Markdown file. The `.mmd` files are also available for editing or importing into compatible diagram tools.
