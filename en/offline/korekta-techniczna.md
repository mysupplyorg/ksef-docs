# Offline technical invoice correction

20/08/2025

## Description of functionality

Technical correction allows you to re-send an invoice issued [offline](../tryby-offline.md) , which was **rejected** after being sent to the KSeF system due to technical errors, e.g.:

- non-compliance with the scheme,
- exceeding the permissible file size,
- duplicate invoice,
- other **technical validation errors** that prevent the assignment `numeru KSeF` .

> **Attention** !

1. The technical correction **does not apply to** situations related to the lack of authorizations of entities appearing on the invoice (e.g. self-billing, validation of relations for local government units or VAT groups).
2. In this mode **, it is not allowed** to correct the content of the invoice – the technical correction only concerns technical problems that prevent its acceptance in the KSeF system.
3. A technical correction can only be sent in [an interactive session](../sesja-interaktywna.md) , but it can apply to offline invoices rejected in both [an interactive](../sesja-interaktywna.md) and a [batch](../sesja-wsadowa.md) session.
4. It is not permitted to technically correct an offline invoice for which another correct correction has already been accepted.

## Example of offline technical invoice correction process

1. **The seller issues an invoice offline.**

    - The invoice contains two QR codes:
        - **QR code I** – enables invoice verification in the KSeF system,
        - **QR Code II** – enables confirmation of the authenticity of the issuer based on [the KSeF certificate](../certyfikaty-KSeF.md) .

2. **The customer receives a visualization of the invoice (e.g. in the form of a printout).**

    - After scanning **the QR code,** the customer receives information that the invoice **has not yet been sent to the KSeF system** .
    - After scanning **the QR II code,** the customer receives information about the KSeF certificate, which confirms the authenticity of the issuer.

3. **The seller sends the invoice offline to the KSeF system.**

    - The KSeF system verifies the document.
    - The invoice is **rejected** due to a technical error (e.g. incorrect compliance with the XSD schema).

4. **The seller updates his software** and generates an invoice again with the same content but in line with the schema.

    - Since the XML content is different from the original version, **the SHA-256 hash of the invoice file is different** .

5. **The seller sends the corrected invoice as a technical correction.**

    - Specifies the SHA-256 hash of the original, rejected offline invoice in `hashOfCorrectedInvoice` field.
    - The `offlineMode` parameter is set to `true` .

6. **The KSeF system correctly accepts the invoice.**

    - The document receives a KSeF number.
    - The invoice is **linked to the original offline invoice** , the hash of which was indicated in the `hashOfCorrectedInvoice` field.
    - This makes it possible to redirect the customer from the "old" QR code I to the corrected invoice.

7. **The customer uses the QR code I located on the original invoice.**

    - The KSeF system informs that **the original invoice has been technically corrected** .
    - The customer receives the metadata of a new, correctly processed invoice and can download it from the system.

## Sending a correction

The correction is sent in accordance with the rules described in the [interactive session](../sesja-interaktywna.md) document, with the following additional setting:

- `offlineMode: true` ,
- `hashOfCorrectedInvoice` – hash of the original invoice.

Example in C#:

```csharp
var sendOnlineInvoiceRequest = SendInvoiceOnlineSessionRequestBuilder
    .Create()
    .WithInvoiceHash(invoiceMetadata.HashSHA, invoiceMetadata.FileSize)
    .WithEncryptedDocumentHash(
        encryptedInvoiceMetadata.HashSHA, encryptedInvoiceMetadata.FileSize)
    .WithEncryptedDocumentContent(Convert.ToBase64String(encryptedInvoice))
    .WithOfflineMode(true)
    .WithHashOfCorrectedInvoice(hashOfCorrectedInvoice)
    .Build();
```

Java example: [OnlineSessionController#sendTechnicalCorrection.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/main/java/pl/akmf/ksef/sdk/api/OnlineSessionController.java#L120)

```java
SendInvoiceOnlineSessionRequest sendInvoiceOnlineSessionRequest = new SendInvoiceOnlineSessionRequestBuilder()
           .withInvoiceHash(invoiceMetadata.getHashSHA())
           .withInvoiceSize(invoiceMetadata.getFileSize())
           .withEncryptedInvoiceHash(encryptedInvoiceMetadata.getHashSHA())
           .withEncryptedInvoiceSize(encryptedInvoiceMetadata.getFileSize())
           .withEncryptedInvoiceContent(Base64.getEncoder().encodeToString(encryptedInvoice))
           .withOfflineMode(true)
           .withHashOfCorrectedInvoice(hashOfCorrectedInvoice)
        .build();
```

## Related documents

- [Offline modes](../tryby-offline.md)
- [QR codes](../kody-qr.md)
