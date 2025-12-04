## Downloading invoices

### Downloading invoices by KSeF number

21/08/2025

Returns the invoice with the given KSeF number.

GET [/invoices/ksef/{ksefReferenceNumber}](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Pobieranie-faktur/paths/~1api~1v2~1invoices~1ksef~1%7BksefNumber%7D/get)

C# example: [KSeF.Client.Tests.Core\E2E\Invoice\InvoiceE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/Invoice/InvoiceE2ETests.cs)

```csharp
string invoice = await ksefClient.GetInvoiceAsync(ksefReferenceNumber, accessToken, cancellationToken);
```

Java example:

[OnlineSessionIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/OnlineSessionIntegrationTest.java)

```java
byte[] invoice = ksefClient.getInvoice(ksefNumber, accessToken);
```

### Downloading the list of invoice metadata

Returns a list of invoice metadata that match the given search criteria.

**_metadata.json file in export package**
 The export package contains a `_metadata.json` file containing an array of `InvoiceMetadata` objects (the model returned by POST `/invoices/query/metadata` - "Getting invoice metadata").

POST [/invoices/query/metadata](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Pobieranie-faktur/paths/~1api~1v2~1invoices~1query~1metadata/post)

C# example: [KSeF.Client.Tests.Core\E2E\Invoice\InvoiceE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/Invoice/InvoiceE2ETests.cs)

```csharp
InvoiceQueryFilters invoiceQueryFilters = new InvoiceQueryFilters
{
    SubjectType = SubjectType.Subject1,
    DateRange = new DateRange
    {
        From = DateTime.UtcNow.AddDays(-30),
        To = DateTime.UtcNow,
        DateType = DateType.Issue
    }
};

PagedInvoiceResponse pagedInvoicesResponse = await ksefClient.QueryInvoiceMetadataAsync(
    invoiceQueryFilters,
    accessToken,
    pageOffset: 0,
    pageSize: 10,
    cancellationToken);
```

Java example:

[QueryInvoiceIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/QueryInvoiceIntegrationTest.java)

```java
InvoiceQueryFilters request = new InvoiceQueryFiltersBuilder()
        .withSubjectType(InvoiceQuerySubjectType.SUBJECT1)
        .withDateRange(
                new InvoiceQueryDateRange(InvoiceQueryDateType.INVOICING, OffsetDateTime.now().minusYears(1),
                        OffsetDateTime.now()))
        .build();

QueryInvoiceMetadataResponse response = ksefClient.queryInvoiceMetadata(pageOffset, pageSize, SortOrder.ASC, request, accessToken);


```

### Initiates an asynchronous invoice retrieval request

Initiates an asynchronous invoice search in the KSeF system based on the filters provided. Encryption information is required in the `encryption` field, which is used to encrypt generated invoice batches.

POST [/invoices/exports](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Pobieranie-faktur/paths/~1api~1v2~1invoices~1exports/post)

C# example: [KSeF.Client.Tests.Core\E2E\Invoice\InvoiceE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/Invoice/InvoiceE2ETests.cs)

```csharp
EncryptionData encryptionData = CryptographyService.GetEncryptionData();

InvoiceQueryFilters query = new InvoiceQueryFilters
{
    DateRange = new DateRange
    {
        From = DateTime.Now.AddDays(-1),
        To = DateTime.Now.AddDays(1),
        DateType = DateType.Invoicing
    },
    SubjectType = SubjectType.Subject1
};

InvoiceExportRequest invoiceExportRequest = new InvoiceExportRequest
{
    Encryption = encryptionData.EncryptionInfo,
    Filters = query
};

OperationResponse invoicesForSellerResponse = await KsefClient.ExportInvoicesAsync(
    invoiceExportRequest,
    _accessToken,
    CancellationToken);
```

The available `DateType` and `SubjectType` values are described [here](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Pobieranie-faktur/paths/~1api~1v2~1invoices~1query~1metadata/post) .

The invoices in the batch are sorted ascending by the date type specified in `DateRange` when initializing the export.

Java example:

[QueryInvoiceIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/QueryInvoiceIntegrationTest.java)

```java
InvoiceExportFilters filters = new InvoicesAsyncQueryFiltersBuilder()
        .withSubjectType(InvoiceQuerySubjectType.SUBJECT1)
        .withDateRange(
                new InvoiceQueryDateRange(InvoiceQueryDateType.INVOICING, OffsetDateTime.now().minusDays(10), OffsetDateTime.now().plusDays(10)))
        .build();

InvoiceExportRequest request = new InvoiceExportRequest(
        new EncryptionInfo(encryptionData.encryptionInfo().getEncryptedSymmetricKey(),
                encryptionData.encryptionInfo().getInitializationVector()), filters);

InitAsyncInvoicesQueryResponse response = ksefClient.initAsyncQueryInvoice(request, accessToken);

```

### Checks the status of an asynchronous invoice download request

Retrieves the status of a previously initiated asynchronous query based on the operation ID. Allows you to track the query's progress and download completed result packages, if available.

GET [/invoices/exports/{operationReferenceNumber}](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Pobieranie-faktur/paths/~1api~1v2~1invoices~1exports~1%7BoperationReferenceNumber%7D/get)

C# example: [KSeF.Client.Tests.Core\E2E\Invoice\InvoiceE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/Invoice/InvoiceE2ETests.cs)

```csharp
InvoiceExportStatusResponse exportStatus = await KsefClient.GetInvoiceExportStatusAsync(
    exportInvoicesResponse.OperationReferenceNumber,
    accessToken);
```

Java example:

[QueryInvoiceIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/QueryInvoiceIntegrationTest.java)

```java
InvoiceExportStatus response = ksefClient.checkStatusAsyncQueryInvoice(operationReferenceNumber, accessToken);

```

## Related documents

- [Incremental invoice download](przyrostowe-pobieranie-faktur.md)
