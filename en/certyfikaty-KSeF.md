## KSeF certificates

31/08/2025

### Entry

A KSeF certificate is a digital confirmation of an entity's identity, issued by the KSeF system at the user's request.

A KSeF certificate request can only be submitted for data contained in the certificate used for [authentication](uwierzytelnianie.md) . Based on this data, the [/certificates/enrollments/data](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Certyfikaty/paths/~1api~1v2~1certificates~1enrollments~1data/get) endpoint returns the identifying data that must be used in the certification request.

> The system does not allow you to apply for a certificate on behalf of another entity.

There are two types of certificates available – each certificate can **only have one type** ( `Authentication` or `Offline` ). It is not possible to issue a certificate that combines both functions.

Type | Description
--- | ---
`Authentication` | Certificate intended for authentication in the KSeF system.<br> **keyUsage:** Digital Signature (80)
`Offline` | This certificate is intended solely for offline invoicing. It is used to confirm the authenticity of the issuer and the integrity of the invoice via [a QR code](kody-qr.md) . It does not provide authentication.<br> **keyUsage:** Non-Repudiation (40)

#### The process of obtaining a certificate

The certificate application process consists of several stages:

1. Checking available limits,
2. Downloading data for the certification application,
3. Sending the application,
4. Downloading the issued certificate,

### 1. Checking the limits

Before an API client submits a request for a new certificate, it is recommended to verify the certificate limit.

The API provides information on:

- the maximum number of certificates that can be available,
- the number of currently active certificates,
- possibility of submitting another application.

GET [/certificates/limits](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Certyfikaty/paths/~1api~1v2~1certificates~1limits/get)

C# example: [KSeF.Client.Tests.Core\E2E\Certificates\CertificatesE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/Certificates/CertificatesE2ETests.cs)

```csharp
CertificateLimitResponse certificateLimitResponse = await KsefClient
    .GetCertificateLimitsAsync(accessToken, CancellationToken);
```

Java example: [CertificateIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/CertificateIntegrationTest.java)

```java
CertificateLimitsResponse response = ksefClient.getCertificateLimits(accessToken);
```

### 2. Downloading data for the certification application

To start the process of applying for a KSeF certificate, you need to download a set of identification data that the system will return in response to an endpoint call.
 GET [/certificates/enrollments/data](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Certyfikaty/paths/~1api~1v2~1certificates~1enrollments~1data/get) .

This data is read from the certificate used for authentication, which may be:

- qualified certificate of a natural person – containing the PESEL or NIP number,
- qualified certificate of the organization (so-called company stamp) – containing the Tax Identification Number,
- Trusted Profile (ePUAP) – used by natural persons, contains the PESEL number,
- KSeF internal certificate – issued by the KSeF system, it is not a qualified certificate, but is recognized in the authentication process.

Based on this, the system returns a complete set of DN (X.500 Distinguished Name) attributes that must be used when building a certification request (CSR). Modifying this data will result in rejection of the request.

**Note** : Downloading certification data is only possible after authentication using a signature (XAdES). Authentication using the KSeF system token does not allow you to submit a certificate request.

C# example: [KSeF.Client.Tests.Core\E2E\Certificates\CertificatesE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/Certificates/CertificatesE2ETests.cs)

```csharp
CertificateEnrollmentsInfoResponse certificateEnrollmentsInfoResponse =
    await KsefClient.GetCertificateEnrollmentDataAsync(accessToken, CancellationToken);
```

Java example: [CertificateIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/CertificateIntegrationTest.java)

```java
CertificateEnrollmentsInfoResponse response = ksefClient.getCertificateEnrollmentInfo(accessToken);
```

Here is the full list of fields that can be returned, presented in a table format containing OIDs:

OID | Name (English) | Description | Natural person | Company seal
--- | --- | --- | --- | ---
2.5.4.3 | commonName | Common name | ✔️ | ✔️
2.5.4.4 | surname | Last name | ✔️ | ❌
2.5.4.5 | serialNumber | Serial number (e.g. PESEL, NIP) | ✔️ | ❌
2.5.4.6 | countryName | Country code (e.g. PL) | ✔️ | ✔️
2.5.4.10 | organizationName | Name of organization/company | ❌ | ✔️
2.5.4.42 | givenName | Name or names | ✔️ | ❌
2.5.4.45 | uniqueIdentifier | Unique identifier (optional) | ✔️ | ✔️
2.5.4.97 | organizationIdentifier | Organization ID (e.g. Tax Identification Number) | ❌ | ✔️

The `givenName` attribute can appear multiple times and is returned as a list of values.

### 3. Preparing a CSR (Certificate Signing Request)

