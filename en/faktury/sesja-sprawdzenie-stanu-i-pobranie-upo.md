## Session – checking the status and downloading the UPO

10/07/2025

This document describes the operations for monitoring the status of a session (interactive or batch) and downloading UPOs for invoices and the entire session.

### 1. Downloading the session list

Returns a list of sessions that match the given search criteria.

GET [sessions](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Status-wysylki-i-UPO/paths/~1api~1v2~1sessions/get)

Returns the current session status along with aggregated data on the number of invoices sent, correctly and incorrectly processed; after closing the session, it also provides a list of references to the collective UPO.

Example in C#:

```csharp
// Pobieranie sesji wsadowych
 var sessions = new List<Session>();
 const int pageSize = 20;
 string? continuationToken = null;
 do
 {
     var response = await ksefClient.GetSessionsAsync(SessionType.Batch, accessToken, pageSize, continuationToken, sessionsFilter, cancellationToken);
     continuationToken = response.ContinuationToken;
     sessions.AddRange(response.Sessions);
 } while (!string.IsNullOrEmpty(continuationToken));

// Pobieranie sesji interaktywnych
 var sessions = new List<Session>();
 const int pageSize = 20;
 string? continuationToken = null;
 do
 {
     var response = await ksefClient.GetSessionsAsync(SessionType.Online, accessToken, pageSize, continuationToken, sessionsFilter, cancellationToken);
     continuationToken = response.ContinuationToken;
     sessions.AddRange(response.Sessions);
 } while (!string.IsNullOrEmpty(continuationToken));
```

Java example: [SessionIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/SessionIntegrationTest.java)

```java
SessionsQueryRequest request = new SessionsQueryRequest();
request.setSessionType(SessionType.ONLINE);
request.setStatuses(List.of(CommonSessionStatus.INPROGRESS));

SessionsQueryResponse sessionsQueryResponse = ksefClient.getSessions(request, pageSize, continuationToken, accessToken();

while (Strings.isNotBlank(activeSessions.getContinuationToken())) {
        sessionsQueryResponse = ksefClient.getSessions(pageSize, sessionsQueryResponse.getContinuationToken(), accessToken);
}
```

### 2. Checking the session status

Checks the current session status.

GET [sessions/{referenceNumber}](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Status-wysylki-i-UPO/paths/~1api~1v2~1sessions~1%7BreferenceNumber%7D/get)

Returns the current session status along with aggregated data on the number of invoices sent, correctly and incorrectly processed; after closing the session, it also provides a list of references to the collective UPO.

Example in C#:

```csharp
var openSessionResult = await kSeFClient.GetSessionStatusAsync(referenceNumber, accessToken, cancellationToken);
var documentCount = openSessionResult.InvoiceCount;
var successfulInvoiceCount = openSessionResult.SuccessfulInvoiceCount;
var failedInvoiceCount = openSessionResult.FailedInvoiceCount;
```

Java example: [OnlineSessionIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/OnlineSessionIntegrationTest.java)

```java
SessionStatusResponse statusResponse = ksefClient.getSessionStatus(sessionReferenceNumber, accessToken);
```

### 3. Downloading information about sent invoices

GET [sessions/{referenceNumber}/invoices](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Status-wysylki-i-UPO/paths/~1api~1v2~1sessions~1%7BreferenceNumber%7D~1invoices/get)

Returns a list of metadata for all submitted invoices along with their statuses and the total number of these invoices in the session.

Example in C#:

```csharp
const int pageSize = 50;
string continuationtoken = null;

do
{
    var sessionInvoices = await ksefClient
                                .GetSessionInvoicesAsync(
                                referenceNumber,
                                accessToken,
                                pageOffset,
                                pageSize,
                                cancellationToken);

    foreach (var doc in getInvoicesResult.Invoices)
    {
        Console.WriteLine($"#{doc.InvoiceNumber}. Status: {doc.Status.Code}");
    }

    continuationtoken = sessionInvoices.ContinuationToken;
}
while (continuationtoken != null);

```

Java example: [OnlineSessionIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/OnlineSessionIntegrationTest.java)

```java
SessionInvoicesResponse sessionInvoices = ksefClient.getSessionInvoices(referenceNumber,continuationtoken, pageSize, authToken);

while (Strings.isNotBlank(sessionInvoices.getContinuationToken())) {
    sessionInvoices = ksefClient.getSessions(pageSize, sessionInvoices.getContinuationToken(), accessToken);
}
```

### 4. Downloading information about a single invoice

Allows you to retrieve detailed information about a single invoice in a session, including its status and metadata.

You must provide the session reference number `referenceNumber` and the invoice reference number `invoiceReferenceNumber` .

GET [sessions/{referenceNumber}/invoices/{invoiceReferenceNumber}](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Status-wysylki-i-UPO/paths/~1api~1v2~1sessions~1%7BreferenceNumber%7D~1invoices~1%7BinvoiceReferenceNumber%7D/get)

Example in C#:

```csharp
var invoice = await ksefClient
                .GetSessionInvoiceAsync(
                referenceNumber,
                invoiceReferenceNumber,
                accessToken,
                cancellationToken);
```

Java example: [QueryInvoiceIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/QueryInvoiceIntegrationTest.java)

```java
SessionInvoiceStatusResponse statusResponse = ksefClient.getSessionInvoiceStatus(sessionReferenceNumber, invoiceReferenceNumber, accessToken);

```

