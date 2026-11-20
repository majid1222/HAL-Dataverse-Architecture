# 06. Security, reliability and operations

## 1. Identity and access

I would keep the two identity domains separate:

- Auth0 authenticates external customers. The Dataverse Contact can store the stable external subject identifier needed for correlation, but never a password or credential.
- Entra ID authenticates employees. Dataverse access is controlled through security roles, teams, business units and record ownership.

For server-to-server integration, use an application identity with the minimum Dataverse privileges required. For Azure workloads, prefer managed identities where the target resource supports them. Use Key Vault for secrets or certificates that cannot be eliminated.

## 2. Dataverse authorization

Define access around job responsibilities. For example, a KYB reviewer should only access the records and fields required for review. Separate administrative privileges from day-to-day review access.

Use field-level security for sensitive attributes where necessary, but do not rely on it as the only security boundary. Review table privileges, record ownership, team membership, sharing and export permissions together.

Enable Dataverse auditing for business-critical tables and fields. Confirm audit retention and access with compliance.

## 3. Document security

The Dataverse document metadata record should not expose a public file URL. Document retrieval should go through an authenticated service that checks the requesting user's authorization and returns a short-lived access mechanism where appropriate.

The document service should define:
- Malware scanning before a document is accepted for review.
- Encryption at rest and in transit.
- Retention, legal hold and deletion rules.
- Access logging.
- File size and content-type validation.
- Data residency requirements.

## 4. Integration reliability

Use:
- Unique message IDs and idempotency keys.
- Correlation IDs across API calls and events.
- Bounded retries with exponential backoff and jitter.
- Dead-letter queues and controlled replay.
- Schema versioning and compatibility rules.
- Reconciliation jobs for critical business data.
- Explicit handling of permanent versus transient errors.

A successful message delivery does not automatically mean the business process is complete. Track processing state and business outcome separately.

## 5. Monitoring

I would create dashboards and alerts for:
- APIM request volume, latency, throttling and error rates.
- Azure Function failures and execution duration.
- Service Bus active messages, age of oldest message and dead-letter count.
- Dataverse API throttling and failed writes.
- End-to-end delay from business event to consumer update.
- Data reconciliation differences.
- Document service failures.

Logs should include correlation ID, event ID, source, destination, record reference, processing result and duration. They should not contain access tokens, full KYB documents, bank account details or unnecessary personal data.

## 6. Environment and release management

Use separate development, test / UAT and production environments. Package Dataverse customizations in solutions and promote them through a controlled pipeline. Keep environment-specific endpoints and configuration outside source code.

Manage Azure resources through reviewed infrastructure-as-code where the client has an established standard. Apply connection references, environment variables, managed identities and least-privilege access per environment.

## 7. Recovery and support

Agree recovery objectives with the business. Define who owns Service Bus dead-letter investigation, replay approval, Dataverse data repair and customer communication. Test recovery procedures before go-live rather than treating them as documentation-only controls.
