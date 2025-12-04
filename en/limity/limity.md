# Limits

21/10/2025

## Entry

The KSeF 2.0 system implements mechanisms that limit the number and size of API operations and parameters related to transmitted data. The purpose of these limits is to:

- protection of system stability on a large scale of operation,
- counteracting abuses and ineffective integrations,
- preventing abuse and potential cybersecurity threats,
- ensuring equal access conditions for all users.

The limits have been designed with the possibility of flexible adjustment to the needs of specific entities requiring more intensive operations.

## API request limits

The KSeF system limits the number of requests that can be sent in a short period of time to ensure stable system operation and equal access for all users. For more information, see [API Request Limits](limity-api.md) .

## Context Limits

Parameter | Default value
--- | ---
Maximum invoice size without attachment | 1 MB
Maximum invoice size with attachment | 3 MB
Maximum number of invoices in an interactive/batch session | 10,000

## Limits per authenticated entity

### Applications and active certificates

ID from certificate | Applications for KSeF certificate | Active KSeF certificates
--- | --- | ---
Tax Identification Number | 300 | 100
PESEL | 6 | 2
Certificate fingerprint | 6 | 2

## Adjusting limits

The KSeF system enables individual adjustment of selected technical limits for:

- API limits - e.g. increasing the number of requests for a selected endpoint,
- context - e.g. increasing the maximum invoice size,
- authenticating entity - e.g. increasing the limits of active KSeF certificates for a natural person (PESEL).

In **a production environment,** increasing limits is only possible upon a substantiated request, supported by a genuine operational need. The request should be submitted via [the contact form](https://ksef.podatki.gov.pl/formularz/) , along with a detailed description of the intended use.

## Checking individual limits

The KSeF system provides endpoints that allow you to check the current limit values for the current context or entity:

### Get limits for the current context

GET [/limits/context](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Limity-i-ograniczenia/paths/~1api~1v2~1limits~1context/get)

Returns the values of the effective interactive and batch session limits for the current context.

Java example:

[ContextLimitIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/ContextLimitIntegrationTest.java)

```java
GetContextLimitResponse response = ksefClient.getContextSessionLimit(accessToken);
```

### Downloading limits for the current entity

GET [/limits/subject](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Limity-i-ograniczenia/paths/~1api~1v2~1limits~1subject/get)

Returns the applicable certificate and certificate request limits for the current authenticator.

Java example:

[SubjectLimitIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/SubjectLimitIntegrationTest.java)

```java
GetSubjectLimitResponse response = ksefClient.getSubjectCertificateLimit(accessToken);
```

## Modifying limits in the test environment

A set of methods for changing and restoring limits to their default values is available in **the test environment** . These operations are available only to authenticated entities and have no impact on the production environment.

### Changing session limits for the current context

POST [/testdata/limits/context/session](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Limity-i-ograniczenia/paths/~1api~1v2~1testdata~1limits~1context~1session/post)

Java example:

[ContextLimitIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/ContextLimitIntegrationTest.java)

```java
ChangeContextLimitRequest request = new ChangeContextLimitRequest();
OnlineSessionLimit onlineSessionLimit = new OnlineSessionLimit();
onlineSessionLimit.setMaxInvoiceSizeInMB(4);
onlineSessionLimit.setMaxInvoiceWithAttachmentSizeInMB(5);
onlineSessionLimit.setMaxInvoices(6);

BatchSessionLimit batchSessionLimit = new BatchSessionLimit();
batchSessionLimit.setMaxInvoiceSizeInMB(4);
batchSessionLimit.setMaxInvoiceWithAttachmentSizeInMB(5);
batchSessionLimit.setMaxInvoices(6);

request.setOnlineSession(onlineSessionLimit);
request.setBatchSession(batchSessionLimit);

ksefClient.changeContextLimitTest(request, accessToken);
```

### Reset context session limits to default values

DELETE [/testdata/limits/context/session](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Limity-i-ograniczenia/paths/~1api~1v2~1testdata~1limits~1context~1session/delete)

[ContextLimitIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/ContextLimitIntegrationTest.java)

```java
ksefClient.resetContextLimitTest(accessToken);
```

### Changing certificate limits for the current entity

POST [/testdata/limits/subject/certificate](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Limity-i-ograniczenia/paths/~1api~1v2~1testdata~1limits~1subject~1certificate/post)

[SubjectLimitIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/SubjectLimitIntegrationTest.java)

```java
ChangeSubjectCertificateLimitRequest request = new ChangeSubjectCertificateLimitRequest();
request.setCertificate(new CertificateLimit(15));
request.setEnrollment(new EnrollmentLimit(15));
request.setSubjectIdentifierType(ChangeSubjectCertificateLimitRequest.SubjectType.NIP);

ksefClient.changeSubjectLimitTest(request, accessToken);
```

### Resetting entity certificate limits to default values

DELETE [/testdata/limits/subject/certificate](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Limity-i-ograniczenia/paths/~1api~1v2~1testdata~1limits~1subject~1certificate/delete)

[SubjectLimitIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/SubjectLimitIntegrationTest.java)

```java
ksefClient.resetSubjectCertificateLimit(accessToken);
```

Related documents:

- [API request limits](limity-api.md)
- [Invoice verification](../faktury/weryfikacja-faktury.md)
- [KSeF certificates](../certyfikaty-KSeF.md)
