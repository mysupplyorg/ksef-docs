## QR verification codes

21/08/2025

A QR (Quick Response) code is a graphical representation of text, most often a URL. In the context of KSeF, it is an encoded link containing invoice identification data. This format allows for quick reading of the information using end devices (smartphones or optical scanners). This allows the link to be scanned and redirected directly to the appropriate KSeF system resource responsible for visualizing and verifying the issuer's invoice or KSeF certificate.

QR codes were introduced for situations where an invoice reaches the recipient through a channel other than direct download from the KSeF API (e.g., as a PDF, printout, or email attachment). In such cases, anyone can:

- check whether a given invoice is actually in the KSeF system and whether it has not been modified,
- download its structured version (XML file) without the need to contact the issuer,
- confirm the authenticity of the issuer (in the case of offline invoices).

Code generation (for both online and offline invoices) occurs locally in the client's application based on the data contained in the issued invoice. The QR code must be compliant with the ISO/IEC 18004:2024 standard. If it is not possible to include the code directly on the invoice (e.g., the data format does not allow it), it should be delivered to the recipient as a separate image file or link.

Depending on the issuing mode (online or offline), the invoice visualization includes:

- **online** - one QR code (CODE I), enabling verification and downloading of the invoice from KSeF,
- **offline** - two QR codes:
    - **CODE I** for invoice verification after it has been sent to KSeF,
    - **CODE II** to confirm the authenticity of the issuer based on [the KSeF certificate](/certyfikaty-KSeF.md) .

### 1. CODE I – Invoice verification and download

`KOD I` contains a link enabling the invoice to be read and verified in the KSeF system. After scanning the QR code or clicking the link, the user will receive a simplified presentation of the invoice's basic details and information about its presence in the KSeF system. Full access to the content (e.g., downloading an XML file) requires entering additional data.

#### Generating a link

The link consists of:

- URL address: `https://ksef-test.mf.gov.pl/client-app/invoice` ,
- invoice issue date (field `P_1` ) in the DD-MM-YYYY format,
- seller's Tax Identification Number,
- a hash of the invoice file calculated with the SHA-256 algorithm (invoice file identifier) in Base64URL format.

For example, for an invoice:

- date of issue: "01-02-2026",
- Seller's Tax Identification Number: "1111111111",
- SHA-256 hash in Base64URL format: "UtQp9Gpc51y-u3xApZjIjgkpZ01js-J8KflSPW8WzIE"

The generated link looks like this:

```
https://ksef-test.mf.gov.pl/client-app/invoice/1111111111/01-02-2026/UtQp9Gpc51y-u3xApZjIjgkpZ01js-J8KflSPW8WzIE
```

Example in `C#` :

```csharp
string url = linkSvc.BuildInvoiceVerificationUrl(nip, issueDate, invoiceHash);
```

Java example:

```java
String url = linkSvc.buildInvoiceVerificationUrl(nip, issueDate, xml);
```

#### Generating a QR code

`C#` example: [KSeF.Client.Tests.Core\E2E\QrCode\QrCodeE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/QrCode/QrCodeE2ETests.cs)

```csharp
private const int PixelsPerModule = 5;
byte[] qrBytes = qrCodeService.GenerateQrCode(url, PixelsPerModule);
```

Java example: [QrCodeOnlineIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/QrCodeOnlineIntegrationTest.java)

```java
byte[] qrOnline = qrCodeService.generateQrCode(invoiceForOnlineUrl);
```

#### Marking under the QR code

The KSeF invoice acceptance process is typically immediate—the KSeF number is generated immediately after the document is submitted. In exceptional cases (e.g., high system load), the number may be assigned with a slight delay.

- **If the KSeF number is known:** the KSeF number of the invoice is placed under the QR code (applies to online invoices and offline invoices already sent to the system).

![QR KSeF](qr/qr-ksef.png)

- **If the KSeF number has not been assigned yet:** the word **OFFLINE** is placed under the QR code (applies to offline invoices before sending or online invoices waiting for the number).

![QR Offline](qr/qr-offline.png)

`C#` example: [KSeF.Client.Tests.Core\E2E\QrCode\QrCodeE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/QrCode/QrCodeE2ETests.cs)

```csharp
byte[] labeled = qrCodeService.AddLabelToQrCode(qrBytes, GeneratedQrCodeLabel);
```

Java example: [QrCodeOnlineIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/QrCodeOnlineIntegrationTest.java)

```java
byte[] qrOnline = qrCodeService.addLabelToQrCode(qrOnline, invoiceKsefNumber);
```

### 2. CODE II - Certificate verification

`KOD II` is generated exclusively for invoices issued offline (offline24, offline system unavailability, emergency mode) and serves to confirm **the issuer's** authenticity and authorization to issue an invoice on behalf of the seller. Generation requires an active [KSeF Offline certificate](/certyfikaty-KSeF.md) – the link contains a cryptographic URL signature using the KSeF Offline certificate private key, preventing link forgery by entities without access to the certificate.

