## Managing authentication sessions

10/07/2025

### Retrieving a list of active authentication sessions

Returns a list of active authentication sessions.

GET [/auth/sessions](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Aktywne-sesje/paths/~1api~1v2~1auth~1sessions/get)

Example in `C#` :

```csharp
const int pageSize = 20;
string? continuationToken = null;
var activeSessions = new List<Item>();
do
{
    var response = await ksefClient.GetActiveSessions(accessToken, pageSize, continuationToken, cancellationToken);
    continuationToken = response.ContinuationToken;
    activeSessions.AddRange(response.Items);
}
while (!string.IsNullOrWhiteSpace(continuationToken));
```

`Java` example: [SessionIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/SessionIntegrationTest.java)

```java
int pageSize = 10;
AuthenticationListResponse activeSessions = createKSeFClient().getActiveSessions(10, null, accessToken);
while (Strings.isNotBlank(activeSessions.getContinuationToken())) {
    activeSessions = createKSeFClient().getActiveSessions(10, activeSessions.getContinuationToken(), accessToken);
}
```

### Invalidating the current session

DELETE [`/auth/sessions/current`](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Aktywne-sesje/paths/~1api~1v2~1auth~1sessions~1current/delete)

Invalidates the session associated with the token used to invoke this endpoint. After the operation:

- the associated `refreshToken` is invalidated,
- Active `accessTokeny` remain valid until their expiration date.

Example in `C#` :

```csharp
await ksefClient.RevokeCurrentSessionAsync(token, cancellationToken);
```

`Java` example: [SessionIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/SessionIntegrationTest.java)

```java
createKSeFClient().revokeCurrentSession(accessToken);
```

### Invalidation of the selected session

DELETE [`/auth/sessions/{referenceNumber}`](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Aktywne-sesje/paths/~1api~1v2~1auth~1sessions~1%7BreferenceNumber%7D/delete)

Invalidates the session with the specified reference number. After the operation:

- the associated `refreshToken` is invalidated,
- Active `accessTokeny` remain valid until their expiration date.

Example in `C#` :

```csharp
await ksefClient.RevokeSessionAsync(referenceNumber, accessToken, cancellationToken);
```

`Java` example: [SessionIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/SessionIntegrationTest.java)

```java
createKSeFClient().revokeSession(secondSessionReferenceNumber, firstAccessTokensPair.accessToken());
```
