# Incremental invoice download

21/11/2025

## Introduction

Incremental invoice downloading, based on package export (POST [`/invoice/exports`](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Pobieranie-faktur/paths/~1api~1v2~1invoices~1exports/post) ), is the recommended mechanism for synchronization between the central KSeF repository and local databases of external systems.

**[The High Water Mark (HWM)](hwm.md)** mechanism plays a key role here - a stable point in time to which the system guarantees data completeness.

## Solution architecture

Incremental downloading relies on three key components:

1. **Synchronization in time windows** - the use of adjacent time windows taking into account HWM, which ensures continuity and no omissions
2. **API limits support** - call rate control, HTTP 429 and Retry-After support.
3. **Deduplication** - eliminating duplicates based on metadata from `_metadata.json` files.

The underlying method: POST [`/invoice/exports`](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Pobieranie-faktur/paths/~1api~1v2~1invoices~1exports/post) initiates an asynchronous export. Once processing is complete, the operation status provides unique URLs for downloading portions of the package.

## Synchronization in time windows (Windowing)

### Concept

Invoice retrieval occurs in adjacent time windows using the `PermanentStorageHwmDate` date. To enable the HWM mechanism, set the `restrictToPermanentStorageHwmDate = true` in the export request. Each subsequent window starts exactly at the end of the previous one, taking into account the HWM (except as described in [the High Water Mark (HWM) mechanism and handling truncated batches](#mechanizm-high-water-mark-hwm-i-obs%C5%82uga-obci%C4%99tych-paczek-istruncated) section). By "end moment" we mean:

- `dateRange.to` value, when provided, or
- `PermanentStorageHwmDate` when `dateRange.to` omitted.

This approach ensures continuity of scopes and eliminates the risk of omitting any invoice. Invoices should be retrieved separately for each entity type ( `Podmiot 1` , `Podmiot 2` , `Podmiot 3` , `Podmiot upoważniony` ) appearing in the document. Iterating through entities ensures data completeness – a company can appear in different roles on invoices.

`C#` example: [KSeF.Client.Tests.Core\E2E\Invoice\IncrementalInvoiceRetrievalE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/Invoice/IncrementalInvoiceRetrievalE2ETests.cs)

```csharp
// Słownik do śledzenia punktu kontynuacji dla każdego SubjectType
Dictionary<SubjectType, DateTime?> continuationPoints = new();
IReadOnlyList<(DateTime From, DateTime To)> windows = BuildIncrementalWindows(batchCreationStart, batchCreationCompleted);

// Tworzenie planu eksportu - krotki (okno czasowe, typ podmiotu)
IEnumerable<SubjectType> subjectTypes = Enum.GetValues<SubjectType>().Where(x => x != SubjectType.SubjectAuthorized);
IOrderedEnumerable<ExportTask> exportTasks = windows
    .SelectMany(window => subjectTypes, (window, subjectType) => new ExportTask(window.From, window.To, subjectType))
    .OrderBy(task => task.From)
    .ThenBy(task => task.SubjectType);


foreach (ExportTask task in exportTasks)
{
    DateTime effectiveFrom = GetEffectiveStartDate(continuationPoints, task.SubjectType, task.From);

    OperationResponse? exportResponse = await InitiateInvoiceExportAsync(effectiveFrom, task.To, task.SubjectType);

    // Dalsza obsługa eksportu...
```

`java` example: [IncrementalInvoiceRetrieveIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/IncrementalInvoiceRetrieveIntegrationTest.java)

```java
Map<InvoiceQuerySubjectType, OffsetDateTime> continuationPoints = new HashMap<>();

List<TimeWindows> timeWindows = buildIncrementalWindows(batchCreationStart, batchCreationCompleted);
List<InvoiceQuerySubjectType> subjectTypes = Arrays.stream(InvoiceQuerySubjectType.values())
        .filter(x -> x != InvoiceQuerySubjectType.SUBJECTAUTHORIZED)
        .toList();

List<ExportTask> exportTasks = timeWindows.stream()
        .flatMap(window -> subjectTypes.stream()
                .map(subjectType -> new ExportTask(window.getFrom(), window.getTo(), subjectType)))
        .sorted(Comparator.comparing(ExportTask::getFrom)
                .thenComparing(ExportTask::getSubjectType))
        .toList();
exportTasks.forEach(task -> {
        EncryptionData encryptionData = defaultCryptographyService.getEncryptionData();
        OffsetDateTime effectiveFrom = getEffectiveStartDate(continuationPoints, task.getSubjectType(), task.getFrom());
        String operationReferenceNumber = initiateInvoiceExportAsync(effectiveFrom, task.getTo(),
            task.getSubjectType(), accessToken, encryptionData.encryptionInfo());

// Dalsza obsługa eksportu...
```

### Recommended window sizes

- **Frequency and limits**
     POST `/invoice/exports` requires an entity type ( `Podmiot 1` , `Podmiot 2` , `Podmiot 3` , `Podmiot upoważniony` ). [API limits](../limity/limity-api.md) allow for a maximum of 20 exports per hour; the scheduler should divide this pool among the selected entity types.
- **Schedule Strategy**
     In continuous synchronization mode, 4 exports/hour can be assumed for each entity type. In practice, `Podmiot 3` and `Podmiot upoważniony` roles typically occur less frequently and may be run sporadically, e.g., once a day during the night window.
- **Minimum interval**
     The recurring interval should not be less than 15 minutes for each entity type (as recommended in the API limits).
- **Window Size** In a continuous synchronization scenario, it is recommended to call export without a specified end date ( `DateRange.To` omitted). In this case, the KSeF system prepares the largest possible, consistent package within the algorithm limits (number of invoices, data size after compression), which limits the number of calls and the load on both sides. When `IsTruncated = true` , the next call should start from `LastPermanentStorageDate` , when `IsTruncated = false` , the next call should start from the returned `PermanentStorageHwmDate` .
- **No Overlap** Ranges must be contiguous; the end of one window is the beginning of the next.
- **Checkpoint** The continuation point determined by HWM - `PermanentStorageHwmDate` or `LastPermanentStorageDate` for truncated packages is the start of the next window.

> The invoice receipt date is the date the KSeF number is assigned. The number is assigned during invoice processing on the KSeF side and is independent of the time of download to the external system.

## API Limits Support

### Limits by endpoint type

All invoice fetching endpoints are subject to strict API limits specified in the [API Limits](../limity/limity-api.md) documentation. These limits are binding and must be respected by any incremental fetching implementation.

If the limits are exceeded, the system returns HTTP code `429` (Too Many Requests) along with a `Retry-After` header indicating the waiting time before the next attempt.

## Initiating invoice export

### The key importance of the PermanentStorage date

For incremental invoice retrieval, it's **necessary** to use the `PermanentStorage` date type, which ensures data reliability. It marks the point in time when a record is permanently materialized, is resistant to asynchronous delays in the data retrieval process, and allows for safe incremental windowing. Therefore, other date types (such as `Issue` or `Invoicing` ) can lead to unpredictable behavior in incremental synchronization.

`C#` example: [KSeF.Client.Tests.Core\E2E\Invoice\IncrementalInvoiceRetrievalE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/Invoice/IncrementalInvoiceRetrievalE2ETests.cs)

```csharp
EncryptionData exportEncryption = CryptographyService.GetEncryptionData();

InvoiceQueryFilters filters = new()
{
    SubjectType = subjectType,
    DateRange = new DateRange
    {
        DateType = DateType.PermanentStorage,
        From = windowFromUtc,
        To = windowToUtc,
        RestrictToPermanentStorageHwmDate = true
    }
};

InvoiceExportRequest request = new()
{
    Filters = filters,
    Encryption = exportEncryption.EncryptionInfo
};

OperationResponse response = await KsefRateLimitWrapper.ExecuteWithRetryAsync(
    ksefApiCall: ct => KsefClient.ExportInvoicesAsync(request, _accessToken, ct, includeMetadata: true),
    endpoint: KsefApiEndpoint.InvoiceExport,
    cancellationToken: CancellationToken);
```

`java` example: [IncrementalInvoiceRetrieveIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/IncrementalInvoiceRetrieveIntegrationTest.java)

```java
EncryptionData encryptionData = defaultCryptographyService.getEncryptionData();
InvoiceExportFilters filters = new InvoiceExportFilters();
filters.setSubjectType(subjectType);
filters.setDateRange(new InvoiceQueryDateRange(
        InvoiceQueryDateType.PERMANENTSTORAGE, windowFrom, windowTo)
);

InvoiceExportRequest request = new InvoiceExportRequest();
request.setFilters(filters);
request.setEncryption(encryptionInfo);

InitAsyncInvoicesQueryResponse response = ksefClient.initAsyncQueryInvoice(request, accessToken);
```

## Parcel collection and processing

Once the export is complete, the invoice batch is available for download as an encrypted ZIP archive divided into sections. The download and processing process includes:

1. **Part download** - each part downloaded separately from the URLs returned in the operation status.
2. **AES-256 Decryption** - Each part is decrypted using the key and IV generated during export initialization.
3. **Packet assembly** - decrypted parts combined into one data stream.
4. **Unpacking the ZIP** archive contains invoice XML files and the `_metadata.json` file.

### _metadata.json file

The contents of the _metadata.json file are a JSON object with `invoices` property (an array of `InvoiceMetadata` elements, as returned by POST `/invoices/query/metadata` ). This file is crucial to the deduplication mechanism because it contains the KSeF numbers of all invoices in the batch.

**Metadata enablement (until 27/10/2025)**
 To include the `_metadata.json` file, you need to add a header to your export request:

```http
X-KSeF-Feature: include-metadata
```

**From 27/10/2025,** the export package will always include the `_metadata.json` file without the need to add a header.

Example in `C#` :

[KSeF.Client.Tests.Core\E2E\Invoice\IncrementalInvoiceRetrievalE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/Invoice/IncrementalInvoiceRetrievalE2ETests.cs)

[KSeF.Client.Tests.Utils\BatchSessionUtils.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Utils/BatchSessionUtils.cs)

```csharp
List<InvoiceSummary> metadataSummaries = new();
Dictionary<string, string> invoiceXmlFiles = new(StringComparer.OrdinalIgnoreCase);

// Pobranie, odszyfrowanie i połączenie wszystkich części w jeden strumień
using MemoryStream decryptedArchiveStream = await BatchUtils.DownloadAndDecryptPackagePartsAsync(
    package.Parts,
    encryptionData,
    CryptographyService,
    cancellationToken: CancellationToken);

// Rozpakowanie ZIP
Dictionary<string, string> unzippedFiles = await BatchUtils.UnzipAsync(decryptedArchiveStream, CancellationToken);

foreach ((string fileName, string content) in unzippedFiles)
{
    if (fileName.Equals(MetadataEntryName, StringComparison.OrdinalIgnoreCase))
    {
        InvoicePackageMetadata? metadata = JsonSerializer.Deserialize<InvoicePackageMetadata>(content, MetadataSerializerOptions);
        if (metadata?.Invoices != null)
        {
            metadataSummaries.AddRange(metadata.Invoices);
        }
    }
    else if (fileName.EndsWith(XmlFileExtension, StringComparison.OrdinalIgnoreCase))
    {
        invoiceXmlFiles[fileName] = content;
    }
}
```

`java` example: [IncrementalInvoiceRetrieveIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/IncrementalInvoiceRetrieveIntegrationTest.java)

```java
 List<InvoicePackagePart> parts = invoiceExportStatus.getPackageParts().getParts();
byte[] mergedZip = FilesUtil.mergeZipParts(
        encryptionData,
        parts,
        part -> ksefClient.downloadPackagePart(part),
        (encryptedPackagePart, key, iv) -> defaultCryptographyService.decryptBytesWithAes256(encryptedPackagePart, key, iv)
);
Map<String, String> downloadedFiles = FilesUtil.unzip(mergedZip);

String metadataJson = downloadedFiles.keySet()
        .stream()
        .filter(fileName -> fileName.endsWith(".json"))
        .findFirst()
        .map(downloadedFiles::get)
        .orElse(null);
InvoicePackageMetadata invoicePackageMetadata = objectMapper.readValue(metadataJson, InvoicePackageMetadata.class);
```

## High Water Mark (HWM) and Truncated Packet Handling (IsTruncated)

### HWM Concept

High Water Mark (HWM) is a mechanism that ensures optimal management of starting points for subsequent exports in incremental invoice collection. HWM consists of two complementary components:

- **`PermanentStorageHwmDate`** - stable upper limit of data included in the package, representing the point in time until which the system guarantees data completeness.
- **`LastPermanentStorageDate`** - date of the last invoice in the package, used when the package has been truncated ( `IsTruncated = true` ).

#### Benefits of the HWM mechanism

- **Minimizing duplicates** - HWM significantly reduces the number of duplicates between successive packages
- **Performance optimization** - reduces the load on the deduplication mechanism
- **Maintaining completeness** - ensures no invoices are missed
- **Synchronization stability** - provides reliable continuation points for long-running processes

### Package Continuation Strategy

The `IsTruncated = true` flag is set when the algorithm limits (number of invoices or data size after compression) have been reached during batch construction. In this case, both HWM properties are available in the operation status. The HWM mechanism uses the following priority hierarchy to determine the continuation point. To maintain download continuity and not skip any documents, the next export call should begin with:

1. **Truncated parcel** ( `IsTruncated = true` ) - next call starts with `LastPermanentStorageDate`
2. **Stable HWM** - using `PermanentStorageHwmDate` as starting point for next window

- the next window starts at the same point as the last one (adjacency); any duplicates will be removed during the deduplication step based on the KSeF numbers from _metadata.json. Below is an example of maintaining a continuation point:

`C#` example: [KSeF.Client.Tests.Core\E2E\Invoice\IncrementalInvoiceRetrievalE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/Invoice/IncrementalInvoiceRetrievalE2ETests.cs)

```csharp
private static void UpdateContinuationPointIfNeeded(
    Dictionary<InvoiceSubjectType, DateTime?> continuationPoints,
    InvoiceSubjectType subjectType,
    InvoiceExportPackage package)
{
    // Priorytet 1: Paczka obcięta - LastPermanentStorageDate
    if (package.IsTruncated && package.LastPermanentStorageDate.HasValue)
    {
        continuationPoints[subjectType] = package.LastPermanentStorageDate.Value.UtcDateTime;
    }
    // Priorytet 2: Stabilny HWM jako granica kolejnego okna
    else if (package.PermanentStorageHwmDate.HasValue)
    {
        continuationPoints[subjectType] = package.PermanentStorageHwmDate.Value.UtcDateTime;
    }
    else
    {
        // Zakres w pełni przetworzony - usunięcie punktu kontynuacji
        continuationPoints.Remove(subjectType);
    }
}
```

`java` example: [IncrementalInvoiceRetrieveIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/IncrementalInvoiceRetrieveIntegrationTest.java)

```java
private void updateContinuationPointIfNeeded(Map<InvoiceQuerySubjectType, OffsetDateTime> continuationPoints,
                                                 InvoiceQuerySubjectType subjectType,
                                                 InvoiceExportPackage invoiceExportPackage) {
    if (Boolean.TRUE.equals(invoiceExportPackage.getIsTruncated()) && Objects.nonNull(invoiceExportPackage.getLastPermanentStorageDate())) {
        continuationPoints.put(subjectType, invoiceExportPackage.getLastPermanentStorageDate());
    } else {
        continuationPoints.remove(subjectType);
    }
}
```

## Invoice deduplication

### Deduplication strategy

Deduplication is performed based on the KSeF numbers contained in the `_metadata.json` file:

`C#` example: [KSeF.Client.Tests.Core\E2E\Invoice\IncrementalInvoiceRetrievalE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/Invoice/IncrementalInvoiceRetrievalE2ETests.cs)

```csharp
Dictionary<string, InvoiceSummary> uniqueInvoices = new(StringComparer.OrdinalIgnoreCase);
bool hasDuplicates = false;

// Przetwarzanie metadanych z paczki - dodawanie unikalnych faktur i wykrywanie duplikatów
hasDuplicates = packageResult.MetadataSummaries
    .DistinctBy(s => s.KsefNumber, StringComparer.OrdinalIgnoreCase)
    .Any(summary => !uniqueInvoices.TryAdd(summary.KsefNumber, summary));
```

`java` example: [IncrementalInvoiceRetrieveIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/IncrementalInvoiceRetrieveIntegrationTest.java)

```java
hasDuplicates.set(packageProcessingResult.getInvoiceMetadataList()
        .stream()
        .anyMatch(summary -> uniqueInvoices.containsKey(summary.getKsefNumber())));

packageProcessingResult.getInvoiceMetadataList()
        .stream()
        .distinct()
        .forEach(summary -> uniqueInvoices.put(summary.getKsefNumber(), summary));
```

## Related documents

- [High Water Mark](hwm.md)
- [API limits](../limity/limity-api.md)
- [Downloading invoices](pobieranie-faktur.md)
