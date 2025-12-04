## Batch session

10/07/2025

Batch sending allows you to send multiple invoices in a single ZIP file at once, instead of sending each invoice separately.

This solution speeds up and simplifies the processing of large numbers of documents – especially for companies that generate many invoices per day.

Each invoice must be prepared in XML format in accordance with the current scheme published by the Ministry of Finance:

- The ZIP package should be divided into parts no larger than 100 MB (before encryption), which are encrypted and sent separately.
- You must provide information about each part of the package in the `fileParts` object.

### Prerequisites

To use batch dispatch, you must first complete the [authentication](uwierzytelnianie.md) process and have a valid access token ( `accessToken` ), which authorizes you to use protected KSeF API resources.

**Recommendation (correlation of statuses by `invoiceHash` )**
 Before creating a batch shipment, it's recommended to calculate the SHA-256 hash for each invoice XML file (original, before encryption) and save the local mapping. This allows for a clear link between KSeF processing statuses and the local source documents (XML) prepared for shipment.

Before opening a session and sending invoices, the following is required:

- generating a 256-bit symmetric key and a 128-bit initialization vector (IV), appended as a prefix to the ciphertext,
- encrypting the document with the AES-256-CBC algorithm with PKCS#7 padding,
- encrypting the symmetric key with the RSAES-OAEP algorithm (OAEP padding with the MGF1 function based on SHA-256 and the SHA-256 hash), using the KSeF public key of the Ministry of Finance.

These operations can be implemented using the `CryptographyService` component, available in the official KSeF client. This library provides ready-made methods for generating and encrypting keys according to system requirements.

C# example: [KSeF.Client.Tests.Core\E2E\BatchSession\BatchSessionE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/BatchSession/BatchSessionE2ETests.cs)

```csharp
EncryptionData encryptionData = cryptographyService.GetEncryptionData();
```

Java example: [BatchIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/BatchIntegrationTest.java)

```java
EncryptionData encryptionData = cryptographyService.getEncryptionData();
```

The generated data is used to encrypt invoices.

### 1. Preparing the ZIP package

You should create a ZIP package containing all invoices that will be sent in one session.

C# example: [KSeF.Client.Tests.Core\E2E\BatchSession\BatchSessionE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/BatchSession/BatchSessionE2ETests.cs)

```csharp
(byte[] zipBytes, Client.Core.Models.Sessions.FileMetadata zipMeta) =
    BatchUtils.BuildZip(invoices, cryptographyService);

//BatchUtils.BuildZip
public static (byte[] ZipBytes, FileMetadata Meta) BuildZip(
    IEnumerable<(string FileName, byte[] Content)> files,
    ICryptographyService crypto)
{
    using var zipStream = new MemoryStream();
    using var archive = new ZipArchive(zipStream, ZipArchiveMode.Create, leaveOpen: true);
    
    foreach (var (fileName, content) in files)
    {
        var entry = archive.CreateEntry(fileName, CompressionLevel.Optimal);
        using var entryStream = entry.Open();
        entryStream.Write(content);
    }
    
    archive.Dispose();
    
    var zipBytes = zipStream.ToArray();
    var meta = crypto.GetMetaData(zipBytes);

    return (zipBytes, meta);
}
```

Java example: [BatchIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/BatchIntegrationTest.java)

```java
byte[] zipBytes = FilesUtil.createZip(invoicesInMemory);

// get ZIP metadata (before crypto)
FileMetadata zipMetadata = defaultCryptographyService.getMetaData(zipBytes);
```

### 2. Binary division of the ZIP package into parts

Due to file size limitations, the ZIP package should be binarily split into smaller parts that will be uploaded separately. Each part should have a unique name and sequential number.

C# example: [KSeF.Client.Tests.Core\E2E\BatchSession\BatchSessionE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/BatchSession/BatchSessionE2ETests.cs)

```csharp

 // Pobierz metadane ZIP-a (przed szyfrowaniem)
 var zipMetadata = cryptographyService.GetMetaData(zipBytes);
int maxPartSize = 100 * 1000 * 1000; // 100 MB
int partCount = (int)Math.Ceiling((double)zipBytes.Length / maxPartSize);
int partSize = (int)Math.Ceiling((double)zipBytes.Length / partCount);
var zipParts = new List<byte[]>();
for (int i = 0; i < partCount; i++)
{
    int start = i * partSize;
    int size = Math.Min(partSize, zipBytes.Length - start);
    if (size <= 0) break;
    var part = new byte[size];
    Array.Copy(zipBytes, start, part, 0, size);
    zipParts.Add(part);
}

```