### 5. Downloading the UPO for the invoice

Allows you to download a UPO for a single, correctly accepted invoice.

#### 5.1 Based on the invoice reference number

GET [sessions/{referenceNumber}/invoices/{invoiceReferenceNumber}/upo](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Status-wysylki-i-UPO/paths/~1api~1v2~1sessions~1%7BreferenceNumber%7D~1invoices~1%7BinvoiceReferenceNumber%7D~1upo/get)

Example in C#:

```csharp
var upo = await ksefClient
                .GetSessionInvoiceUpoByReferenceNumberAsync(
                referenceNumber,
                invoiceReferenceNumber,
                accessToken,
                cancellationToken)
```

Java example: [OnlineSessionIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/OnlineSessionIntegrationTest.java)

```java
byte[] upoResponse = ksefClient.getSessionInvoiceUpoByReferenceNumber(sessionReferenceNumber, invoiceReferenceNumber, accessToken);
```

#### 5.2 Based on the KSeF invoice number

GET [sessions/{referenceNumber}/invoices/{ksefNumber}/upo](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Status-wysylki-i-UPO/paths/~1api~1v2~1sessions~1%7BreferenceNumber%7D~1invoices~1ksef~1%7BksefNumber%7D~1upo/get)

Example in C#:

```csharp
var upo = await ksefClient
                .GetSessionInvoiceUpoByKsefNumberAsync(
                referenceNumber,
                ksefNumber,
                accessToken,
                cancellationToken)
```

Java example: [OnlineSessionIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/OnlineSessionIntegrationTest.java)

```java
byte[] upoResponse = ksefClient.getSessionInvoiceUpoByKsefNumber(sessionReferenceNumber, ksefNumber, accessToken);
```

The resulting XML document is:

- signed in the XADES format by the Ministry of Finance
- compliant with the [XSD](/faktury/upo/schemy/upo-v4-2.xsd) schema.

### 6. Downloading a list of incorrectly accepted invoices

GET [sessions/{referenceNumber}/invoices/failed](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Status-wysylki-i-UPO/paths/~1api~1v2~1sessions~1%7BreferenceNumber%7D~1invoices~1failed/get)

Returns the total number of rejected invoices in the session and detailed information (status and error details) for each incorrectly processed invoice.

Example in C#:

```csharp
const int pageSize = 50;
string continuationToken = "";

do
{
    var sessionInvoices = await ksefClient
                                .GetSessionFailedInvoicesAsync(
                                referenceNumber,
                                accessToken,
                                pageSize,
                                continuationToken,
                                cancellationToken);

    continuationToken = failedResult.Invoices.ContinuationToken

}
while (!string.IsNullOrEmpty(continuationToken));
```

Java example: [SessionController.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/main/java/pl/akmf/ksef/sdk/SessionController.java)

```java
SessionInvoicesResponse response = ksefClient.getSessionFailedInvoices(referenceNumber, continuationToken, pageSize, authToken);

while (Strings.isNotBlank(response.getContinuationToken())) {
        response = ksefClient.getSessionFailedInvoices(pageSize, response.getContinuationToken(), accessToken);
}
```

Endpoint allows you to selectively download only rejected invoices, which facilitates error analysis in sessions containing a large number of invoices.

### 7. Downloading the UPO session

The session UPO is a collective confirmation of receipt of all invoices correctly submitted within a given session.

After closing the session, in response to checking its [status](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Status-wysylki-i-UPO/paths/~1api~1v2~1sessions~1%7BreferenceNumber%7D/get) (step 2 – Checking the session status), not only information about the number of correctly and incorrectly processed invoices is returned, but also a list of references to collective UPOs.

Each element of `upo.pages[]` array contains a UPO reference number ( `referenceNumber` ) and a link ( `downloadUrl` ) enabling its download:

```json
"upo": {
    "pages": [
        {
            "referenceNumber": "20250901-EU-47FDBE3000-5961A5D232-BF",
            "downloadUrl": "/api/v2/sessions/20250901-SB-47FA636000-5960B49115-9D/upo/20250901-EU-47FDBE3000-5961A5D232-BF"
        },
        {
            "referenceNumber": "20250901-EU-48D8488000-59667BB54C-C8",
            "downloadUrl": "/api/v2/sessions/20250901-SB-47FA636000-5960B49115-9D/upo/20250901-EU-48D8488000-59667BB54C-C8"
        }
    ]
}

```

With this list, the API client can download UPOs one by one by calling the endpoint indicated in the `downloadUrl` field, i.e.
 GET [/sessions/{referenceNumber}/upo/{upoReferenceNumber}](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Status-wysylki-i-UPO/paths/~1api~1v2~1sessions~1%7BreferenceNumber%7D~1upo~1%7BupoReferenceNumber%7D/get)

The resulting XML document complies with the [XSD](/faktury/upo/schemy/upo-v4-2.xsd) schema and can contain a maximum of 10,000 invoice items.

Example in C#:

```csharp
 var upo = await ksefClient.GetSessionUpoAsync(
            sessionReferenceNumber,
            upoReferenceNumber,
            accessToken,
            cancellationToken
        );
```

Java example: [OnlineSessionIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/OnlineSessionIntegrationTest.java)

```java
byte[] sessionUpo = ksefClient.getSessionUpo(sessionReferenceNumber, upoReferenceNumber, accessToken);
```

## Related documents

- [KSeF number – structure and validation](numer-ksef.md)
