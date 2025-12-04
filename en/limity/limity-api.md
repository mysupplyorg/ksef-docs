## API request limits

22/11/2025

Due to the scale of KSeF's operations and its public nature, mechanisms have been implemented to limit the intensity of API requests. Their purpose is to protect the system's stability, defend against cyber threats, and ensure equal access for all users. These limits define the maximum number of queries that can be executed within a given time period and enforce an integration method that is consistent with the system's architectural assumptions.

### General rules for limits

#### 1. How limits are calculated

All requests to the KSeF API are subject to limits. These limits apply to every call to the protected endpoint. For traffic billing purposes, requests are grouped by context and IP address pair.

- **context** - specified by `ContextIdentifier` ( `Nip` , `InternalId` or `NipVatUe` ) passed during authentication.
- **IP address** - the IP address from which the network connection is established.

Request limits are calculated independently for each context and IP address pair. This means that traffic in the same context but from different IP addresses is billed separately.

Example
 Accounting office A collects invoices on behalf of company B, using company B's context (NIP) and connecting to KSeF from IP address 1. At the same time, company B collects invoices independently, in the same context (its NIP), but from a different IP address – IP 2. Despite the shared context, the different IP addresses cause limits to be calculated independently. In this situation, the system treats each connection as a separate pair (context + IP address) and calculates limits independently: separately for accounting office A and separately for company B.

**Limit units**
 The following notations are used in the limit tables:

- req/s - number of requests per second,
- req/min - number of requests per minute,
- req/h - number of requests per hour.

**Limit calculation model (sliding/rolling window)**
 Limits are enforced using a sliding time window model. At any given time, requests executed during the following period are counted:

- for the req/h threshold - in the last 60 minutes,
- for the req/min threshold - in the last 60 seconds,
- for the req/s threshold - in the last second.

Windows are not aligned to full hours or minutes (they do not reset at :00). All thresholds (req/s, req/min, req/h) apply in parallel - the lock is triggered the first time any of them is exceeded.

#### 2. When the limit is exceeded, the system blocks access to the API

If API request limits are exceeded, HTTP code **429 Too Many Requests** is returned and further requests are temporarily blocked.
 The duration of the block is **dynamic** and depends on the frequency and severity of violations. The exact duration of the block is returned in the `Retry-After` response header (in seconds). Multiple violations may result in a significantly longer block.

Example of a 429 response:

```json
HTTP/1.1 429 Too Many Requests
Content-Type: application/json
Retry-After: 30

{
  "status": {
    "code": 429,
    "description": "Too Many Requests",
    "details": [ "Przekroczono limit 20 żądań na minutę. Spróbuj ponownie po 30 sekundach." ]
  }
}

```

#### 3. Recording of exceedances

All instances of request limit violations are logged and analyzed by security mechanisms. This data is used to monitor API stability and detect potential abuse. The system pays particular attention to patterns indicating attempts to circumvent limits, such as through the parallel and systemic use of multiple IP addresses within a single context. Such actions may be considered a security threat.

In the event of repeated violations or extreme load, the system can automatically apply protective measures such as:

- blocking access to the KSeF API for a given entity or IP address range,
- limiting accessibility to the most demanding contexts.

#### 4. Higher limits during night hours

Higher download limits apply between 8:00 PM and 6:00 AM than during the day. Detailed limits will be determined during the initial phase of KSeF 2.0, after parameters are fine-tuned to actual load conditions.

#### 5. Initial assumptions of limits

API request limits are determined based on anticipated system usage scenarios and workload models.

The actual traffic pattern will depend on how the integration is implemented in external systems and the load patterns they generate. This means that limits established during the design phase may differ from those maintained in a production environment.

Therefore, the limits are dynamic and can be adjusted depending on operating conditions and integrator behavior. In particular, they may be temporarily lowered in cases of intensive or inefficient API use.

### Limits on environments

**TE (Test) Environment:** In the TE environment, limits have been configured to allow integrators to work freely and test integrations without the risk of blockages. Default limit values are **ten times higher** than in production, allowing for intensive testing. Additionally, shared endpoints allow for simulation of various scenarios:

- [POST /testdata/rate-limits/production](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Limity-i-ograniczenia/paths/~1api~1v2~1testdata~1rate-limits~1production/post) - activates limits as in production (PRD),
- [POST /testdata/rate-limits](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Limity-i-ograniczenia/paths/~1api~1v2~1testdata~1rate-limits/post) - allows you to set your own values,
- [DELETE /testdata/rate-limits](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Limity-i-ograniczenia/paths/~1api~1v2~1testdata~1rate-limits/delete) - restores the default TE environment limits.