> **Note** : The `Authentication` certificate cannot be used to generate CODE II. It is intended solely for API authentication.

The Offline KSeF certificate can be obtained using the [`/certificates`](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Certyfikaty/paths/~1api~1v2~1certificates~1enrollments/post) endpoint.

#### Verification after scanning QR code II

After clicking on the link in the QR code, the KSeF system automatically verifies the issuer's certificate. This process involves the following steps:

1. **Issuer's KSeF Certificate**

    - The certificate exists in the KSeF certificate register and is **valid** .
    - The certificate has not been **revoked** , **blocked** or expired ( `validTo` ).

2. **Issuer's signature**

    - The system verifies **the correctness of the signature** attached in the URL.

3. **Issuer's rights**

    - The entity identified by the issuer's certificate has **active authorization** to issue an invoice in the context ( `ContextIdentifier` ),
    - Verification is carried out in accordance with the rules described in the document [aktywanie.md](uwierzytelnianie.md) ,
    - For example: an accountant signing an invoice on behalf of company A must be granted `InvoiceWrite` right in the context of that company.

4. **Matching the context and seller's VAT number**

    - The system checks whether the context ( `ContextIdentifier` ) has the right to issue an invoice for a given **seller's VAT number** (Invoice `Podmiot1` ). This applies to cases such as:
        - self-invoicing ( `SelfInvoicing` ),
        - tax representative ( `TaxRepresentative` ),
        - VAT groups,
        - local government units,
        - subordinate units identified by an internal identifier,
        - repossession man,
        - enforcement authority,
        - PEF invoices issued on behalf of another entity by the Peppol service provider,
        - invoices issued by a European entity.

    **Example 1. Issuing an invoice by an entity in its own context**

    The entity issues an invoice using a certificate containing its own VAT number. The invoice is issued in the context of the same entity, and the seller's VAT number is indicated in the VAT number field.

    Issuer ID (certificate) | Context | Seller's Tax Identification Number
    --- | --- | ---
    1111111111 | 1111111111 | 1111111111

    **Example 2. Issuing an invoice by an authorized person on behalf of the entity**

    An individual (e.g., an accountant) using a KSeF certificate containing a PESEL number issues an invoice in the context of the entity on whose behalf they hold the appropriate authorization. The seller's Tax Identification Number (NIP) field displays the entity's Tax Identification Number (NIP).

    Issuer ID (certificate) | Context | Seller's Tax Identification Number
    --- | --- | ---
    22222222222 | 1111111111 | 1111111111

    **Example 3. Issuing an invoice on behalf of another entity**

    A natural person issues an invoice in the context of entity A, but the invoice in the seller's Tax Identification Number field indicates the Tax Identification Number of another entity B. This situation is possible when entity A has been granted the authority to issue invoices on behalf of entity B, e.g. tax representative, self-billing.

    Issuer ID (certificate) | Context | Seller's Tax Identification Number
    --- | --- | ---
    22222222222 | 1111111111 | 3333333333

#### Generating a link

The verification link consists of:

- URL: `https://ksef-test.mf.gov.pl/client-app/certificate` ,
- login context identifier type ( [`ContextIdentifier`](uwierzytelnianie.md) ): "Nip", "InternalId", "NipVatUe", "PeppolId"
- login context identifier values,
- seller's Tax Identification Number,
- serial number of the issuer's KSeF certificate,
- SHA-256 invoice file hash in Base64URL format,
- link signing using the private key of the issuer's KSeF certificate (Base64URL encoded).

