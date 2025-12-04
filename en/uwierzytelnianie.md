## Authentication

10/07/2025

## Entry

Authentication in the KSeF API 2.0 system is a mandatory step that must be completed before accessing protected system resources. This process involves **obtaining an access token** `accessToken` `JWT` ), which is then used to authorize API operations.

The authentication process is based on two elements:

- Login context – specifies the entity on whose behalf operations will be performed in the system, e.g. a company identified by its Tax Identification Number.
- Authenticator – indicates who is attempting authentication. How this information is conveyed depends on the authentication method chosen.

**Available authentication methods:**

- **Using the XAdES signature**<br> An XML document ( `AuthTokenRequest` ) containing a digital signature in XAdES format is sent. Information about the authenticator is read from the certificate used for the signature (e.g., NIP, PESEL, or certificate fingerprint).
- **Using the KSeF token**<br> A JSON document containing a previously obtained system token (the so-called [KSeF token](tokeny-ksef.md) ) is sent. Information about the authenticator is read based on the sent [KSeF token](tokeny-ksef.md) .

The authenticator is subject to verification – the system checks whether the specified entity has at least one active permission for the selected context. Failure to do so prevents obtaining an access token and using the API.

The obtained token is valid only for a specified period and can be refreshed without re-authentication. Tokens are automatically invalidated if privileges are lost.

## Authentication process

> **Quick start (demo)**
>
> To demonstrate the full authentication process (downloading the challenge, preparing and signing the XAdES, uploading, checking the status, downloading `accessToken` and `refreshToken` ), you can use the demo application. Details can be found in the document: **[XAdES Test Certificates and Signatures](auth/testowe-certyfikaty-i-podpisy-xades.md)** .
>
> **Note:** Self-signed certificates are only allowed in a test environment.

### 1. Obtaining the auth challenge

The authentication process begins by retrieving the **auth challenge** , which is required to further construct the authentication request. The challenge is retrieved using the following call:<br> POST [/auth/challenge](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Uwierzytelnianie/paths/~1api~1v2~1auth~1challenge/post)<br>

The challenge lifetime is 10 minutes.

`C#` example: [KSeF.Client.Tests.Core\E2E\KsefToken\KsefTokenE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/KsefToken/KsefTokenE2ETests.cs)

```csharp

AuthChallengeResponse challenge = await KsefClient.GetAuthChallengeAsync();
```

`Java` example: [BaseIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/configuration/BaseIntegrationTest.java)

```java
AuthenticationChallengeResponse challenge = ksefClient.getAuthChallenge();
```

The response returns a challenge and a timestamp.

### 2. Choosing an identity confirmation method

### 2.1. Authentication **with a qualified electronic signature**

#### 1. Preparing the XML document (AuthTokenRequest)