To apply for a KSeF certificate, you must prepare a Certificate Signing Request (CSR) in the PKCS#10 standard, in DER format, encoded in Base64. The CSR contains:

- information identifying the entity (DN – Distinguished Name),
- public key that will be associated with the certificate.

Requirements for the private key used to sign the CSR:

- Types allowed:

    - RSA (OID: 1.2.840.113549.1.1.1), key length: 2048 bits,
    - EC (elliptical keys, OID: 1.2.840.10045.2.1), NIST P-256 curve (secp256r1).

- The use of EC keys is recommended.

- Allowed signature algorithms:

    - RSA PKCS#1 v1.5,
    - RSA PSS,
    - ECDSA (RFC 3279 compliant signature format).

- Allowed hash functions used for CSR signature:

    - SHA1,
    - SHA256,
    - SHA384,
    - SHA512.

All identification data (X.509 attributes) should match the values returned by the system in the previous step (/certificates/enrollments/data). Modifying this data will result in rejection of the application.

C# example (using `ICryptographyService` ): [KSeF.Client.Tests.Core\E2E\Certificates\CertificatesE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/Certificates/CertificatesE2ETests.cs)

```csharp
var (csr, key) = CryptographyService.GenerateCsrWithRSA(TestFixture.EnrollmentInfo);
```

Java example: [CertificateIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/CertificateIntegrationTest.java)

```java
CsrResult csr = defaultCryptographyService.generateCsrWithRsa(enrollmentInfo);
```

- `csrBase64Encoded` – contains the CSR request encoded in Base64 format, ready to be sent to KSeF
- `privateKeyBase64Encoded` – Contains the private key associated with the generated CSR, encoded in Base64. This key will be needed for certificate signing operations.

**Note** : The private key should be stored securely and in accordance with the organization's security policy.

### 4. Sending a certification request

After preparing the certification request (CSR), it should be sent to the KSeF system using the call

POST [/certificates/enrollments](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Certyfikaty/paths/~1api~1v2~1certificates~1enrollments/post)

In the submitted application, please provide:

- **certificate name** – later visible in the certificate metadata, facilitating identification,
- **certificate type** – `Authentication` or `Offline` ,
- **CSR** in PKCS#10 (DER) format, encoded as a Base64 string,
- (optional) **validFrom** – the validity start date. If not specified, the certificate will be valid from the date it is issued.

Make sure the CSR contains exactly the same data that was returned by the /certificates/enrollments/data endpoint.

C# example: [KSeF.Client.Tests.Core\E2E\Certificates\CertificatesE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/Certificates/CertificatesE2ETests.cs)

```csharp
SendCertificateEnrollmentRequest sendCertificateEnrollmentRequest = SendCertificateEnrollmentRequestBuilder
    .Create()
    .WithCertificateName(TestCertificateName)
    .WithCertificateType(CertificateType.Authentication)
    .WithCsr(csr)
    .WithValidFrom(DateTimeOffset.UtcNow.AddDays(CertificateValidityDays))
    .Build();

CertificateEnrollmentResponse certificateEnrollmentResponse = await KsefClient
    .SendCertificateEnrollmentAsync(sendCertificateEnrollmentRequest, accessToken, CancellationToken);
```

Java example: [CertificateIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/CertificateIntegrationTest.java)

```java
SendCertificateEnrollmentRequest request = new SendCertificateEnrollmentRequestBuilder()
        .withValidFrom(OffsetDateTime.now().toString())
        .withCsr(csr.csr())
        .withCertificateName("certificate")
        .withCertificateType(CertificateType.AUTHENTICATION)
        .build();

CertificateEnrollmentResponse response = ksefClient.sendCertificateEnrollment(request, accessToken);
```

In response, you will receive `referenceNumber` , which allows you to monitor the status of the application and later download the issued certificate.

### 5. Checking the status of your application

The certificate issuance process is asynchronous. This means that the system does not return the certificate immediately after submitting the application, but allows for its subsequent retrieval after processing. The application status should be checked periodically using the `referenceNumber` returned in the application response (/certificates/enrollments).

GET [/certificates/enrollments/{referenceNumber}](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Certyfikaty/paths/~1api~1v2~1certificates~1enrollments~1%7BreferenceNumber%7D/get)

If the certification request is rejected, we will receive an error message in response.

C# example: [KSeF.Client.Tests.Core\E2E\Certificates\CertificatesE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/Certificates/CertificatesE2ETests.cs)

```csharp
CertificateEnrollmentStatusResponse certificateEnrollmentStatusResponse = await KsefClient
    .GetCertificateEnrollmentStatusAsync(TestFixture.EnrollmentReference, accessToken, CancellationToken);
```

