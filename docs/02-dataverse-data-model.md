# 02. Dataverse data model

## 1. Modelling approach

I would use the standard Dataverse **Account** and **Contact** tables as the foundation rather than creating separate company records for buyers and sellers.

A company may act as a buyer, a seller or both. Keeping one Account record avoids duplicate company identities. A related custom table represents each marketplace role and its status.

I would use custom tables where HAL has a distinct business lifecycle, such as KYB applications, review decisions and financing summaries. I would not create a custom table for every attribute or every screen.

## 2. Proposed tables

| Table | Type | Purpose / key attributes |
|---|---|---|
| Account | Standard | Legal company profile: legal name, registration number, country, company type, primary address, status |
| Contact | Standard | Individual contact: name, email, phone, job title, preferred language |
| Marketplace Role | Custom | Account lookup, role (Buyer / Seller), status, effective dates, Magento customer/company reference |
| KYB Application | Custom | Account lookup, application number, status, submitted date, current version, risk tier if approved for use |
| KYB Submission | Custom | KYB Application lookup, submission version, submitted by, submitted date, declaration / snapshot reference |
| KYB Review | Custom | Application lookup, reviewer, decision, decision date, reason code, notes |
| KYB Verification | Custom | Application lookup, verification type, provider reference, result, checked date |
| KYB Document | Custom metadata | Application lookup, document type, storage reference, file name, content type, expiry date, verification status |
| Product | Standard or custom | Synchronized catalogue reference; Magento product ID, SKU, name, category reference, active flag |
| Product Category | Standard or custom | Catalogue category reference and hierarchy, if CRM users need category browsing |
| Financing Application Summary | Custom | Account / Contact lookup, external application ID, partner, product type, status, requested amount/currency, submitted date, last status update |
| Lending Partner | Custom | Partner code, name, integration reference, active status |
| Integration Message | Custom operational | External message ID, source, event type, correlation ID, processing status, received / processed timestamps, error reference |
| Integration Error | Custom operational | Related message, error category, retry count, first / last failure, resolution status |

The exact table choices for Product and Product Category depend on how much catalogue interaction CRM users need. If the requirement is only to display a lightweight catalogue copy, custom tables may be more appropriate than enabling unnecessary product capabilities.

## 3. Relationships

- One Account can have many Contacts.
- One Account can have multiple Marketplace Roles, but there should be at most one active record per role unless the business defines a more complex role history.
- One Account can have many KYB Applications over time.
- One KYB Application can have many Submissions, Reviews, Verifications and Documents.
- One Account can have many Financing Application Summaries.
- One Lending Partner can be linked to many Financing Application Summaries.
- Integration Message and Integration Error records provide operational traceability and should not become a substitute for platform telemetry.

See [entity-relationship.mmd](../diagrams/entity-relationship.mmd).

## 4. Important modelling decisions

### Company and contact identity
I would store the existing CRM / marketplace identifiers as external IDs. I would define alternate keys only after confirming the uniqueness rules. A registration number may not be globally unique without a country or issuing-authority context, so the key may need a composite such as country + registration authority + registration number.

### Buyer and seller roles
The Account remains the company identity. Marketplace Role records represent the company's participation. This also allows a company to become a seller later without creating a second Account.

### KYB lifecycle and history
I would keep the KYB Application separate from the Account because one company may submit more than one application over time, and each application has its own status and review lifecycle.

I would preserve submission versions and review decisions. I would not overwrite an old decision when a company resubmits documents. The business should confirm which fields must be immutable and which retention rules apply.

### Documents
The KYB Document table stores metadata and a secure storage reference, not necessarily the binary file. The existing document service or secured object storage remains responsible for file storage, scanning, retention and controlled download.

### Financing
The Financing Application Summary is a CRM-facing representation. The Financing Portal remains responsible for application processing and partner-bank communication. Dataverse should not become a second engine for credit decisions.

### Catalogue
Magento remains the product master. Dataverse receives only the fields required by CRM processes and user experiences. Catalogue synchronization should use Magento IDs as external references and support deactivation / deletion semantics.

## 5. Status design

I would use Dataverse Choice columns for controlled status values and define explicit transition rules. Illustrative KYB statuses:

`Draft -> Submitted -> Under Review -> Approved / Rejected`

A rejected application may be resubmitted as a new submission version or a new application, depending on compliance policy. That rule needs business confirmation.

Marketplace Role statuses could include `Pending`, `Active`, `Suspended` and `Inactive`.

## 6. Data quality and duplicate prevention

- Normalize country, currency and registration identifiers.
- Use external IDs and alternate keys where uniqueness is proven.
- Validate required fields at the API boundary and in Dataverse business logic where appropriate.
- Define duplicate detection for companies and contacts.
- Record source system and last synchronization timestamp where useful.
- Build reconciliation reports for migrated and synchronized records.

## 7. Security considerations for the model

Use business units, owner teams and security roles according to the real access model. Restrict sensitive KYB fields and document references. Use field-level security only where it adds meaningful protection; table-level and record-level access still need to be designed. Enable auditing for business-critical changes and agree audit retention with compliance.
