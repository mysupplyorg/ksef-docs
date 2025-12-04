## Interactive session

10/07/2025

The interactive session is used to submit individual structured invoices to the KSeF API. Each invoice must be prepared in XML format, in accordance with the current schema published by the Ministry of Finance.

### Prerequisites

To use interactive dispatch, you must first complete the [authentication](uwierzytelnianie.md) process and have a valid access token ( `accessToken` ), which authorizes you to use protected KSeF API resources.

Before opening a session and sending invoices, the following is required:

- generating a 256-bit symmetric key and a 128-bit initialization vector (IV), appended as a prefix to the ciphertext,
- encrypting the document with the AES-256-CBC algorithm with PKCS#7 padding,
- encrypting the symmetric key with the RSAES-OAEP algorithm (OAEP padding with the MGF1 function based on SHA-256 and the SHA-256 hash), using the KSeF public key of the Ministry of Finance.

These operations can be implemented using the `CryptographyService` component, available in the KSeF client.

C# example: [KSeF.Client.Tests.Core\E2E\OnlineSession\OnlineSessionE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/OnlineSession/OnlineSessionE2ETests.cs)

```csharp
EncryptionData encryptionData = CryptographyService.GetEncryptionData();
```

Java example: [OnlineSessionIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/OnlineSessionIntegrationTest.java)

```java
EncryptionData encryptionData = cryptographyService.getEncryptionData();
```

### 1. Opening of the session

Initializing a new interactive session with:

- invoice schema versions: [FA(2)](faktury/schemy/FA/schemat_FA(2)_v1-0E.xsd) , [FA(3)](faktury/schemy/FA/schemat_FA(3)_v1-0E.xsd)<br> determines which XSD version the system will use to validate sent invoices.
- encrypted symmetric key<br> symmetric key for encrypting XML files, encrypted with the public key of the Ministry of Finance; it is recommended to use a newly generated key for each session.

Opening a session is a lightweight and synchronous operation – you can maintain multiple interactive sessions open simultaneously under a single authentication.

POST [sessions/online](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Wysylka-interaktywna/operation/onlineSession.open)

In response, an object is returned containing:

- `referenceNumber` – a unique identifier of the interactive session that should be passed in all subsequent API calls.
- `validUntil` – The session expiration date. After this time, the session will be automatically closed. The lifetime of an interactive session is 12 hours from its creation.

C# example: [KSeF.Client.Tests.Core\E2E\OnlineSession\OnlineSessionE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/OnlineSession/OnlineSessionE2ETests.cs)

```csharp
OpenOnlineSessionRequest openOnlineSessionRequest = OpenOnlineSessionRequestBuilder
    .Create()
    .WithFormCode(systemCode: SystemCodeHelper.GetValue(systemCode), schemaVersion: DefaultSchemaVersion, value: DefaultFormCodeValue)
    .WithEncryption(
        encryptedSymmetricKey: encryptionData.EncryptionInfo.EncryptedSymmetricKey,
        initializationVector: encryptionData.EncryptionInfo.InitializationVector)
    .Build();

OpenOnlineSessionResponse openOnlineSessionResponse = await KsefClient.OpenOnlineSessionAsync(openOnlineSessionRequest, accessToken, CancellationToken);
```

Java example: [OnlineSessionIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/OnlineSessionIntegrationTest.java)

```java
OpenOnlineSessionRequest request = new OpenOnlineSessionRequestBuilder()
        .withFormCode(new FormCode(systemCode, schemaVersion, value))
        .withEncryptionInfo(encryptionData.encryptionInfo())
        .build();

OpenOnlineSessionResponse openOnlineSessionResponse = ksefClient.openOnlineSession(request, accessToken);
```

### 2. Sending the invoice

The encrypted invoice should be sent to the endpoint:

POST [sessions/online/{referenceNumber}/invoices/](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Wysylka-interaktywna/paths/~1api~1v2~1sessions~1online~1%7BreferenceNumber%7D~1invoices/post)