Java example: [CertificateIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/CertificateIntegrationTest.java)

```java
CertificateEnrollmentStatusResponse response = ksefClient.getCertificateEnrollmentStatus(referenceNumber, accessToken);

```

Once you have obtained the certificate serial number ( `certificateSerialNumber` ), you can retrieve its content and metadata in the next steps of the process.

### 6. Downloading the certificate list

The KSeF system allows you to retrieve the contents of previously issued internal certificates based on a list of serial numbers. Each certificate is returned in DER format, encoded as a Base64 string.

POST [/certificates/retrieve](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Certyfikaty/paths/~1api~1v2~1certificates~1retrieve/post)

C# example: [KSeF.Client.Tests.Core\E2E\Certificates\CertificatesE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/Certificates/CertificatesE2ETests.cs)

```csharp
CertificateListRequest certificateListRequest = new CertificateListRequest { CertificateSerialNumbers = TestFixture.SerialNumbers };

CertificateListResponse certificateListResponse = await KsefClient
    .GetCertificateListAsync(certificateListRequest, accessToken, CancellationToken);
```

Java example: [CertificateIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/CertificateIntegrationTest.java)

```java
CertificateListResponse certificateResponse = ksefClient.getCertificateList(new CertificateListRequest(List.of(certificateSerialNumber)), accessToken);
```

Each response element includes:

Field | Description
--- | ---
`certificateSerialNumber` | Certificate serial number
`certificateName` | Certificate name assigned upon registration
`certificate` | Certificate content encoded in Base64 (DER format)
`certificateType` | Certificate type ( `Authentication` , `Offline` ).

### 7. Downloading the certificate metadata list

You can download a list of internal certificates submitted by a given entity. This data includes both active and historical certificates, along with their status, validity range, and identifiers.

POST [/certificates/query](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Certyfikaty/paths/~1api~1v2~1certificates~1query/post)

Filtering parameters (optional):

- `status` - certificate status ( `Active` , `Blocked` , `Revoked` , `Expired` )
- `expiresAfter` - certificate expiry date (optional)
- `name` - certificate name (optional)
- `type` - certificate type ( `Authentication` , `Offline` ) (optional)
- `certificateSerialNumber` - certificate serial number (optional)
- `pageSize` - number of elements on the page (default 10)
- `pageOffset` - results page number (default 0)

C# example: [KSeF.Client.Tests.Core\E2E\Certificates\CertificatesE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/Certificates/CertificatesE2ETests.cs)

```csharp
var request = GetCertificateMetadataListRequestBuilder
    .Create()
    .WithCertificateSerialNumber(serialNumber)
    .WithName(name)
    .Build();
    CertificateMetadataListResponse certificateMetadataListResponse = await KsefClient
            .GetCertificateMetadataListAsync(accessToken, requestPayload, pageSize, pageOffset, CancellationToken);
```

Java example: [CertificateIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/CertificateIntegrationTest.java)

```java
QueryCertificatesRequest request = new CertificateMetadataListRequestBuilder().build();

CertificateMetadataListResponse response = ksefClient.getCertificateMetadataList(request, pageSize, pageOffset, accessToken);


```

In response we will receive certificate metadata.

### 8. Revocation of certificates

A KSeF certificate can only be revoked by its owner in the event of private key compromise, termination of its use, or organizational change. Once revoked, the certificate cannot be used for further authentication or operations in the KSeF system. Revocation is based on the certificate's serial number ( `certificateSerialNumber` ) and an optional revocation reason.

POST [/certificates/{certificateSerialNumber}/revoke](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Certyfikaty/paths/~1api~1v2~1certificates~1%7BcertificateSerialNumber%7D~1revoke/post)

C# example: [KSeF.Client.Tests.Core\E2E\Certificates\CertificatesE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/Certificates/CertificatesE2ETests.cs)

```csharp
CertificateRevokeRequest certificateRevokeRequest = RevokeCertificateRequestBuilder
        .Create()
        .WithRevocationReason(CertificateRevocationReason.KeyCompromise)
        .Build();

await ksefClient.RevokeCertificateAsync(request, certificateSerialNumber, accessToken, cancellationToken)
     .ConfigureAwait(false);
```

Java example: [CertificateIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/CertificateIntegrationTest.java)

```java
CertificateRevokeRequest request = new CertificateRevokeRequestBuilder()
        .withRevocationReason(CertificateRevocationReason.KEYCOMPROMISE)
        .build();

ksefClient.revokeCertificate(request, serialNumber, accessToken);
```

Once revoked, a certificate cannot be reused. If you need to use it again, you must apply for a new certificate.