**Signature format**
 For the signature, a fragment of the URL path is used without the protocol prefix (https://) and without the trailing / character, e.g.:

```
ksef-test.mf.gov.pl/client-app/certificate/Nip/1111111111/1111111111/01F20A5D352AE590/UtQp9Gpc51y-u3xApZjIjgkpZ01js-J8KflSPW8WzIE
```

**Signature algorithms:**

- **RSA (RSASSA-PSS)**

    - Hash function: SHA-256
    - MGF: MGF1 with SHA-256
    - Random salt length: 32 bytes
    - Required key length: Minimum 2048 bits.

    The string to be signed is first hashed with the SHA-256 algorithm, and then the signature is generated according to the RSASSA-PSS scheme.

- **ECDSA (P-256/SHA-256)**
     The signature string is hashed with the SHA-256 algorithm, and then the signature is generated using an ECDSA private key based on the NIST P-256 curve (secp256r1), the choice of which must be indicated when generating the CSR.

    The signature value is a pair of integers (r, s). It can be encoded in one of two formats:

    - **IEEE P1363 Fixed Field Concatenation** – **the recommended method** due to its shorter result string and fixed length. This format is simpler and shorter than DER. The signature is a concatenation of R || S (32 big-endian bytes each).
    - **ASN.1 DER SEQUENCE (RFC 3279)** – the signature is encoded as ASN.1 DER. The signature size is variable. We recommend using this signature type only when IEEE P1363 is not possible due to technological limitations.

In both cases (regardless of the choice of RSA or ESDSA), the received signature value should be encoded in Base64URL format.

For example, for an invoice:

- login context identifier type: "Tip",
- context id value: "1111111111",
- Seller's Tax Identification Number: "1111111111",
- serial number of the issuer's KSeF certificate: "01F20A5D352AE590",
- SHA-256 hash in Base64URL format: "UtQp9Gpc51y-u3xApZjIjgkpZ01js-J8KflSPW8WzIE",
- link signature using the private key of the issuer's KSeF certificate: "BRoRSfcLRh71PAonJCFPg55JYXZW24aEsQrZBRctRjQUnxngrVUJmWhMSHH7ikTp7VMnWYkfWOrUTXELmhJ6x-PNZn3cjm0e741c59h6Q5E-KWIQKONvBmn3XWLkncMrOlFMufwP3lFFXz58hSOvnoOzu3j87nLr7niV0jfkwmW ZVV2oEjrWZTBCKueWX7Dk7WBUX9pPjFFafkE2iCQdm8MuaW8l-y94xTXYesn3mi8IxpCNo3hcTw_yrGnw-ucAA BdhVw7K7MJJacCT2-7_Luh4qiWFiPNcP7Jp_IiI9RQH05xWsxXKA-Z9kgDyjP2KADyKu_vro82bAab4_VW8zQ"

The generated link looks like this:

```
https://ksef-test.mf.gov.pl/client-app/certificate/Nip/1111111111/1111111111/01635E98D9669239/UtQp9Gpc51y-u3xApZjIjgkpZ01js-J8KflSPW8WzIE/BRoRSfcLRh71PAonJCFPg55JYXZW24aEsQrZBRctRjQUnxngrVUJmWhMSHH7ikTp7VMnWYkfWOrUTXELmhJ6x-PNZn3cjm0e741c59h6Q5E-KWIQKONvBmn3XWLkncMrOlFMufwP3lFFXz58hSOvnoOzu3j87nLr7niV0jfkwmWZVV2oEjrWZTBCKueWX7Dk7WBUX9pPjFFafkE2iCQdm8MuaW8l-y94xTXYesn3mi8IxpCNo3hcTw_yrGnw-ucAABdhVw7K7MJJacCT2-7_Luh4qiWFiPNcP7Jp_IiI9RQH05xWsxXKA-Z9kgDyjP2KADyKu_vro82bAab4_VW8zQ
```

Example in `C#` :

```csharp
 var cert = new X509Certificate2(Convert.FromBase64String(certbase64));
 var url = linkSvc.BuildCertificateVerificationUrl(nip, certSerial, invoiceHash, cert, privateKey);
```

Java example: [QrCodeOfflineIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/QrCodeOfflineIntegrationTest.java)

```java
String pem = privateKeyPemBase64.replaceAll("\\s+", "");
byte[] keyBytes = java.util.Base64.getDecoder().decode(pem);

String url = verificationLinkService.buildCertificateVerificationUrl(
    contextNip,
    ContextIdentifierType.NIP,
    contextNip,
    certificate.getCertificateSerialNumber(),
    invoiceHash,
    cryptographyService.parsePrivateKeyFromPem(keyBytes));
```

#### Generating a QR code

`C#` example: [KSeF.Client.Tests.Core\E2E\QrCode\QrCodeE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/QrCode/QrCodeE2ETests.cs)

```csharp
byte[] qrBytes = qrCodeService.GenerateQrCode(url, PixelsPerModule);
```

Java example: [QrCodeOnlineIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/QrCodeOnlineIntegrationTest.java)

```java
byte[] qrOnline = qrCodeService.generateQrCode(invoiceForOnlineUrl);
```

#### Marking under the QR code

The signature **CERTYFIKAT** should appear under the QR code, indicating the KSeF certificate verification function.

`C#` example: [KSeF.Client.Tests.Core\E2E\QrCode\QrCodeE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/QrCode/QrCodeE2ETests.cs)

```csharp
private const string GeneratedQrCodeLabel = "CERTYFIKAT";
byte[] labeled = qrCodeService.AddLabelToQrCode(qrBytes, GeneratedQrCodeLabel);
```

Java example: [QrCodeOnlineIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/QrCodeOnlineIntegrationTest.java)

```java
byte[] qrOnline = qrCodeService.addLabelToQrCode(qrOnline, invoiceKsefNumber);
```

![QR Certificate](qr/qr-cert.png)