The response contains the document `referenceNumber` – used to identify the invoice in subsequent operations (e.g. document list).

Once the invoice has been successfully submitted, asynchronous invoice verification begins ( [verification details](faktury/weryfikacja-faktury.md) ).

C# example: [KSeF.Client.Tests.Core\E2E\OnlineSession\OnlineSessionE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/OnlineSession/OnlineSessionE2ETests.cs)

```csharp
byte[] encryptedInvoice = cryptographyService.EncryptBytesWithAES256(invoice, encryptionData.CipherKey, encryptionData.CipherIv);
FileMetadata invoiceMetadata = cryptographyService.GetMetaData(invoice);
FileMetadata encryptedInvoiceMetadata = cryptographyService.GetMetaData(encryptedInvoice);

SendInvoiceRequest sendOnlineInvoiceRequest = SendInvoiceOnlineSessionRequestBuilder
    .Create()
    .WithInvoiceHash(invoiceMetadata.HashSHA, invoiceMetadata.FileSize)
    .WithEncryptedDocumentHash(encryptedInvoiceMetadata.HashSHA, encryptedInvoiceMetadata.FileSize)
    .WithEncryptedDocumentContent(Convert.ToBase64String(encryptedInvoice))
    .Build();

SendInvoiceResponse sendInvoiceResponse = await KsefClient.SendOnlineSessionInvoiceAsync(sendOnlineInvoiceRequest, referenceNumber, accessToken);
```

Java example: [OnlineSessionIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/OnlineSessionIntegrationTest.java)

```java
byte[] invoice = "";

byte[] encryptedInvoice = defaultCryptographyService.encryptBytesWithAES256(invoice,
        encryptionData.cipherKey(),
        encryptionData.cipherIv());

FileMetadata invoiceMetadata = defaultCryptographyService.getMetaData(invoice);
FileMetadata encryptedInvoiceMetadata = defaultCryptographyService.getMetaData(encryptedInvoice);

SendInvoiceOnlineSessionRequest sendInvoiceOnlineSessionRequest = new SendInvoiceOnlineSessionRequestBuilder()
        .withInvoiceHash(invoiceMetadata.getHashSHA())
        .withInvoiceSize(invoiceMetadata.getFileSize())
        .withEncryptedInvoiceHash(encryptedInvoiceMetadata.getHashSHA())
        .withEncryptedInvoiceSize(encryptedInvoiceMetadata.getFileSize())
        .withEncryptedInvoiceContent(Base64.getEncoder().encodeToString(encryptedInvoice))
        .build();

SendInvoiceResponse sendInvoiceResponse = ksefClient.onlineSessionSendInvoice(sessionReferenceNumber, sendInvoiceOnlineSessionRequest, accessToken);

```

### 3. Closing the session

Once all invoices have been sent, the session must be closed, which initiates the asynchronous generation of a collective UPO.

POST [/sessions/online/{referenceNumber}/close](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Wysylka-interaktywna/paths/~1api~1v2~1sessions~1online~1%7BreferenceNumber%7D~1close/post)

The collective UPO will be available after checking the session status.

C# example: [KSeF.Client.Tests.Core\E2E\OnlineSession\OnlineSessionE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/OnlineSession/OnlineSessionE2ETests.cs)

```csharp
await KsefClient.CloseOnlineSessionAsync(referenceNumber, accessToken, CancellationToken);
```

Java example: [OnlineSessionIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/OnlineSessionIntegrationTest.java)

```java
ksefClient.closeOnlineSession(sessionReferenceNumber, accessToken);
```

Related documents:

- [Checking the status and downloading the UPO](faktury/sesja-sprawdzenie-stanu-i-pobranie-upo.md)
- [Invoice verification](faktury/weryfikacja-faktury.md)
- [KSeF number – structure and validation](faktury/numer-ksef.md)
