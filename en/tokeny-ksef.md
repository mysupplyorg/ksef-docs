## KSeF token management

29/06/2025

The KSeF token is a unique, generated authentication identifier that, in a similar way to [a qualified electronic signature](uwierzytelnianie.md#21-uwierzytelnianie-kwalifikowanym-podpisem-elektronicznym) , allows [authentication](uwierzytelnianie.md#22-uwierzytelnianie-tokenem-ksef) to the KSeF API.

`Token KSeF` is issued with an immutable set of permissions specified at its creation; any modification to these permissions requires the generation of a new token.

> **Attention!**<br> `Token KSeF` acts as **a confidential authentication secret** - it should only be stored in a trusted and secure store.

### Prerequisites

Generating `tokena KSeF` is only possible after one-time authentication [with an electronic signature (XAdES)](uwierzytelnianie.md#21-uwierzytelnianie-kwalifikowanym-podpisem-elektronicznym) .

### 1. Generating a token

The token can only be generated in the context of `Nip` or `InternalId` . Generation is performed by calling the endpoint:
 POST [/tokens](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Tokeny/paths/~1api~1v2~1tokens/post)

By providing a collection of permissions and a token description in the request body.

**Implementation examples:** <br>

Field | Sample value | Description
--- | --- | ---
Permissions | `["InvoiceRead", "InvoiceWrite", "CredentialsRead", "CredentialsManage"]` | List of permissions assigned to the token
Description | `"Token do odczytu faktur i danych konta"` | Token description

Example in C#:

```csharp
 var tokenRequest = new KsefTokenRequest
    {
        Permissions = [
            KsefTokenPermissionType.InvoiceRead,
            KsefTokenPermissionType.InvoiceWrite
            ],
        Description = "Demo token",
    };
 var token = await ksefClient.GenerateKsefTokenAsync(tokenRequest, accessToken, cancellationToken);
```

Java example: [KsefTokenIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/configuration/KsefTokenIntegrationTest.java)

```java
KsefTokenRequest request = new KsefTokenRequestBuilder()
        .withDescription("test description")
        .withPermissions(List.of(TokenPermissionType.INVOICE_READ, TokenPermissionType.INVOICE_WRITE))
        .build();
GenerateTokenResponse ksefToken = ksefClient.generateKsefToken(request, authToken.accessToken());
```

### 2. Token Filtering

KSeF token metadata can be retrieved and filtered using the call:<br> GET [/tokens](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Tokeny/paths/~1api~1v2~1tokens/get)

Example in C#:

```csharp
var result = new List<AuthenticationKsefToken>();
const int pageSize = 20;
var status = AuthenticationKsefTokenStatus.Active;
var continuationToken = string.Empty;

do
 {
     var tokens = await ksefClient.QueryKsefTokensAsync(accessToken, [status], continuationToken, pageSize, cancellationToken);
     result.AddRange(tokens.Tokens);
     continuationToken = tokens.ContinuationToken;
 } while (!string.IsNullOrEmpty(continuationToken));
```

Java example: [KsefTokenIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/KsefTokenIntegrationTest.java)

```java
List<AuthenticationTokenStatus> status = List.of(AuthenticationTokenStatus.ACTIVE);
Integer pageSize = 10;
QueryTokensResponse tokens = ksefClient.queryKsefTokens(status, StringUtils.EMPTY, null, null, null, pageSize, accessToken);
```

The response returns token metadata, including information about who and in what context generated the KSef token and the permissions assigned to it.

### 3. Downloading a specific token

To retrieve the details of a specific token, use the call:<br> GET [/tokens/{referenceNumber}](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Tokeny/paths/~1api~1v2~1tokens~1%7BreferenceNumber%7D/get)

`referenceNumber` is the unique identifier of the token, which can be obtained when it is created or from the token list.

Example in C#:

```csharp
var token = await ksefClient.GetKsefTokenAsync(referenceNumber, accessToken, cancellationToken);
```

Java example: [KsefTokenIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/KsefTokenIntegrationTest.java)

```java
AuthenticationToken ksefToken = ksefClient.getKsefToken(token.getReferenceNumber(), accessToken);
```

### 4. Token invalidation

To invalidate a token, use the call:<br> DELETE [/tokens/{referenceNumber}](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Tokeny/paths/~1api~1v2~1tokens~1%7BreferenceNumber%7D/delete)

`referenceNumber` is the unique identifier of the token we want to invalidate.

Example in C#:

```csharp
await ksefClient.RevokeKsefTokenAsync(referenceNumber, accessToken, cancellationToken);
```

Java example: [KsefTokenIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/KsefTokenIntegrationTest.java)

```java
ksefClient.revokeKsefToken(token.getReferenceNumber(), accessToken);
```