**DEMO (Pre-Production) Environment: The** DEMO environment has **the same limits as the production environment** for a given context. These values are **replicated from the PRD** and are used for final validation of integration performance and stability before production deployment.

**PRD (Production) Environment:** The PRD environment uses **the default limits** specified in this documentation. In justified cases—e.g., large-scale invoice processing—you can **submit a request to increase these limits** via a dedicated form (in preparation).

## Invoice download limits

### Architectural assumptions

The KSeF invoice retrieval API was designed as a mechanism **for document synchronization** between the central repository and the local database of external systems. The key assumption is that business operations, such as searching, filtering, and reporting, are performed on locally stored data that has been previously synchronized with KSeF. This approach increases operational stability, minimizes the risk of system overload, and allows for more flexible data utilization by client applications.

The KSeF invoice collection API is not intended for real-time, direct end-user interaction. This means it should not be used for:

- downloading individual invoices at the user's request, e.g. invoice preview,
- downloading a list of invoice metadata or initiating the export of batches in response to current actions in the application, except when the user consciously triggers data synchronization.

### Recommended integration method for downloading

The `/invoices/query/metadata` endpoint is used for incremental synchronization. Detailed rules for incremental synchronization are described in a separate document.

Depending on the volume of invoices, different approaches can be used to collect them:

1. **Low volume scenarios** - if the number of invoices is limited and can be handled within the available limits in the expected time, they can be downloaded synchronously by calling `/invoices/ksef/{ksefNumber}` for selected documents.
2. **High-volume scenarios** - if the number of documents is significant and synchronous processing becomes impractical, it is recommended to use the export mechanism ( `/invoices/exports` ). Exporting works asynchronously, is queued, and therefore does not negatively impact system performance.
3. **Business operations** - regardless of the chosen strategy, all user actions (search, filtering, reporting) should be performed on **a local database** , previously synchronized with KSeF.

### Invoice synchronization and download modes

Downloading an invoice to the accounting system can be done in three modes:

1. **At user request** - incremental synchronization is started **manually** by the user, from the last confirmed checkpoint.
2. **Cyclically** - incremental synchronization is performed automatically according to the system schedule.
3. **Mixed mode** - incremental synchronization runs cyclically, and the user can also start it manually on demand.

### Polling frequency

- **High-frequency schedules are not recommended** . In a production environment, the recurring interval should be no less than 15 minutes for each entity on the invoice (Entity 1, Entity 2, Entity 3, Authorized Entity).
- **Low-volume profiles.** On-demand downloads are recommended, supplemented by a cycle, e.g., once a day during the night window.
- **Invoice receipt date. The** invoice receipt date is the date the KSeF number is assigned. This number is assigned automatically by the system when the invoice is processed and is independent of when it is downloaded into the accounting system.

### Examples of not recommended implementation

Improper integration can lead to API lockout. Common errors include:

1. Synchronization is performed solely by downloading individual invoices (synchronous path), without using invoice batch export. This approach is only acceptable for low-volume profiles; for larger document volumes, the `/invoices/exports` mechanism should be used.
2. Handling end-user requests (e.g. displaying the full invoice content in the application, downloading an XML file) via direct KSeF API calls instead of using a local database.

### Detailed limits

Endpoint |  | req/s | req/min | req/h
--- | --- | --- | --- | ---
Downloading the list of invoice metadata | POST /invoices/query/metadata | 8 | 16 | 20
Exporting a batch of invoices | POST /invoices/exports | 4 | 8 | 20
Downloading an invoice by KSeF number | GET /invoices/ksef/{ksefNumber} | 8 | 16 | 64

**Note:** If the available invoice download limits are insufficient in your organization's business scenarios, please contact the KSeF support department for an individual analysis and selection of an appropriate solution.

## Invoice sending limits

### Architectural assumptions

- Invoice shipments, regardless of the shipping type, are queued.
- Processing is optimized to confirm the invoice's accuracy as quickly as possible and return the KSeF number.

#### Batch shipment (invoice packages):

- A batch of invoices is treated as one message in the queue (a reference to the batch instead of separate entries for each invoice) and processed with the same priority as a single document.
- Parcel shipping reduces network and operational overhead because:
    - fewer HTTP requests are made,
    - Content operations (decryption, validation, saving) are performed in batch, which is the most efficient way to handle multiple documents at the same time.
- Batch compression. Due to the XML format and the high repeatability of elements between invoices (consistent structure, similar field names, repeatable blocks), the achieved compression ratio is usually very favorable, significantly reducing data volume and transmission time. In practice, it is faster to transmit a single batch containing, for example, 100 invoices than 100 individual invoices in an interactive session.
- Limits. The limit mechanism operates independently of the sending mode. Batch sending inherently reduces the number of requests and facilitates efficient use of available limits.
- Application: Batch mode is recommended wherever more than one document is submitted within a single operating window. It is particularly useful for recurring customer billing, e-commerce, and automated invoicing processes.