Java example: [BatchIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/BatchIntegrationTest.java)

```java
List<byte[]> zipParts = FilesUtil.splitZip(partsCount, zipBytes);
```

### 3. Encrypting part of the package

Each part must be encrypted with a newly generated AES‑256‑CBC key with PKCS#7 padding.

Example in C#:

```csharp
    //  Szyfruj każdy part i pobierz metadane
        var encryptedParts = new List<(byte[] Data, FileHash Metadata)>();
        for (int i = 0; i < zipParts.Count; i++)
        {
            var encrypted = cryptographyService.EncryptBytesWithAES256(zipParts[i], encryptionData.CipherKey, encryptionData.CipherIv);
            var metadata = cryptographyService.GetMetaData(encrypted);
            encryptedParts.Add((encrypted, metadata));

            // Zapisz part na dysku
            var partFileName = Path.Combine(BatchPartsDirectory, $"faktura_part{i + 1}.zip.aes");
            System.IO.File.WriteAllBytes(partFileName, encrypted);
        }
```

Java example: [BatchIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/BatchIntegrationTest.java)

```java
 List<BatchPartSendingInfo> encryptedZipParts = new ArrayList<>();
 for (int i = 0; i < zipParts.size(); i++) {
     byte[] encryptedZipPart = defaultCryptographyService.encryptBytesWithAES256(
             zipParts.get(i),
             cipherKey,
             cipherIv
     );
     FileMetadata zipPartMetadata = defaultCryptographyService.getMetaData(encryptedZipPart);
     encryptedZipParts.add(new BatchPartSendingInfo(encryptedZipPart, zipPartMetadata, (i + 1)));
 }

```

### 4. Opening a batch session

Initialize a new batch session with:

- invoice schema versions: [FA(2)](faktury/schemy/FA/schemat_FA(2)_v1-0E.xsd) , [FA(3)](faktury/schemy/FA/schemat_FA(3)_v1-0E.xsd)<br> determines which XSD version the system will use to validate sent invoices.
- encrypted symmetric key<br> symmetric key for encrypting XML files, encrypted with the public key of the Ministry of Finance; it is recommended to use a newly generated key for each session.
- metadata of the ZIP package and its parts: file name, hash, size and list of parts (if the package is split)

POST [/sessions/batch](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Wysylka-wsadowa/operation/batch.open)

In response to opening the session, we will receive an object containing `referenceNumber` , which will be used in the next steps to identify the batch session.

C# example: [KSeF.Client.Tests.Core\E2E\BatchSession\BatchSessionE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/BatchSession/BatchSessionE2ETests.cs)

```csharp
Client.Core.Models.Sessions.BatchSession.OpenBatchSessionRequest openBatchRequest =
    BatchUtils.BuildOpenBatchRequest(zipMeta, encryptionData, encryptedParts, systemCode);

Client.Core.Models.Sessions.BatchSession.OpenBatchSessionResponse openBatchSessionResponse =
    await BatchUtils.OpenBatchAsync(KsefClient, openBatchRequest, accessToken);

//BatchUtils.BuildOpenBatchRequest
public static OpenBatchSessionRequest BuildOpenBatchRequest(
    FileMetadata zipMeta,
    EncryptionData encryption,
    IEnumerable<BatchPartSendingInfo> encryptedParts,
    SystemCode systemCode = DefaultSystemCode,
    string schemaVersion = DefaultSchemaVersion,
    string value = DefaultValue)
{
    var builder = OpenBatchSessionRequestBuilder
        .Create()
        .WithFormCode(systemCode: SystemCodeHelper.GetValue(systemCode), schemaVersion: schemaVersion, value: value)
        .WithBatchFile(fileSize: zipMeta.FileSize, fileHash: zipMeta.HashSHA);

    foreach (var p in encryptedParts)
    {
        builder = builder.AddBatchFilePart(
            ordinalNumber: p.OrdinalNumber,
            fileName: $"part_{p.OrdinalNumber}.zip.aes",
            fileSize: p.Metadata.FileSize,
            fileHash: p.Metadata.HashSHA);
    }

    return builder
        .EndBatchFile()
        .WithEncryption(
            encryptedSymmetricKey: encryption.EncryptionInfo.EncryptedSymmetricKey,
            initializationVector: encryption.EncryptionInfo.InitializationVector)
        .Build();
}

//BatchUtils.OpenBatchAsync
public static async Task<OpenBatchSessionResponse> OpenBatchAsync(
    IKSeFClient client,
    OpenBatchSessionRequest openReq,
    string accessToken)
    => await client.OpenBatchSessionAsync(openReq, accessToken);
```

