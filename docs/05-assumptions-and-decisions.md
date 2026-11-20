# 05. Assumptions and architecture decisions

The assignment leaves several implementation details open. I have made the following assumptions so the proposal can be evaluated. I would validate them with the client before implementation.

## 1. Assumptions

| ID | Assumption | Why it matters / validation needed |
|---|---|---|
| A-01 | Dataverse will progressively replace the CRM API as the authoritative CRM data platform. | Confirm migration scope, cutover sequence and retirement plan. |
| A-02 | Existing customer-facing applications must continue operating during migration. | Confirm existing API contracts and compatibility requirements. |
| A-03 | Profile Service remains in place initially. | Confirm whether it will continue orchestration or be retired later. |
| A-04 | Magento remains the storefront and product catalogue master. | Confirm Magento edition, API version, webhook capability and catalogue ownership. |
| A-05 | Auth0 remains the external customer identity provider. | Confirm token claims and stable Auth0 subject identifier. |
| A-06 | Entra ID remains the internal staff identity provider. | Confirm tenant, conditional access and staff provisioning model. |
| A-07 | The Financing Portal remains responsible for partner-bank workflows. | Confirm application and status contracts with each lending partner. |
| A-08 | Document binaries remain in the existing document service or secured object storage. | Confirm data residency, malware scanning, retention and access requirements. |
| A-09 | Azure Service Bus remains the messaging platform. | Confirm current topology, tier, throughput and operational ownership. |
| A-10 | Dataverse is used for CRM-facing financing visibility, not credit decision processing. | Confirm which financing attributes staff need to see. |

## 2. Key decisions

### ADR-01: Keep Account as the company identity
**Decision:** Use standard Account and Contact tables and represent buyer / seller participation through a related Marketplace Role table.

**Reason:** A company can participate in more than one marketplace role. One company record avoids duplicate identity data.

**Trade-off:** The role table adds a relationship and requires validation of allowed role combinations and active-state rules.

### ADR-02: Keep external authentication separate
**Decision:** Retain Auth0 for customers and Entra ID for staff.

**Reason:** The scenario explicitly identifies these providers. CRM profile records should not become authentication records.

**Trade-off:** Identity correlation and lifecycle synchronization must be designed carefully.

### ADR-03: Use asynchronous events for cross-system business changes
**Decision:** Use Service Bus for decoupled business events and REST for operations that need an immediate response.

**Reason:** This reduces runtime coupling and allows consumers to process work independently.

**Trade-off:** Consumers must handle eventual consistency, duplicate delivery and out-of-order events.

### ADR-04: Keep document binaries outside Dataverse by default
**Decision:** Store document metadata and secure references in Dataverse; keep binaries in the document service / secured object storage.

**Reason:** The existing platform already has document services, and KYB files may be sensitive or large.

**Trade-off:** Document access requires a secure integration and a clear retention / deletion process.

### ADR-05: Preserve API contracts during transition
**Decision:** Introduce an adapter / façade and migrate consumers incrementally.

**Reason:** This protects customer-facing channels and reduces the risk of a big-bang cutover.

**Trade-off:** The adapter is an additional component that must be monitored and eventually simplified or retired.

## 3. Decisions still requiring client confirmation

- Which existing CRM API endpoints must remain backward-compatible?
- Is the Admin Backoffice staying custom, or is a model-driven Power App a target?
- What is the exact KYB state machine, including resubmission and appeal?
- Which countries and registration authorities are in scope?
- What data may be exposed to each lending partner?
- What are the data residency and retention requirements?
- What are expected peak API and event volumes?
- What availability, latency, RTO and RPO targets apply?
- Which Magento events and APIs are available in the current deployment?
- Is dual-running required, and what are the cutover / rollback criteria?