Example scenarios for using batch mode:

- **Online store (e-commerce).** Orders and payments are processed asynchronously, and invoices are issued automatically by the ERP system or invoicing module. A single invoice does not need to be sent to KSeF immediately after issuance. A dedicated process can aggregate issued invoices and periodically—e.g., every 5 minutes—send them in batches to KSeF, significantly reducing the number of HTTP requests and optimizing quota utilization.
- **Subscription services/recurring billing.** Invoices are generated in bulk on a daily or monthly basis (e.g., in telecommunications or utilities) and sent in a single package as part of a scheduled batch session.
- **Automated invoicing processes in enterprises.** These occur, for example, in the distribution, logistics, manufacturing, or B2B services sectors. Invoices are generated automatically based on system events (deliveries, order fulfillment) and sent in bulk, for example, after the completion of an operation.

**Recommendation:** To ensure maximum integration efficiency, it is recommended to aggregate documents into a single batch session wherever business processes allow. This reduces the number of API requests and optimizes the use of available quotas.

**Detailed limits**

Endpoint |  | req/s | req/min | req/h
--- | --- | --- | --- | ---
Opening a batch session * | POST /sessions/batch | 10 | 20 | 120
Closing the batch session | POST /sessions/batch/{referenceNumber}/close | 10 | 20 | 120

**Sending Partial Packages** - Requests sending parts of a package within a single batch session are not subject to API limits. For packages split into multiple parts, it is recommended to send them in parallel (multi-threaded), which significantly reduces delivery times.

#### Interactive Shipping (Single)

Interactive mode is designed for scenarios requiring the rapid registration of individual invoices and immediate acquisition of a KSeF number. Unlike a batch session, each invoice is submitted independently within an active interactive session. Its goal is to minimize the time required to obtain a KSeF number for a single document. It is suitable for low-volume scenarios where individual invoices are submitted.

Example scenarios for using interactive mode:

- **Retail Point of Sale (POS)** . Once the transaction is completed, an invoice is issued and the system immediately records it in the KSeF and returns the KSeF number for printing or presentation to the customer.
- **Mobile applications and lightweight sales systems** that do not have a queuing or buffering mechanism and send invoices immediately after they are issued.
- **Single or irregular events** , e.g. a single corrective invoice.

Interactive mode, despite the higher network overhead associated with larger volumes, is a necessary complement to batch mode in scenarios requiring immediate response or immediate document registration in KSeF. It should only be used where immediate invoice processing is critical to the business process or when the scale of the operation does not justify the use of a batch session.

**Detailed limits**

Endpoint |  | req/s | req/min | req/h
--- | --- | --- | --- | ---
Opening of the interactive session | POST /sessions/online | 10 | 30 | 120
Sending an invoice * | POST /sessions/online/{referenceNumber}/invoices | 10 | 30 | 180
Closing the interactive session | POST /sessions/online/{referenceNumber}/close | 10 | 30 | 120

* **Note:** If your organization's business scenarios regularly result in sending limits being reached in an interactive session, consider using batch mode first, as it allows for more efficient use of available resources and limits. In situations where using an interactive session is essential but the limits being reached are still insufficient, please contact KSeF support for a personalized analysis and assistance in selecting a solution.

### Session and invoice status

**Detailed limits**

Endpoint |  | req/s | req/min | req/h
--- | --- | --- | --- | ---
Retrieving invoice status from session | GET /sessions/{referenceNumber}/invoices/{invoiceReferenceNumber} | 30 | 120 | 720
Downloading the session list | GET /sessions | 5 | 10 | 60
Download session invoices | GET /sessions/{referenceNumber}/invoices | 10 | 20 | 200
Downloading incorrectly processed session invoices | GET /sessions/{referenceNumber}/invoices/failed | 10 | 20 | 200
The remaining | GET /sessions/* | 10 | 120 | 720

## The remaining

Default limits apply to all API resources that are not specifically defined in this documentation. Each such endpoint has its own limit counter, and its requests do not impact other resources.

These limits apply only to protected resources. They do not include public API resources like `/auth/challenge` , which do not require authentication and have their own protection mechanisms – the limit is 60 requests per second per IP address.

Endpoint |  | req/s | req/min | req/h
--- | --- | --- | --- | ---
The remaining | POST/GET /* | 10 | 30 | 120

Related documents:

- [Limits](limity.md)
