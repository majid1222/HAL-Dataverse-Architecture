# 01. Solution overview

## 1. Context

HAL is a UAE B2B marketplace and financing platform. Its current Azure-hosted services support company registration, KYB onboarding, marketplace trading and financing applications. Magento provides the storefront. Auth0 authenticates customers, Entra ID authenticates staff, and Azure Service Bus is already used for events.

The business wants to adopt Dynamics 365 / Dataverse as its CRM platform. The key constraint is that customer-facing channels must continue to work.

## 2. My proposed direction

I would introduce Dataverse as the CRM data platform and migrate CRM responsibilities in stages. I would not replace Magento, Auth0, the Financing Portal or every existing microservice as part of this change.

The existing Profile Service can continue to coordinate registration and onboarding for the channels that depend on it. An integration layer will translate between existing service contracts and Dataverse. This gives HAL time to migrate consumers individually rather than force a coordinated rewrite.

## 3. Responsibility boundaries

| Component | Responsibility in the proposed design |
|---|---|
| Dataverse / Dynamics 365 | CRM accounts and contacts, marketplace roles, KYB records and review outcomes, CRM-facing financing summaries |
| Magento | Storefront and authoritative product catalogue |
| Profile Service | Existing registration and onboarding orchestration; gradually delegates CRM persistence to the integration layer / Dataverse |
| Admin Backoffice | Internal user experience for reviewing companies and viewing history; may remain custom or be replaced later by a model-driven app |
| Financing Portal | Loan and account-opening application workflow and partner-bank connectivity |
| Partner banks | Credit assessment and final lending decisions |
| Auth0 | External customer authentication |
| Entra ID | Internal employee authentication |
| Document service / Blob Storage | Document binary storage and controlled retrieval |
| Azure API Management | API gateway, policy enforcement and API façade |
| Azure Functions | Mapping, orchestration, event consumers and integration workers |
| Azure Service Bus | Durable asynchronous messaging between services |
| Application Insights / Azure Monitor | Tracing, metrics, alerting and operational diagnostics |

## 4. System-of-record boundaries

I would avoid describing Dataverse as the owner of every data item. A clear ownership model prevents conflicting updates.

| Data | Authoritative owner | Dataverse role |
|---|---|---|
| Company CRM profile | Dataverse, after migration | Master CRM record |
| Contact CRM profile | Dataverse, after migration | Master CRM record |
| Marketplace buyer / seller role | Dataverse | Master relationship |
| KYB application and internal review | Dataverse, subject to compliance sign-off | Master workflow record |
| Customer credentials and login | Auth0 | Store external subject reference only |
| Employee identity | Entra ID | Use platform identity and security roles |
| Product catalogue | Magento | Store synchronized catalogue reference / copy if needed for CRM |
| Financing application processing | Financing Portal | Store relationship and status summary |
| Lending decision | Relevant partner bank | Store returned decision and reference |
| Document binary | Existing document service or secured object storage | Store metadata, classification and secure reference |

If the existing CRM API remains authoritative during an agreed transition period, that temporary ownership must be documented per entity and have a retirement plan.

## 5. Transition approach

### Phase 1 - Discover and protect existing contracts
- Inventory current CRM API endpoints, events and consumers.
- Capture current schemas, identifiers, validation rules and error behaviours.
- Confirm which service currently owns each field and business process.
- Establish correlation IDs and baseline integration monitoring.

### Phase 2 - Establish Dataverse foundation
- Create the core Account, Contact and marketplace-role model.
- Configure security roles, teams, auditing and environment strategy.
- Define external identifiers and alternate keys.
- Build and test the integration façade.

### Phase 3 - Migrate CRM and KYB responsibilities
- Cleanse and migrate company and contact data.
- Reconcile record counts and key business attributes.
- Move KYB records and review history according to retention rules.
- Route new writes through the agreed integration boundary.

### Phase 4 - Migrate consumers incrementally
- Keep existing API contracts available through the adapter.
- Move one consumer at a time and run contract / regression tests.
- Monitor data reconciliation and message processing.
- Retire old CRM API responsibilities only after consumers have moved and rollback criteria are met.

## 6. Non-functional considerations

I would agree measurable targets with the business before implementation: API latency, event processing delay, availability, recovery time, recovery point, data retention, expected peak volume and support hours. I would not invent service-level targets before those requirements are known.
