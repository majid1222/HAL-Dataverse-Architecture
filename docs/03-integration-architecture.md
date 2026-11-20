# 03. Integration architecture

## 1. Overview

I would use an Azure integration layer between Dataverse and the existing applications. The goal is to preserve existing customer-facing contracts while allowing Dataverse to become the CRM platform.

The main building blocks are:

- **Azure API Management (APIM):** API façade, authentication policy enforcement, throttling and versioned endpoints.
- **Azure Functions:** Stateless integration workers for mapping, validation, orchestration and event handling.
- **Azure Service Bus:** Queues and topics for durable asynchronous communication.
- **Dataverse Web API:** Supported REST interface for Dataverse data operations.
- **Application Insights / Azure Monitor:** End-to-end tracing, metrics, logs and alerts.
- **Key Vault / Managed Identities:** Secret storage and workload identity where supported.

## 2. Communication patterns

| Interaction | Proposed protocol / mechanism | Pattern | Data exchanged |
|---|---|---|---|
| Magento -> existing Profile Service | REST over HTTPS | Synchronous request/response | Registration, company and contact details |
| Profile Service -> integration façade | REST over HTTPS via APIM | Synchronous where immediate validation / acknowledgement is needed | Registration command and external identifiers |
| Integration worker -> Dataverse | Dataverse Web API (HTTPS / OData) | Synchronous call made by the worker | Account, Contact, Marketplace Role and KYB records |
| Dataverse business change -> integration layer | Asynchronous event publication using a supported Dataverse-to-Azure integration pattern | Event-driven | Record reference, event type, version and correlation metadata |
| Integration layer -> Service Bus | AMQP 1.0 | Asynchronous | Versioned domain event / command envelope |
| Service Bus -> Azure Functions | Service Bus trigger | Asynchronous | Event or command message |
| Admin Backoffice -> application / Dataverse | Existing API or Dataverse Web API, depending on portal design | Request/response | Review commands, decisions and read models |
| Financing Portal -> integration façade | REST over HTTPS for commands/status callbacks; Service Bus for asynchronous updates where available | Mixed | Application reference, status, partner reference and decision outcome |
| Magento catalogue -> integration worker | Magento API / webhook if supported by current version and configuration | Event-driven or scheduled incremental sync | Product, SKU, category, active status and changed timestamp |
| Integration worker -> Dataverse | Dataverse Web API | Asynchronous worker processing | Catalogue copy / CRM-required attributes |
| Integration worker -> document service | Existing document API over HTTPS | Request/response or asynchronous, depending on service | Upload / retrieval request and document metadata |

These are proposed choices. The exact Magento API version, existing CRM API contracts and event capabilities must be confirmed during discovery.

## 3. Dataverse event publication

I would not make a synchronous Dataverse plug-in call an external service as part of the user transaction. That couples the user operation to network availability and can make saves slow or fragile.

For changes that must publish an event, I would evaluate a supported asynchronous Dataverse integration mechanism. If the business requires stronger delivery guarantees than the selected mechanism provides, I would use a durable outbox-style design and document the consistency boundary.

The event should contain enough information for the consumer to retrieve the current record, but should not unnecessarily expose sensitive personal or KYB data. A typical envelope includes:

- Event ID
- Event type and schema version
- Occurred-at timestamp
- Source system
- Aggregate / record ID
- Correlation ID
- Causation ID where applicable
- Minimal business payload or record reference

## 4. API design

I would expose business-oriented operations through APIM rather than expose raw Dataverse table operations to every external consumer. The façade can preserve existing contracts and map them to Dataverse.

API design should include:
- Versioned endpoints and documented request / response schemas.
- OAuth 2.0 access tokens and audience / scope validation.
- Input validation and consistent error responses.
- Rate limiting and payload size controls.
- Idempotency keys for retried commands.
- Correlation IDs propagated across services.
- No sensitive values in URLs or routine logs.

## 5. Reliability

- Use bounded retries with exponential backoff and jitter for transient failures.
- Do not retry permanent validation or authorization failures as if they were transient.
- Use dead-letter queues for messages that exceed retry policy.
- Make consumers idempotent using event IDs or business operation keys.
- Handle out-of-order events using version / timestamp checks where the business process requires it.
- Provide a controlled replay process with audit and operator permissions.
- Run reconciliation jobs to identify missed or inconsistent updates.

## 6. Security

- Use managed identities for Azure workloads when the target service supports them.
- Use an appropriately governed Dataverse application user for server-to-server access.
- Keep secrets and certificates in Key Vault; rotate them.
- Keep Auth0 for customer sign-in and Entra ID for staff sign-in.
- Apply least privilege at API, Azure resource and Dataverse layers.
- Encrypt data in transit and at rest.
- Mask personal and KYB information in telemetry.

## 7. Observability

Every request and message should carry a correlation ID. Capture event ID, source, destination, entity / record reference, processing duration, attempt count and outcome. Do not log full identity documents, access tokens, bank account details or unnecessary personal data.

Monitor API error rate and latency, Service Bus active / dead-letter message counts, Function failures, Dataverse throttling, integration lag and reconciliation differences. Agree thresholds with operations before go-live.