Java example: [BatchIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/BatchIntegrationTest.java)

```java
OpenBatchSessionRequestBuilder builder = OpenBatchSessionRequestBuilder.create()
        .withFormCode(SystemCode.FA_2, SchemaVersion.VERSION_1_0E, SessionValue.FA)
        .withOfflineMode(false)
        .withBatchFile(zipMetadata.getFileSize(), zipMetadata.getHashSHA());

for (int i = 0; i < encryptedZipParts.size(); i++) {
        BatchPartSendingInfo part = encryptedZipParts.get(i);
        builder = builder.addBatchFilePart(i + 1, "faktura_part" + (i + 1) + ".zip.aes",part.getMetadata().getFileSize(), part.getMetadata().getHashSHA());
}

OpenBatchSessionRequest request = builder.endBatchFile()
        .withEncryption(
                        encryptionData.encryptionInfo().getEncryptedSymmetricKey(),
                        encryptionData.encryptionInfo().getInitializationVector()
                )
        .build();

OpenBatchSessionResponse response = ksefClient.openBatchSession(request, accessToken);
```

The method returns a list of package parts; for each part it provides the upload address (URL), the required HTTP method and a set of headers that should be sent with the given part.

### 5. Sending the declared parts of the parcel

Using the data returned when opening the session in `partUploadRequests` , i.e., a unique URL with an access key, an HTTP method, and the required headers, you should send each declared part of the package ( `fileParts` ) to the specified address, using exactly these values for that part. The link between the declaration and the sending instruction is the `ordinalNumber` property.

The request body should contain the bytes of the appropriate part of the file (without wrapping in JSON).

> Note: Do not add access token ( `accessToken` ) to the headers.

Each part is sent in a separate HTTP request. Returned response codes:

- `201` - correct file receipt,
- `400` - incorrect data,
- `401` - invalid authentication,
- `403` - no write permissions (e.g. write timeout).

C# example: [KSeF.Client.Tests.Core\E2E\BatchSession\BatchSessionE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/BatchSession/BatchSessionE2ETests.cs)

```csharp
await KsefClient.SendBatchPartsAsync(openBatchSessionResponse, encryptedParts);
```

Java example: [BatchIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/BatchIntegrationTest.java)

```java
ksefClient.sendBatchParts(response, encryptedZipParts);
```

**Time limit for uploading batches in a batch session**
 File uploads in a batch session are time-limited. This time depends solely on the number of declared batches and is 20 minutes per batch. Each additional batch proportionally increases the time limit **for each batch** in the batch.

Total time to ship each batch = number of batches × 20 minutes.
 Example: The package contains 2 parts – each part has 40 minutes to ship.

The size of the party is not important for determining the time limit – the only criterion is the number of parties declared at the opening of the session.

Authorization is verified at the beginning of each HTTP request. If the address is valid when the request is accepted, the transfer operation is completed in full. Expiration during the transfer does not interrupt the initiated operation.

### 6. Closing the batch session

After sending all parts of the batch, close the batch session, which initiates asynchronous processing of the invoice batch ( [verification details](faktury/weryfikacja-faktury.md) ) and the generation of a collective UPO.

POST [/sessions/batch/{referenceNumber}/close](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Wysylka-wsadowa/paths/~1api~1v2~1sessions~1batch~1%7BreferenceNumber%7D~1close/post) }]

C# example: [KSeF.Client.Tests.Core\E2E\BatchSession\BatchSessionE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/BatchSession/BatchSessionE2ETests.cs)

```csharp
await KsefClient.CloseBatchSessionAsync(referenceNumber, accessToken);
```

Java example: [BatchIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/BatchIntegrationTest.java)

```java
ksefClient.closeBatchSession(referenceNumber, accessToken);
```

Look

- [Checking the status and downloading the UPO](faktury/sesja-sprawdzenie-stanu-i-pobranie-upo.md)
- [Invoice verification](faktury/weryfikacja-faktury.md)
- [KSeF number – structure and validation](faktury/numer-ksef.md)
