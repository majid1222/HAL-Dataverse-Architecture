# 04. Main integration flows

## 1. Company registration

1. A customer signs in through Auth0 and starts registration through the existing Magento / customer channel.
2. Magento sends the registration request to the Profile Service using the existing REST contract.
3. The Profile Service validates the request and calls the integration façade.
4. The integration layer maps the request to Dataverse Account, Contact and Marketplace Role records.
5. Dataverse returns the created / matched record references.
6. The Profile Service returns the result to the customer channel.
7. Any downstream work that does not need to block the response is published asynchronously.

I would use an idempotency key so a customer retry does not create duplicate company or contact records. The exact point at which registration is considered complete must be agreed with the business.

## 2. KYB submission and review

1. The customer submits company information and document metadata through the existing channel.
2. The document service stores the binary file and returns a secure reference.
3. The integration layer creates or updates the KYB Application and KYB Submission records in Dataverse.
4. The Admin Backoffice presents the application to an authorized reviewer.
5. The reviewer records a decision in Dataverse, including reviewer, timestamp, decision and reason.
6. The status change is published asynchronously.
7. The Profile Service and relevant customer-facing channel receive the status update through the agreed integration contract.
8. Notification services send the customer an appropriate message.

The workflow must preserve review history. A resubmission should not erase the previous submission or decision.

## 3. Product catalogue synchronization

1. Magento remains the catalogue master.
2. A webhook or scheduled incremental process identifies changed products and categories.
3. An Azure Function retrieves the required catalogue fields from Magento.
4. The Function maps Magento identifiers to Dataverse external keys.
5. The Function upserts the CRM catalogue copy through the Dataverse Web API.
6. Deactivated or removed products are handled according to an agreed lifecycle rule.

I would not synchronize every Magento field by default. Only fields required by CRM users and business processes should be copied.

## 4. Financing application status

1. The customer applies through the Financing Portal.
2. The Financing Portal validates the request and manages communication with the selected lending partner.
3. Dataverse receives a CRM-facing application summary and external application reference.
4. Partner status changes are processed by the Financing Portal.
5. The Financing Portal sends status changes through a secured API callback or asynchronous event.
6. The integration layer updates the Dataverse summary.
7. The customer-facing portal remains the channel for detailed application processing; Dataverse provides relationship visibility.

The bank remains authoritative for its lending decision. Dataverse should record the returned outcome, not calculate or override it.

## 5. Failure and recovery example

If a KYB approval event cannot be delivered to a downstream consumer:

1. The message remains available for retry according to the configured policy.
2. The consumer checks the event ID to avoid processing the same event twice.
3. If retries are exhausted, the message is moved to a dead-letter queue.
4. Operations investigates the failure using the correlation ID and error details.
5. After the cause is corrected, an authorized operator replays the message.
6. A reconciliation process confirms that the downstream status matches the Dataverse business state.

The replay process must be safe to run more than once.