Once the auth challenge is obtained, an XML document compliant with the [AuthTokenRequest](https://ksef-test.mf.gov.pl/docs/v2/schemas/authv2.xsd) schema must be prepared for use in the subsequent authentication process. This document contains:

Key | Value
--- | ---
Challenge | `Wartość otrzymana z wywołania POST [/auth/challenge](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Uzyskiwanie-dostepu/paths/~1api~1v2~1auth~1challenge/post)`
ContextIdentifier | `Identyfikator kontekstu, dla którego realizowane jest uwierzytelnienie (NIP, identyfikator wewnętrzny, identyfikator złożony VAT UE)`
SubjectIdentifierType | `Sposób identyfikacji podmiotu uwierzytelniającego się. Możliwe wartości: certificateSubject (np. NIP/PESEL z certyfikatu) lub certificateFingerprint (odcisk palca certyfikatu).`
(optional) AuthorizationPolicy | `Reguły autoryzacyjne. Obecnie obsługiwana lista dozwolonych adresów IP klienta.`

Sample XML documents:

- SubjectIdentifierType from [certificateSubject](auth/subject-identifier-type-certificate-subject.md)
- SubjectIdentifierType from [certificateFingerprint](auth/subject-identifier-type-certificate-fingerprint.md)
- ContextIdentifier with [Tax Identification Number](auth/context-identifier-nip.md)
- ContextIdentifier from [InternalId](auth/context-identifier-internal-id.md)
- ContextIdentifier from [NipVatUe](auth/context-identifier-nip-vat-ue.md)

In the next step, the document will be signed using the entity's certificate.

**Implementation examples:** <br>

`ContextIdentifier` | `SubjectIdentifierType` | Meaning
--- | --- | ---
`Type: nip`<br>`Value: 1234567890` | `certificateSubject`<br>` (NIP 1234567890 w certyfikacie)` | Authentication applies to a company with the Tax Identification Number (NIP) 1234567890. The signature will be made with a certificate containing the Tax Identification Number (NIP) 1234567890 in field 2.5.4.97.
`Type: nip`<br>`Value: 1234567890` | `certificateSubject`<br>` (pesel 88102341294 w certyfikacie)` | Authentication applies to a company with the Tax Identification Number (NIP) 1234567890. The signature will be made with a certificate of a natural person containing the PESEL number 88102341294 in field 2.5.4.5. The KSeF system will check whether this person is **authorized to act** on behalf of the company (e.g. based on the ZAW-FA application).
`Type: nip`<br>`Value: 1234567890` | `certificateFingerprint:`<br>` (odcisk certyfikatu  70a992150f837d5b4d8c8a1c5269cef62cf500bd)` | The authentication applies to the company with the Tax Identification Number (NIP) 1234567890. The signature will be made with a certificate with the imprint 70a992150f837d5b4d8c8a1c5269cef62cf500bd, which has been **authorized to act** on behalf of the company (e.g. based on the ZAW-FA application).

C# example: [KSeF.Client.Tests.Core\E2E\Authorization\AuthorizationE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/Authorization/AuthorizationE2ETests.cs)

```csharp
var authorizationPolicy = new AuthenticationTokenAuthorizationPolicy
{
    AllowedIps = new AuthenticationTokenAllowedIps
    {
        Ip4Addresses = ["192.168.0.1", "192.222.111.1"],
        Ip4Masks = ["192.168.1.0/24"], // Przykładowa maska
        Ip4Ranges = ["222.111.0.1-222.111.0.255"] // Przykładowy zakres IP
    }
};

AuthenticationTokenRequest authTokenRequest = AuthTokenRequestBuilder
    .Create()
    .WithChallenge(challengeResponse.Challenge)
    .WithContext(AuthenticationTokenContextIdentifierType.Nip, ownerNip)
    .WithIdentifierType(AuthenticationTokenSubjectIdentifierTypeEnum.CertificateSubject)
    .WithAuthorizationPolicy(authorizationPolicy)
    .Build();
```

`Java` example: [BaseIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/configuration/BaseIntegrationTest.java)

```java
AuthTokenRequest authTokenRequest = new AuthTokenRequestBuilder()
        .withChallenge(challenge.getChallenge())
        .withContextNip(context)
        .withSubjectType(SubjectIdentifierTypeEnum.CERTIFICATE_SUBJECT)
        .withAuthorizationPolicy(authorizationPolicy)
        .build();
```

#### 2. Signing the document (XAdES)

Once the `AuthTokenRequest` document is prepared, it must be digitally signed using the XAdES (XML Advanced Electronic Signatures) format. This is the required signature format for the authentication process. To sign the document, you can use:

- Qualified certificate of a natural person – containing the PESEL or NIP number of the person authorized to act on behalf of the company,
- Qualified certificate of the organization (so-called company stamp) - containing the Tax Identification Number.
- Trusted Profile (ePUAP) – enables signing a document; used by individuals who can submit it via [gov.pl.](https://www.gov.pl/web/gov/podpisz-dokument-elektronicznie-wykorzystaj-podpis-zaufany)
- [KSeF Certificate](certyfikaty-KSeF.md) – Issued by the KSeF system. This certificate is not a qualified certificate, but is recognized in the authentication process. The KSeF certificate is used exclusively for the KSeF system.
- Peppol Service Provider Certificate - containing the provider ID.

In the test environment, it is allowed to use a self-generated certificate that is equivalent to qualified certificates, which allows for convenient signature testing without the need for a qualified certificate.

The KSeF Client library ( [csharp]((https://github.com/CIRFMF/ksef-client-csharp)) , [java]((https://github.com/CIRFMF/ksef-client-java)) ) has the functionality of creating a digital signature in the XAdES format.

Once the XML document is signed, it should be sent to the KSeF system to obtain a temporary token ( `authenticationToken` ).

Detailed information on supported XAdES signature formats and requirements for qualified certificate attributes can be found [here](auth/podpis-xades.md) .

Example in `C#` :

Generating a test certificate (usable only in the test environment) of an individual with sample identifiers: [KSeF.Client.Tests.Core\E2E\KsefToken\KsefTokenE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/KsefToken/KsefTokenE2ETests.cs)

```csharp
X509Certificate2 ownerCertificate = CertificateUtils.GetPersonalCertificate("Jan", "Kowalski", "TINPL", ownerNip, "M B");

//CertificateUtils.GetPersonalCertificate
public static X509Certificate2 GetPersonalCertificate(
    string givenName,
    string surname,
    string serialNumberPrefix,
    string serialNumber,
    string commonName,
    EncryptionMethodEnum encryptionType = EncryptionMethodEnum.Rsa)
{
    X509Certificate2 certificate = SelfSignedCertificateForSignatureBuilder
                .Create()
                .WithGivenName(givenName)
                .WithSurname(surname)
                .WithSerialNumber($"{serialNumberPrefix}-{serialNumber}")
                .WithCommonName(commonName)
                .AndEncryptionType(encryptionType)
                .Build();
    return certificate;
}
```

Generating a test certificate (usable only in the test environment) of the organization with sample identifiers:

```csharp
// Odpowiednik certyfikatu kwalifikowanego organizacji (tzw. pieczęć firmowa)
X509Certificate2 euEntitySealCertificate = CertificateUtils.GetCompanySeal("Kowalski sp. z o.o", euEntityNipVatEu, "Kowalski");

//CertificateUtils.GetCompanySeal
public static X509Certificate2 GetCompanySeal(
    string organizationName,
    string organizationIdentifier,
    string commonName)
{
    X509Certificate2 certificate = SelfSignedCertificateForSealBuilder
                .Create()
                .WithOrganizationName(organizationName)
                .WithOrganizationIdentifier(organizationIdentifier)
                .WithCommonName(commonName)
                .Build();
    return certificate;
}
```

Using `ISignatureService` and having a certificate with a private key to sign the document:

Example in `C#` :

[KSeF.Client.Tests.Utils\AuthenticationUtils.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Utils/AuthenticationUtils.cs)

```csharp
string unsignedXml = AuthenticationTokenRequestSerializer.SerializeToXmlString(authTokenRequest);

string signedXml = signatureService.Sign(unsignedXml, certificate);
```

`Java` example: [BaseIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/configuration/BaseIntegrationTest.java)

Generating a test certificate (only usable in a test environment) of the organization with sample identifiers

For organizations

```java
SelfSignedCertificate cert = certificateService.getCompanySeal("Kowalski sp. z o.o", "VATPL-" + subject, "Kowalski", encryptionMethod);
```

Or for a private person

```java
SelfSignedCertificate cert = certificateService.getPersonalCertificate("M", "B", "TINPL", ownerNip,"M B",encryptionMethod);
```

Using SignatureService and having a certificate with a private key, you can sign a document

```java
String xml = AuthTokenRequestSerializer.authTokenRequestSerializer(authTokenRequest);

String signedXml = signatureService.sign(xml.getBytes(), cert.certificate(), cert.getPrivateKey());
```

#### 3. Sending signed XML

After signing the AuthTokenRequest document, it should be sent to the KSeF system using an endpoint call<br> POST [/auth/xades-signature](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Uwierzytelnianie/paths/~1api~1v2~1auth~1xades-signature/post) .<br> Since the authentication process is asynchronous, the response returns a temporary JWT ( `authenticationToken` ) along with a reference number ( `referenceNumber` ). Both identifiers are used to:

- checking the status of the authentication process,
- downloading the appropriate access token ( `accessToken` ) in JWT format.

Example in `C#` :

```csharp
SignatureResponse authOperationInfo = await ksefClient.SubmitXadesAuthRequestAsync(signedXml, verifyCertificateChain: false);
```

`Java` example: [BaseIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/configuration/BaseIntegrationTest.java)

```java
SignatureResponse submitAuthTokenResponse = ksefClient.submitAuthTokenRequest(signedXml, false);
```

### 2.2. **KSeF token** authentication

The KSeF token authentication variant requires sending **an encrypted string** consisting of the KSeF token and a timestamp received in the challenge. The token serves as the actual authentication secret, while the timestamp acts as a nonce (IV), ensuring the freshness of the operation and preventing the ciphertext from being recreated in subsequent sessions.

#### 1. Preparing and encrypting the token

String in the format:

```csharp
{tokenKSeF}|{timestamp_z_challenge}
```

should be encrypted with the KSeF public key, using the `RSA-OAEP` algorithm with the `SHA-256 (MGF1)` hash function. The resulting ciphertext should be encoded in `Base64` .

`C#` example: [KSeF.Client.Tests.Core\E2E\KsefToken\KsefTokenE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/KsefToken/KsefTokenE2ETests.cs)

```csharp
AuthChallengeResponse challenge = await KsefClient.GetAuthChallengeAsync();
long timestampMs = challenge.Timestamp.ToUnixTimeMilliseconds();

// Przygotuj "token|timestamp" i zaszyfruj RSA-OAEP SHA-256 zgodnie z wymaganiem API
string tokenWithTimestamp = $"{ksefToken}|{timestampMs}";
byte[] tokenBytes = System.Text.Encoding.UTF8.GetBytes(tokenWithTimestamp);
byte[] encrypted = CryptographyService.EncryptKsefTokenWithRSAUsingPublicKey(tokenBytes);
string encryptedTokenB64 = Convert.ToBase64String(encrypted);
```

`Java` example: [KsefTokenIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/KsefTokenIntegrationTest.java)

```java
AuthenticationChallengeResponse challenge = ksefClient.getAuthChallenge();
byte[] encryptedToken = switch (encryptionMethod) {
    case Rsa -> defaultCryptographyService
            .encryptKsefTokenWithRSAUsingPublicKey(ksefToken.getToken(), challenge.getTimestamp());
    case ECDsa -> defaultCryptographyService
            .encryptKsefTokenWithECDsaUsingPublicKey(ksefToken.getToken(), challenge.getTimestamp());
};
```

#### 2. Sending an authentication request [with a KSeF token](tokeny-ksef.md)

The encrypted Ksef token should be sent together with

Key | Value
--- | ---
Challenge | `Wartość otrzymana z wywołania /auth/challenge`
Context | `Identyfikator kontekstu, dla którego realizowane jest uwierzytelnienie (NIP, identyfikator wewnętrzny, identyfikator złożony VAT UE)`
(optional) AuthorizationPolicy | `Reguły dotyczące walidacji adresu IP klienta podczas korzystania z wydanego tokena dostępu (accessTokena).`

using the endpoint call:

POST [/auth/ksef-token](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Uwierzytelnianie/paths/~1api~1v2~1auth~1ksef-token/post) .<br>

`C#` example: [KSeF.Client.Tests.Core\E2E\KsefToken\KsefTokenE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/KsefToken/KsefTokenE2ETests.cs)

```csharp
// Sposób 1: Budowa zapytania za pomocą buildera
var builder = AuthKsefTokenRequestBuilder
    .Create()
    .WithChallenge(challenge)
    .WithContext(contextIdentifierType, contextIdentifierValue)
    .WithEncryptedToken(encryptedToken);
var authKsefTokenRequest = builder.Build();

// Sposób 2: manualne tworzenie obiektu
AuthenticationKsefTokenRequest request = new AuthenticationKsefTokenRequest
{
    Challenge = challenge.Challenge,
    ContextIdentifier = new AuthenticationTokenContextIdentifier
    {
        Type = AuthenticationTokenContextIdentifierType.Nip,
        Value = nip
    },
    EncryptedToken = encryptedTokenB64,
    AuthorizationPolicy = null
};

SignatureResponse signature = await KsefClient.SubmitKsefTokenAuthRequestAsync(request, CancellationToken);
```

`Java` example: [KsefTokenIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/KsefTokenIntegrationTest.java)

```java
 AuthKsefTokenRequest authTokenRequest = new AuthKsefTokenRequestBuilder()
        .withChallenge(challenge.getChallenge())
        .withContextIdentifier(new ContextIdentifier(ContextIdentifier.IdentifierType.NIP, contextNip))
        .withEncryptedToken(Base64.getEncoder().encodeToString(encryptedToken))
        .build();

SignatureResponse response = ksefClient.authenticateByKSeFToken(authTokenRequest);
```

Since the authentication process is asynchronous, a temporary operational token ( `authenticationToken` ) is returned in response along with a reference number ( `referenceNumber` ). Both identifiers are used to:

- checking the status of the authentication process,
- downloading the appropriate access token (accessToken) in JWT format.

### 3. Checking the authentication status

After sending the signed XML document ( `AuthTokenRequest` ) and receiving the response containing `authenticationToken` and `referenceNumber` , you should check the status of the ongoing authentication operation by specifying &lt;authenticationToken&gt; in the `Authorization` Bearer header.<br> GET [/auth/{referenceNumber}](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Uwierzytelnianie/paths/~1api~1v2~1auth~1%7BreferenceNumber%7D/get) The response returns the status – the code and description of the operation status (e.g. "Authentication in progress", Authentication "completed successfully").

**Attention**
 In pre-production and production environments, in addition to verifying the XAdES signature, the system checks the current status of the certificate with its issuer (OCSP/CRL services). Until a binding response is received from the certificate provider, the operation status will return "Authentication in progress"—this is a normal consequence of the verification process and does not indicate a system error. Status checking is asynchronous; the result must be queried until successful. The verification time depends on the certificate issuer.

**Recommendation for the production environment - KSeF certificate**
 To eliminate the wait for certificate status verification in OCSP/CRL services by qualified trust service providers, authentication [with a KSeF certificate](certyfikaty-KSeF.md) is recommended. KSeF certificate verification takes place internally and occurs immediately after receiving the signature.

**Error handling**
 If unsuccessful, the response may include error codes related to invalid signatures, lack of authorization, or technical issues. A detailed list of error codes will be available in the endpoint's technical documentation.

`C#` example: [KSeF.Client.Tests.Core\E2E\KsefToken\KsefTokenE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/KsefToken/KsefTokenE2ETests.cs)

```csharp
AuthStatus status;
status = await KsefClient.GetAuthStatusAsync(signature.ReferenceNumber, signature.AuthenticationToken.Token);
```

`Java` example: [BaseIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/configuration/BaseIntegrationTest.java)

```java
AuthStatus authStatus = ksefClient.getAuthStatus(referenceNumber, tempToken);
```

### 4. Obtaining an access token (accessToken)

The endpoint returns a single pair of tokens generated for a successfully completed authentication process. Any subsequent calls with the same `authenticationToken` will return a 400 error.

POST [/auth/token/redeem](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Uwierzytelnianie/paths/~1api~1v2~1auth~1token~1redeem/post)

`C#` example: [KSeF.Client.Tests.Core\E2E\KsefToken\KsefTokenE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/KsefToken/KsefTokenE2ETests.cs)

```csharp
AuthOperationStatusResponse tokens = await KsefClient.GetAccessTokenAsync(signature.AuthenticationToken.Token);
```

`Java` example: [BaseIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/configuration/BaseIntegrationTest.java)

```java
AuthOperationStatusResponse tokenResponse = ksefClient.redeemToken(response.getAuthenticationToken().getToken());
```

The following is returned in response:

- `accessToken` – JWT access token used to authorize operations in the API (in the Authorization: Bearer ... header), has a limited validity period (e.g. several minutes, specified in the exp field),
- `refreshToken` – a token that allows you to refresh `accessToken` without re-authentication, has a much longer validity period (up to 7 days) and can be used multiple times to refresh the access token.

**Attention!**

1. `accessToken` and `refreshToken` should be treated as confidential data – its storage requires appropriate security measures.
2. The access token ( `accessToken` ) remains valid until the date specified in the `exp` field, even if the user's permissions change.

#### 5. Refreshing the access token ( `accessToken` )

To maintain continuous access to protected API resources, the KSeF system provides a mechanism for refreshing the access token ( `accessToken` ) using a special refresh token ( `refreshToken` ). This solution eliminates the need to repeat the full authentication process each time, but also improves system security – `accessToken` short lifetime reduces the risk of unauthorized use if intercepted.

POST [/auth/token/refresh](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Uwierzytelnianie/paths/~1api~1v2~1auth~1token~1refresh/post)<br> `RefreshToken` should be passed in the Authorization header in the format:

```
Authorization: Bearer {refreshToken}
```

The response contains a new `accessToken` (JWT) with the current set of permissions and roles.

Example in `C#` :

```csharp
RefreshTokenResponse refreshedAccessTokenResponse = await ksefClient.RefreshAccessTokenAsync(accessTokenResult.RefreshToken.Token);
```

`Java` example: [AuthorizationIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/AuthorizationIntegrationTest.java)

```java
AuthenticationTokenRefreshResponse refreshTokenResult = ksefClient.refreshAccessToken(initialRefreshToken);
```

#### 6. Managing authentication sessions

For details about managing active authentication sessions, see [Managing Sessions](auth/sesje.md) .

Related documents:

- [KSeF certificates](certyfikaty-KSeF.md)
- [XAdES Test Certificates and Signatures](auth/testowe-certyfikaty-i-podpisy-xades.md)
- [Signature XAdES](auth/podpis-xades.md)
- [KSeF Token](tokeny-ksef.md)
