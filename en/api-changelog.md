## Changes in API 2.0

### Version 2.0.0 RC6.0

- **API limits**

    - On the **TE** (test) environment, [an API limits](limity/limity-api.md) policy was enabled and defined with values 10x higher than on **PRD** ; details: ["Limits on environments"](/limity/limity-api.md#limity-na-%C5%9Brodowiskach) .
    - [API limits](limity/limity-api.md) have been enabled on the **TR** (DEMO) environment with values identical to those on **PRD** . The values are replicated from production; see ["Limits on environments"](/limity/limity-api.md#limity-na-%C5%9Brodowiskach) for details.
    - Added endpoint POST `/testdata/rate-limits/production` - sets API limit values in the current context according to the production profile. Available only in **TE** environments.

- **Exporting a batch of invoices (POST `/invoices/exports` ). Downloading a list of invoice metadata (POST /invoices/query/metadata)**

    - Added [High Water Mark (HWM)](pobieranie-faktur/hwm.md) document describing the mechanism for managing data completeness over time.
    - Updated [Incremental Invoice Download](pobieranie-faktur/przyrostowe-pobieranie-faktur.md) with recommendations for using the `HWM` mechanism.
    - The response model has been extended with `permanentStorageHwmDate` property (string, date-time, nullable). This only applies to queries with `dateType = PermanentStorage` and denotes the point below which the data is complete; for `dateType = Issue/Invoicing` - null.
    - Added `restrictToPermanentStorageHwmDate` (boolean, nullable) property on `dateRange` object, which enables High Water Mark ( `HWM` ) mechanism and limits the date range to the current `HWM` value. Applies only to queries with `dateType = PermanentStorage` . Using the parameter significantly reduces duplicates between subsequent exports and ensures consistency during long-term incremental synchronization.

- **UPO - XSD update to v4-3**

    - The `pattern` `NumerKSeFDokumentu` element has been changed to also allow KSeF numbers generated for invoices from KSeF 1.0 (36 characters).
    - `TrybWysylki` element has been added - the mode of sending the document to KSeF: `Online` or `Offline` .
    - `NazwaStrukturyLogicznej` value has been changed to the format: Schemat_{systemCode}_v{schemaVersion}.xsd (e.g. Schemat_FA(3)_v1-0E.xsd).
    - The value of `NazwaPodmiotuPrzyjmujacego` on test environments has been changed by adding a suffix with the environment name:
        - `TE` : Ministry of Finance - test environment (TE),
        - `TR` : Ministry of Finance - pre-production environment (TR).

    `PRD` : no changes - Ministry of Finance.

    - Currently, UPO v4-2 is returned by default. To receive UPO v4-3, you need to add the header: `X-KSeF-Feature: upo-v4-3` when opening a session (online/batch).
    - From `2025-12-22` the default version will be UPO v4-3.
    - XSD UPO v4-3: [schema](/faktury/upo/schemy/upo-v4-3.xsd) .

- **Session status** (GET `/sessions/{referenceNumber}` )
     Clarified the description of code `440` - Session canceled: possible reasons are "Shipping time exceeded" or "Invoices not sent".

- **Invoice status** (GET `/sessions/{referenceNumber}/invoices/{invoiceReferenceNumber}` )
     `InvoiceStatusInfo` type (extends `StatusInfo` ) has been added with an `extensions` field—an object with structured status details. `details` field remains unchanged. Example (duplicate invoice):

    ```json
    "status": {
      "code": 440,
      "description": "Duplikat faktury",
      "details": [
        "Duplikat faktury. Faktura o numerze KSeF: 5265877635-20250626-010080DD2B5E-26 została już prawidłowo przesłana do systemu w sesji: 20250626-SO-2F14610000-242991F8C9-B4"
      ],
      "extensions": {
        "originalSessionReferenceNumber": "20250626-SO-2F14610000-242991F8C9-B4",
        "originalKsefNumber": "5265877635-20250626-010080DD2B5E-26"
      }
    }
    ```

- **Right**
     Added `subjectDetails` property - "Data about the subject to whom permissions are granted" to all endpoints granting permissions (/permissions/.../grants). In RC6.0, the field is optional; from 2025-12-19 it will be required.

- **Searching for granted permissions** (POST `/permissions/query/authorizations/grants` )
     Access rules extended to include `PefInvoiceWrite` .

- **Test data - attachments (POST /testdata/attachment/revoke)**
     `AttachmentPermissionRevokeRequest` request model has been extended with the `expectedEndDate` field (optional) - the date of withdrawal of consent to send invoices with an attachment.

- **OpenAPI**

    - Added HTTP `429` - "Too Many Requests" response to all endpoints. The response's `description` property publishes a tabular presentation of the limits ( `req/s` , `req/min` , `req/h` ) and the name of the limit group assigned to the endpoint. The `429` mechanism and semantics remain as described in the [limits](/limity/limity-api.md) documentation.
    - `x-rate-limits` metadata with limit values ( `req/s` , `req/min` , `req/h` ) was added to each endpoint.
    - Removed explicit properties `exclusiveMaximum` : `false` and `exclusiveMinimum` : `false` from numeric definitions (only minimum/maximum remain). This is a cleanup change – it doesn't affect validation (in OpenAPI, the default values for these properties are `false` ).
    - Added length constraints for string properties: `minLength` .
    - Removed explicit `style` : `form` settings for in:query parameters.
    - The order `BuyerIdentifierType` enuma values has been changed (currently: `None` , `Other` , `Nip` , `VatUe` ). This is a clean change - no impact on functionality.
    - Fixed a typo in the `KsefNumber` property description.
    - The format of `PublicKeyCertificate` property representing `Base64` encoded binary data has been clarified, the format has been set to: `byte` .
    - Minor linguistic and punctuation corrections have been made in the `description` fields.

### Version 2.0.0 RC5.7

- **Opening a batch session (POST `/sessions/batch` )**
     Marked `BatchFilePartInfo.fileName` as `deprecated` in the request model (planned removal: 2025-12-05).

- **Asynchronous operation statuses**
     Added status `550` - "The operation was canceled by the system." Description: "Processing was interrupted due to internal system reasons. Please try again."

- **OpenAPI**

    - Added limits on the number of elements in an array: `minItems` , `maxItems` .
    - Added length constraints for string properties: `minLength` and `maxLength` .
    - Updated property descriptions ( `invoiceMetadataAuthorizedSubject.role` , `invoiceMetadataBuyer` , `invoiceMetadataThirdSubject.role` , `buyerIdentifier` ).
    - Updated regex patterns for `vatUeIdentifier` , `authorizedFingerprintIdentifier` , `internalId` , `nipVatUe` , `peppolId` .

### Version 2.0.0 RC5.6

- **Get session status (GET `/sessions/{referenceNumber}` )**
     `UpoPageResponse.downloadUrlExpirationDate` field has been added to the response - the date and time when the UPO download address expires; after this point `downloadUrl` is no longer active.

- **Get certificate metadata list (POST `/certificates/query` )**
     The response ( `CertificateListItem` ) has been extended with `requestDate` property - the date of submitting the certification request.

- **Get the list of Peppol service providers (GET `/peppol/query` )**

    - The response model has been extended with `dateCreated` field - the date of registration of the Peppol service provider in the system.
    - Marked `dateCreated` , `id` , `name` property in the response model as always returned.
    - The `PeppolI` schema (string, 9 characters) was defined and used in `PeppolProvider` .

- **OpenAPI**

    - Added `x-sort` metadata to all endpoints that return lists. Added a Sorting section to endpoint descriptions with default ordering (e.g., "requestDate (Desc)").
    - Added length constraints for string properties: `minLength` and `maxLength` .
    - Clarified the format of properties representing `Base64` encoded binary data: set format: `byte` ( `encryptedInvoiceContent` , `encryptedSymmetricKey` , `initializationVector` , `encryptedToken` ).
    - A common `Sha256HashBase64` scheme was defined and applied to all properties representing a `SHA-256` hash in `Base64` (including `invoiceHash` ).
    - A common `ReferenceNumber` schema (string, length 36) is defined and applied to all parameters and properties representing the reference number of an asynchronous operation (in paths, queries, and responses).
    - A common `Nip` schema (string, 10 characters, regex pattern) was defined and applied to all properties representing NIPs.
    - The `Pesel` schema (string, 11 characters, regex pattern) was defined and used in the property representing the PESEL.
    - A common `KsefNumber` schema (string, 35-36 characters, regex pattern) was defined and applied to all properties representing a KSeF number.
    - `Challenge` scheme (string, 36 characters) defined and used in `AuthenticationChallengeResponse` . `challenge` .
    - A common `PermissionId` scheme (string, 36 characters) was defined and used in all places: in parameters and in response properties.
    - Added regular expressions for selected text fields.

### Version 2.0.0 RC5.5

- **Get current API limits (GET `/api/v2/rate-limits` )**
     Added endpoint returning effective API call limits in `perSecond` / `perMinute` / `perHour` format for individual areas (including `onlineSession` , `batchSession` , `invoiceSend` , `invoiceStatus` , `invoiceExport` , `invoiceDownload` , `other` ).

- **Invoice status in session**
     The response for GET `/sessions/{referenceNumber}/invoices` ("Getting session invoices") and GET `/sessions/{referenceNumber}/invoices/{invoiceReferenceNumber}` ("Getting invoice status from session") has been extended with the property: `upoDownloadUrlExpirationDate` - "date and time of the address expiration. After this date, `UpoDownloadUrl` link will no longer be active". The description `upoDownloadUrl` has been extended.

- ***InMib fields removed (change as announced in 5.3)**
     `maxInvoiceSizeInMib` and `maxInvoiceWithAttachmentSizeInMib` properties have been removed. The change affects:

    - GET `/limits/context` – responses ( `onlineSession` , `batchSession` ),
    - POST `/testdata/limits/context/session` – request model ( `onlineSession` , `batchSession` ),
    - Models: `BatchSessionContextLimitsOverride` , `BatchSessionEffectiveContextLimits` , `OnlineSessionContextLimitsOverride` , `OnlineSessionEffectiveContextLimits` . Only the *InMB fields are used to indicate sizes (1 MB = 1,000,000 B).

- **Removed `operationReferenceNumber` (change as announced in 5.3)**
     `operationReferenceNumber` property has been removed from the response model; the only valid name is `referenceNumber` . The change includes:

    - GET `/invoices/exports/{referenceNumber}` - "Invoice batch export status",
    - POST `/permissions/operations/{referenceNumber}` - "Get permissions operation status".

- **Exporting a batch of invoices (POST `/invoices/exports` )**

    - Added new error code: `21182` - "The limit for ongoing exports has been reached. The maximum limit of {count} concurrent invoice exports has been reached for the authenticated entity in the current context. Please try again later."
    - The response model has been enhanced with `packageExpirationDate` property, which indicates the expiration date of a prepared package. After this date, the package will no longer be available for download.
    - Added error code `210` - "Invoice export has expired and is no longer available for download."

- **Invoice batch export status (GET `/invoices/exports/{referenceNumber}` )**
     The descriptions of the link fields for downloading parts of the package have been clarified:

    - `url` - "URL address to which a request to download a part of the package should be sent. The link is dynamically generated when the export operation status is queried. It is not subject to API limits and does not require sending an access token upon download."
    - `expirationDate` - "The date and time when the link enabling downloading part of the package expires. After this time, the link is no longer active."

- **Authorization**

    - Extended access rules with `SubunitManage` for POST `/permissions/query/persons/grants` : the operation can be performed if the entity has `CredentialsManage` , `CredentialsRead` , `SubunitManage` .
    - Granting permissions indirectly (POST `/permissions/indirect/grants` ) Updated description of `targetIdentifier.description` property: clarified that the absence of a context identifier means that a general indirect permission has been granted.

- **OpenAPI**
     Increased the maximum `pageSize` value from 100 to 500 for endpoints:

    - GET `/sessions`
    - GET `/sessions/{referenceNumber}/invoices`
    - GET `/sessions/{referenceNumber}/invoices/failed`

### Version 2.0.0 RC5.4

- **Downloading a list of invoice metadata (POST /invoices/query/metadata)**

    - Added `sortOrder` parameter to specify the sorting direction of the results.

- **Session status**
     Fixed a bug that prevented this property from being populated in invoice API responses (the field was previously not returned). The value is populated asynchronously at persistence and may be temporarily null.

- **Test data (only on test environments)**

    - Changing API limits for the current context (POST `testdata/rate-limits` )
         An endpoint has been added to allow temporary overrides of the effective API limits for the current context. This change prepares the limits simulator for launch in the TE environment.
    - Restoring default limits (DELETE `/testdata/rate-limits` ) Added an endpoint that restores default limit values for the current context.

- **OpenAPI**

    - Clarified array parameter definitions in query; `style: form` applied. Multiple values should be passed by repeating the parameter, e.g. `?statuses=InProgress&statuses=Succeeded` . Documentation change, no impact on API functionality.
    - Updated property descriptions ( `partUploadRequests` , `encryptedSymmetricKey` , `initializationVector` ).

### Version 2.0.0 RC5.3

- **Exporting a batch of invoices (POST `/invoices/exports` )**
     Added the ability to include the `_metadata.json` file in the export package. The file is a JSON object with an `invoices` array containing `InvoiceMetadata` objects (the model returned by POST `/invoices/query/metadata` ). Inclusion (preview): `X-KSeF-Feature` : `include-metadata` must be added to the request header. As of 2025-10-27, the default endpoint behavior is changing - the export package will always include the `_metadata.json` file (the header will not be required).

- **Invoice status**

    - In case of processing with an error, when it was possible to read the invoice number (e.g. error code `440` - duplicate invoice), the response contains `invoiceNumber` property with the read number.
    - Marked `invoiceHash` , `referenceNumber` property in response model as always returned.

- **Standardization of size units (MB, SI)**
     The limits in the documentation and API have been unified: values are presented in MB (SI), where 1 MB = 1,000,000 B.

- **Get limits for the current context (GET `/limits/context` )**
     Added `maxInvoiceSizeInMB` , `maxInvoiceWithAttachmentSizeInMB` for `onlineSession` and `batchSession` properties in the response model. `maxInvoiceSizeInMib` , `maxInvoiceWithAttachmentSizeInMib` properties marked as deprecated (planned removal: 2025-10-27).

- **Changing session limits for the current context (POST `/testdata/limits/context/session` )**
     Added `maxInvoiceSizeInMB` , `maxInvoiceWithAttachmentSizeInMB` for `onlineSession` and `batchSession` properties in the request model. `maxInvoiceSizeInMib` , `maxInvoiceWithAttachmentSizeInMib` properties marked as deprecated (planned removal: 2025-10-27).

- **Invoice batch export status (GET `/invoices/exports/{referenceNumber}` )**
     Renamed the path parameter from `operationReferenceNumber` to `referenceNumber` .
     The change does not affect the HTTP contract (path and value meaning remain unchanged) or the behavior of the endpoint.

- **Right**

    - Updated endpoint descriptions and endpoint examples from the permissions/* area. This change only affects the documentation (clarification of descriptions, formats, and examples); no changes to API behavior or the contract.
    - Renaming the path parameter from `operationReferenceNumber` to `referenceNumber` in "Getting operation status" (POST `/permissions/operations/{referenceNumber}` ).
         The change does not affect the HTTP contract (path and value meaning remain unchanged) or the behavior of the endpoint.
    - "POST `permissions/indirect/grants` "
         Added support for internal identifier - extended `targetIdentifier` property with the `InternalId` value.
    - "Get list of own permissions" (POST `/permissions/query/personal/grants` )
        - `targetIdentifier` property in the request model has been extended with the `InternalId` value (the ability to specify an internal identifier).
        - `PersonalPermissionScope.Owner` value has been removed from the response model. Ownership permissions (granted by ZAW-FA or NIP/PESEL link) are not returned.

- **Authentication Status (GET `/auth/{referenceNumber}` )**
     The error code table has been expanded to include `470` - "Authentication failed" with the clarification: "Attempt to use the authorization methods of a deceased person".

- **PEF invoice handling**
     Changed enum values ( `FormCode` ):

    - `FA_PEF (3)` to `PEF (3)` ,
    - `FA_KOR_PEF (3)` to `PEF_KOR (3)` .

- **Generating a new token (POST `/tokens` )**

    - In the request model ( `GenerateTokenRequest` ), `description` and `permissions` fields have been marked as required.
    - In the response model ( `GenerateTokenResponse` ), `referenceNumber` and `token` fields are marked as always returned.

- **KSeF token status (GET /tokens/{referenceNumber})**

    - Marked `authorIdentifier` , `contextIdentifier` , `dateCreated` , `description` , `referenceNumber` , `requestedPermissions` , `status` properties in the response model as always returned.

- **Downloading a list of generated tokens (GET /tokens)**

    - Marked `authorIdentifier` , `contextIdentifier` , `dateCreated` , `description` , `referenceNumber` , `requestedPermissions` , `status` properties in the response model as always returned.

- **Test data - create a natural person (POST `/testdata/person` )**
     Extended the request with `isDeceased` property (boolean) allowing the creation of a test deceased person (e.g. for scenarios verifying status code `470` ).

- **OpenAPI**

    - Clarified constraints for integer properties in requests by adding `minimum` / `exclusiveMinimum` , `maximum` / `exclusiveMaximum` attributes.
    - Extended the response with the `referenceNumber` field (it contains the same value as the existing `operationReferenceNumber` ). Marked `operationReferenceNumber` as `deprecated` and will be removed from the 2025-10-27 response; it should be switched to `referenceNumber` . Nature of the change: transient rename while maintaining compatibility (both properties returned in parallel until the deletion date).
         Applies to endpoints:
    - POST `/permissions/persons/grants` ,
    - POST `/permissions/entities/grants` ,
    - POST `/permissions/authorizations/grants` ,
    - POST `/permissions/indirect/grants` ,
    - POST `/permissions/subunits/grants` ,
    - POST `/permissions/eu-entities/administration/grants` ,
    - POST `/permissions/eu-entities/grants` ,
    - DELETE `/permissions/common/grants/{permissionId}` ,
    - DELETE `/permissions/authorizations/grants/{permissionId}` ,
    - POST `/invoices/exports` .
    - Removed the `required` attribute from `pageSize` property in the GET `/sessions` request ("Get a list of sessions").
    - Updated examples in endpoint definitions.

### Version 2.0.0 RC5.2

- **Right**

    - "Granting subunits administrator privileges" (POST `/permissions/subunits/grants` )
         Added `subunitName` property in the request. This field is required when the subunit is identified by an internal identifier.
    - "Getting the list of permissions of administrators of entities and subunits" (POST `/permissions/query/subunits/grants` )
         Added `subunitName` property in response.
    - "Downloading the list of permissions to work in KSeF granted to individuals or entities" (POST `permissions/query/persons/grants` )
         The `Owner` permission has been removed from the results. `Owner` permission is systemically assigned to an individual and cannot be granted, so it should not appear in the list of granted permissions.
    - "Getting a list of your own permissions" (POST `/permissions/query/personal/grants` )
         Extended the `PersonalPermissionType` filter enum with the `VatUeManage` value.

- **Limits**

    - Added endpoints to check set limits (context, authenticated entity):
        - GET `/limits/context`
        - GET `/limits/subject`
    - Added endpoints for limit management (context, authenticated entity) in the test environment:
        - POST/DELETE `/testdata/limits/context/session`
        - POST/DELETE `/testdata/limits/subject/certificate`
    - Updated [Limits](limity/limity.md) .

- **Invoice status**
     Added `invoicingMode` property to the response model. Updated documentation: [Automatically Determining Offline Shipping Mode](offline/automatyczne-okreslanie-trybu-offline.md) .

- **OpenAPI**

    - Clarified constraints for integer properties in requests by adding `minimum` / `exclusiveMinimum` , `maximum` / `exclusiveMaximum` attributes and `default` values.
    - Updated examples in endpoint definitions.
    - Endpoint descriptions have been clarified.
    - Added `required` attribute for required properties in requests and responses.

### Version 2.0.0 RC5.1

- **Get certificate metadata list (POST /certificates/query)**
     The representation of the subject identifier has been changed from `subjectIdentifier` + `subjectIdentifierType` property pair to `subjectIdentifier` { `type` , `value` } complex object.

- **Downloading a list of invoice metadata (POST /invoices/query/metadata)**

    - The representation of selected identifiers has been changed from type + value property pairs to complex objects { type, value }:
        - `invoiceMetadataBuyer.identifier` + `invoiceMetadataBuyer.identifierType` to the complex object `invoiceMetadataBuyerIdentifier` { `type` , `value` },
        - `invoiceMetadataThirdSubject.identifier` + `invoiceMetadataThirdSubject.identifierType` to composite object `InvoiceMetadataThirdSubjectIdentifier` { `type` , `value` }.
    - Removed `obsoleted` `Identitifer` properties from `InvoiceMetadataSeller` and `InvoiceMetadataAuthorizedSubject` objects.
    - Changed `invoiceQuerySeller` property to `sellerNip` in request filter.
    - Changed `invoiceQueryBuyer` property to `invoiceQueryBuyerIdentifier` with { `type` , `value` } properties in request filter.

- **Right**
     The representation of selected identifiers has been changed from type + value property pairs to complex objects { type, value }:

    - "Getting a list of your own permissions" (POST `/permissions/query/personal/grants` ):
        - `contextIdentifier` + `contextIdentifierType` -&gt; `contextIdentifier` { `type` , `value` },
        - `authorizedIdentifier` + `authorizedIdentifierType` -&gt; `authorizedIdentifier` { `type` , `value` },
        - `targetIdentifier` + `targetIdentifierType` -&gt; `targetIdentifier` { type, value }.
    - "Downloading the list of permissions to work in KSeF granted to individuals or entities" (POST `/permissions/query/persons/grants` ),
        - `contextIdentifier` + `contextIdentifierType` -&gt; `contextIdentifier` { `type` , `value` },
        - `authorizedIdentifier` + `authorizedIdentifierType` -&gt; `authorizedIdentifier` { `type` , `value` },
        - `targetIdentifier` + `targetIdentifierType` -&gt; `targetIdentifier` { `type` , `value` },
        - `authorIdentifier` + `authorIdentifierType` -&gt; `authorIdentifier` { `type` , `value` }.
    - "Getting the list of permissions of entity and subunit administrators" (POST `/permissions/query/subunits/grants` ):
        - `authorizedIdentifier` + `authorizedIdentifierType` -&gt; `authorizedIdentifier` { `type` , `value` },
        - `subunitIdentifier` + `subunitIdentifierType` -&gt; `subunitIdentifier` { `type` , `value` },
        - `authorIdentifier` + `authorIdentifierType` -&gt; `authorIdentifier` { `type` , `value` }.
    - "Getting entity role list" (POST `/permissions/query/entities/roles` ):
        - `parentEntityIdentifier` + `parentEntityIdentifierType` -&gt; `parentEntityIdentifier` { `type` , `value` }.
    - "Getting the list of subordinate entities" (POST `/permissions/query/subordinate-entities/roles` ):
        - `subordinateEntityIdentifier` + `subordinateEntityIdentifierType` -&gt; `subordinateEntityIdentifier` { `type` , `value` }.
    - "Getting the list of entity permissions for handling invoices" (POST `/permissions/query/authorizations/grants` ):
        - `authorizedEntityIdentifier` + `authorizedEntityIdentifierType` -&gt; `authorizedEntityIdentifier` { `type` , `value` },
        - `authorizingEntityIdentifier` + `authorizingEntityIdentifierType` -&gt; `authorizingEntityIdentifier` { `type` , `value` },
        - `authorIdentifier` + `authorIdentifierType` -&gt; `authorIdentifier` { `type` , `value` }
    - "Get the list of permissions of administrators or representatives of EU entities entitled to self-billing" (POST `/permissions/query/eu-entities/grants` ):
        - `authorIdentifier` + `authorIdentifierType` -&gt; `authorIdentifier` { `type` , `value` }

- **Granting administrator rights to an EU entity (POST permissions/eu-entities/administration/grants)**
     Changed the property name in the request from `subjectName` to `euEntityName` .

- **Authentication using the KSeF token**
     Removed redundant enum values `None` , `AllPartners` in `contextIdentifier.type` property of POST `/auth/ksef-token` request.

- **KSeF Tokens**

    - Disambiguated GET `/tokens` response model: `authorIdentifier.type` , `authorIdentifier.value` , `contextIdentifier.type` , `contextIdentifier.value` properties are always returned (required, non-nullable),
    - Removed redundant enum values `None` , `AllPartners` in `authorIdentifier.type` and `contextIdentifier.type` properties in the GET `/tokens` response model ("Get list of generated tokens").

- **Batch session**
     Removed redundant error code `21401` - "Document does not conform to schema (json)".

- **Get session status (GET /sessions/{referenceNumber})**

    - Added error code `420` - "Invoice limit exceeded in session".

- **Retrieving invoice metadata (GET `/invoices/query/metadata` )**

    - Added (always returned) `isTruncated` (boolean) property in the response - "Determines whether the result was truncated due to exceeding the invoice count limit (10,000)",
    - Marked `amount.type` property in the request filter as required.

- **Exporting a batch of invoices: order (POST `/invoices/exports` )**

    - Marked `operationReferenceNumber` property in the response model as always returned,
    - Marked `amount.type` property in the request filter as required.

- **Downloading the list of permissions to work in KSeF granted to individuals or entities (POST /permissions/query/persons/grants)**

    - Added `contextIdentifier` in request filter and response model.

- **OpenAPI**
     Removed unused `operationId` from specification. Cleanup change.

### Version 2.0.0 RC5

- **PEF invoice handling and Peppol service providers**

    - Support for `PEF` invoices sent by the Peppol service provider has been added. These new capabilities do not change the existing KSeF behavior for other formats; they are an API extension.
    - A new authentication context type has been introduced: `PeppolId` , allowing you to work in the context of the Peppol service provider.
    - Automatic supplier registration: when a Peppol service provider is authenticated for the first time (using a dedicated certificate), it is automatically registered in the system.
    - Added endpoint GET `/peppol/query` ("Peppol Service Provider List") returning registered providers.
    - Updated access rules for opening and closing sessions, sending invoices requires the `PefInvoiceWrite` permission.
    - New invoice templates added: `FA_PEF (3)` , `FA_KOR_PEF (3)` ,
    - Extended `ContextIdentifier` with `PeppolId` in `AuthTokenRequest` xsd.

- **UPO**

    - `Uwierzytelnienie` element has been added, which organizes the data from the UPO header and extends it with additional information; it replaces the existing `IdentyfikatorPodatkowyPodmiotu` and `SkrotZlozonejStruktury` .
    - `Uwierzytelnienie` includes:
        - `IdKontekstu` – authentication context identifier,
        - proof of authentication (depending on the method):
            - `NumerReferencyjnyTokenaKSeF` - the authentication token identifier in the KSeF system,
            - `SkrotDokumentuUwierzytelniajacego` - the hash function value of the authentication document in the form received by the system (including the electronic signature).
    - In the `Dokument` element, the following was added:
        - Seller's Tax Identification Number,
        - InvoiceIssueDate,
        - DateofAssignmentofKSeFNumber.
    - The UPO schema has been unified. UPO for invoices and sessions use a common schema, [upo-v4-2.xsd](/faktury/upo/schemy/upo-v4-2.xsd) . It replaces the previous upo-faktura-v3-0.xsd and upo-sesja-v4-1.xsd.

- **API request limits**
     Added [API request limits](limity/limity-api.md) specification.

- **Authentication**

    - Clarified status codes in GET `/auth/{referenceNumber}` , `/auth/sessions` :
        - `415` (no permissions),
        - `425` (authentication invalidated),
        - `450` (invalid token: invalid token, invalid time, invalidated, inactive),
        - `460` (Certificate error: invalid, chain verification error, untrusted chain, revoked, invalid).
    - Updated optional IP policy in `AuthTokenRequest` XSD: Replaced `IpAddressPolicy` with new `AuthorizationPolicy` / `AllowedIps` structure. Updated [Authentication](uwierzytelnianie.md) document.

- **Authorization**

    - Access rules have been extended with `VatUeManage` , `SubunitManage` for DELETE `/permissions/common/grants/{permissionId}` : the operation can be performed if the entity has `CredentialsManage` , `VatUeManage` or `SubunitManage` .
    - Extended access rules with `Introspection` for GET `/sessions/{referenceNumber}/...` : each of these endpoints can now be called with `InvoiceWrite` or `Introspection` .
    - Extended access rules for `InvoiceWrite` for GET `/sessions` ("Get list of sessions"): with the `InvoiceWrite` permission, you can only retrieve sessions created by the authenticator; with `Introspection` permission, you can retrieve all sessions.
    - Changed access rules for DELETE `/tokens/{referenceNumber}` : removed requirement `CredentialsManage` permission.

- **Downloading data for the certification application (GET `certificates/enrollments/data` )**

    - Changing the structure of the answer:
        - Removed: givenNames (string array).
        - Added: givenName (string).
        - Nature of change: breaking (changing the name and type of the field from array to text).
    - Added error code `25011` - "Invalid CSR signature algorithm".
    - The requirements for the private key used to sign the CSR in [KSeF Certificates](certyfikaty-KSeF.md) have been clarified.

- **KSeF Tokens**

    - Added error code for POST `/tokens` response ("Generating new token"): `26002` - "Unable to generate token for current context type." Token can only be generated in context of `Nip` or `InternalId` .
    - The catalog of permissions that can be assigned to a token has been expanded: `SubunitManage` and `EnforcementOperations` have been added.
    - Added query parameters to filter results for GET `/tokens` :
        - `description` - search in the token description (case-insensitive), min. 3 characters,
        - `authorIdentifier` - search by author ID (case-insensitive), min. 3 characters,
        - `authorIdentifierType` - the type of creator identifier used for authorIdentifier (Nip, Pesel, Fingerprint).
    - Property added
        - `lastUseDate` - "Date of last token use",
        - `statusDetails` - "Additional status information, returned in case of errors"
             in replies to:
    - GET `/tokens` ("tokens list"),
    - GET `/tokens/{referenceNumber}` ("token status").

- **Retrieving invoice metadata (GET `/invoices/query/metadata` )**

    - Filters:
        - paging: increased maximum page size to 250 records,
        - removed `schemaType` property (with values `FA1` , `FA2` , `FA3` ), previously marked as deprecated,
        - added `seller.nip` ; `seller.identifier` marked as deprecated (will be removed in the next release),
        - added `authorizedSubject.nip` ; `authorizedSubject.identifier` marked as deprecated (will be removed in the next release),
        - the description has been clarified: no value in `dateRange.to` means using the current date and time (UTC),
        - The maximum allowed `DateRange` has been clarified to 2 years.
    - Sorting:
        - the results are sorted ascending by the date type indicated in `DateRange` ; for incremental retrieval, the `PermanentStorage` type is recommended,
    - Model answer:
        - `totalCount` property removed,
        - `fileHash` renamed to `invoiceHash` ,
        - added `seller.nip` ; `seller.identifier` marked as deprecated (will be removed in the next release),
        - added `authorizedSubject.nip` ; `authorizedSubject.identifier` marked as deprecated (will be removed in the next release),
        - `invoiceHash` marked as always refunded,
        - `invoicingMode` marked as always returned,
        - `authorizedSubject.role` ("Authorized Subject") has been marked as always returned,
        - `invoiceMetadataAuthorizedSubject.role` ("Authorized entity's Tax Identification Number") has been marked as always returned,
        - marked `invoiceMetadataThirdSubject.role` ("Third Party List") as always returned.
    - Removed [Mock] tags from property descriptions.

- **Exporting a batch of invoices: order (POST `/invoices/exports` )**

    - Filters:

        - added `seller.nip` ; `seller.identifier` marked as deprecated (will be removed in the next release),

    - [Mock] markings removed.

    - Changed error code: from `21180` to `21181` ("Invalid invoice export request").

    - Sorting rules have been clarified. Invoices in a batch are sorted ascending by the date type specified in `DateRange` when initiating the export.

    - **Invoice batch export: status (GET `/invoices/exports/{operationReferenceNumber}` )**

        - Status descriptions: export status documentation has been updated:
            - `100` - "Invoice export in progress"
            - `200` - "Invoice export completed successfully"
            - `415` - "Error decrypting the supplied key"
            - `500` - "Unknown error ({statusCode})"
        - `package` response model:
            - added:
                - `invoiceCount` - "Total number of invoices in the batch. The maximum number of invoices in the batch is 10,000",
                - `size` - "The size of the package in bytes. The maximum package size is 1 GiB (1,073,741,824 bytes)",
                - `isTruncated` - "Determines whether the export result was truncated due to exceeding the invoice count or package size limit",
                - `lastIssueDate` - "Date of issuing the last invoice included in the batch.\nThe field appears only when the batch was cut and the export was filtered by the `Issue` date type",
                - `lastInvoicingDate` - "Date of receipt of the last invoice included in the batch.\nThe field appears only when the batch was cut and the export was filtered by `Invoicing` date type",
                - `lastPermanentStorageDate` - "Date of permanent storage of the last invoice included in the batch.\nThe field only appears when the batch was truncated and the export was filtered by the `PermanentStorage` date type."
        - `package.parts` response model
            - removed `fileName` , `headers` ,
            - added:
                - `partName` - "Package part file name",
                - `partSize` - "The size of the package part in bytes. The maximum part size is 50MiB (52,428,800 bytes)",
                - `partHash` - "SHA256 hash of the package part file, encoded in Base64 format",
                - `encryptedPartSize` - "Size of the encrypted part of the package in bytes",
                - `encryptedPartHash` - "SHA256 hash of the encrypted part of the package, encoded in Base64 format",
                - `expirationDate` - "The expiration date of the link to download the part",
            - all properties in `package` have been marked as always returned,
        - [Mock] markings removed.

- **Right**

    - Extended the POST `/permissions/eu-entities/administration/grants` request ("Granting EU entity administrator privileges") with the "SubjectName" `subjectName` .
    - Enhanced the `/permissions/query/persons/grants` POST request with a new `System` value for the `authorIdentifier` filter and removed the requirement from `authorIdentifier.value` field.
    - Enhanced the `/permissions/query/persons/grants` POST request with a new `AllPartners` value for `targetIdentifier` filter and removed the requirement from `targetIdentifier.value` field.
    - Added a POST request to `/permissions/query/personal/grants` to retrieve a list of your own permissions.
    - Added a new value `AllPartners` for the "target entity ID" to the `/permissions/indirect/grants` POST request, meaning general permissions

- **Downloading an invoice (GET `/invoices/ksef/{ksefNumber}` )**
     Added error code for response 400: `21165` - "The invoice with the given KSeF number is not yet available."

- **Invoice attachments**
     Added endpoint GET `/permissions/attachments/status` to check the status of consent to issue invoices with attachments.

- **Downloading the session list**
     Expanded permissions for GET `/sessions` : added `InvoiceWrite` . With the `InvoiceWrite` permission, you can only retrieve sessions created by the authenticator; with the `Introspection` permission, you can retrieve all sessions.

- **Interactive session**

    - Updated error codes for POST `/sessions/online/{referenceNumber}/invoices` ("Sending invoice"):
        - removed `21154` - "Interactive session ended",
        - added `21180` - "The session status does not allow the operation to be performed."
    - Added error `21180` - "The session status does not allow the operation" for POST `/sessions/online/{referenceNumber}/close` ("Closing interactive session").

- **Batch session**

    - Added error `21180` - "The session status does not allow the operation" for POST `/sessions/batch/{referenceNumber}/close` ("Closing the batch session").

- **Invoice status in session**
     The response for GET `/sessions/{referenceNumber}/invoices` ("Getting session invoices") and GET `/sessions/{referenceNumber}/invoices/{invoiceReferenceNumber}` ("Getting invoice status from session") has been extended with the following properties:

    - `permanentStorageDate` – date of permanent storage of the invoice in the KSeF repository (from this moment the invoice is available for download),
    - `upoDownloadUrl` – UPO download address.

- **OpenAPI**

    - Added universal input validation error code `21405` to all endpoints. Error content from the validator returned in the response.
    - Added a 400 validation response returning error code `30001` ("The subject or permission already exists.") for POST `/testdata/subject` and POST `/testdata/person` .
    - Updated examples in endpoint definitions.

- **Documentation**

    - Signature algorithms and examples have been clarified in [QR Codes](kody-qr.md) .
    - Updated C# and Java code examples.

### Version 2.0.0 RC4

- **KSeF certificates**

    - Added a new `type` property in KSeF certificates.
    - Available certificate types:
        - `Authentication` – certificate for authentication in the KSeF system,
        - `Offline` – a certificate limited solely to confirming the authenticity of the issuer and the integrity of the invoice in offline mode (KOD II).
    - Updated documentation for `/certificates/enrollments` , `/certificates/query` , `/certificates/retrieve` processes.

- **QR codes**

    - It has been clarified that CODE II can only be generated based on an `Offline` type certificate.
    - Added a security warning that `Authentication` certificates cannot be used to issue invoices offline.

- **Session status**

    - Authorization update - downloading information about sessions, invoices and UPO requires the permission: `InvoiceWrite` .
    - *Processing* status code changed: from `300` to `150` for batch session.

- **Retrieving invoice metadata ( `/invoices/query/metadata` )**
     The response model has been extended with the following fields:

    - `fileHash` – SHA256 hash of the invoice,
    - `hashOfCorrectedInvoice` – SHA256 hash of the corrected offline invoice,
    - `thirdSubjects` – list of third entities,
    - `authorizedSubject` – authorized entity (new `InvoiceMetadataAuthorizedSubject` object containing `identifier` , `name` , `role` ),

- Added the ability to filter by document type ( `InvoiceQueryFormType` ), available values: `FA` , `PEF` , `RR` .

- `schemaType` field marked as deprecated – planned to be removed in future API versions.

- **Documentation**

    - A document describing [the KSeF number](faktury/numer-ksef.md) has been added.
    - A document describing [technical correction](offline/korekta-techniczna.md) for invoices issued offline has been added.
    - The method [of detecting duplicates](faktury/weryfikacja-faktury.md) has been clarified

- **OpenAPI**

    - Downloading the list of invoice metadata
        - Added property: `hasMore` (boolean) – indicates when another page of results is available. `totalCount` property has been marked as deprecated (it remains in the response for now for backward compatibility).
        - When filtering by `dateRange` `to` (end date of the range) property is no longer mandatory.
    - Searching for granted permissions - in response, added `hasMore` property, removed `pageSize` , `pageOffset` .
    - Getting authentication status - removed redundant `referenceNumber` , `isCurrent` from the response.
    - Pagination unification - the `/sessions/{referenceNumber}/invoices` endpoint (retrieving session invoices) is now using pagination based on `x-continuation-token` request header; `pageOffset` parameter has been removed, and `pageSize` remains unchanged. The first page has no header; subsequent pages are retrieved by passing the token value returned by the API. This change is consistent with other resources using `x-continuation-token` (e.g., `/auth/sessions` , `/sessions/{referenceNumber}/invoices/failed` ).
    - Removed support for the `InternalId` in the `targetIdentifier` field when granting indirect permissions ( `/permissions/indirect/grants` ). Only the `Nip` Identifier is now allowed.
    - Status of the authorization granting operation – the list of possible status codes in the response has been extended:
        - 410 – The provided identifiers do not match or have an invalid relationship.
        - 420 – The credentials used do not have permission to perform this operation.
        - 430 – The identifier context does not match the required role or permissions.
        - 440 – Operation not allowed for the specified identifier associations.
        - 450 – Operation not allowed for the specified identifier or its type.
    - Added support for error **21418** – "The passed continuation token has an invalid format" on all endpoints using `continuationToken` pagination mechanism ( `/auth/sessions` , `/sessions` , `/sessions/{referenceNumber}/invoices` , `/sessions/{referenceNumber}/invoices/failed` , `/tokens` ).
    - The process of downloading a batch of invoices has been clarified:
        - `/invoices/exports` – starting the process of creating a batch of invoices,
        - `/invoices/async-query/{operationReferenceNumber}` – checking the status and receiving the finished package.
    - The `InvoiceMetadataQueryRequest` model has been renamed to `QueryInvoicesMetadataResponse` .
    - `PersonPermissionsAuthorIdentifier` type has been expanded with a new value, `System` (System Identifier). This value is used to identify permissions granted by KSeF based on a submitted ZAW-FA application. The change affects the endpoint: `/permissions/query/persons/grants` .

### Version 2.0.0 RC3

- **Adding an endpoint to download the list of invoice metadata**

    - `/invoices/query` (mock) replaced by `/invoices/query/metadata` – production endpoint for retrieving invoice metadata
    - Update related data models.

- **Update of the mock `invoices/async-query` endpoint to initialize the invoice download query**
     Updated related data models.

- **OpenAPI**

    - The endpoint specification has been supplemented with required permissions ( `x-required-permissions` ).
    - Added `403 Forbidden` and `401 Unauthorized` responses to the endpoint specification.
    - Added `required` attribute in permission query responses.
    - Updated description of the `/tokens` endpoint
    - Removed duplicate `enum` definitions
    - Unified the SessionInvoiceStatusResponse response model in `/sessions/{referenceNumber}/invoices` and `/sessions/{referenceNumber}/invoices/{invoiceReferenceNumber}` .
    - Added validation status 400: "Authentication failed | No permissions assigned."

- **Session status**

    - Added `Cancelled` status - "Canceling session. The timeout for sending was exceeded in the batch session, or no invoices were sent in the interactive session."
    - New error codes added:
        - 415 - "Unable to send invoice with attachment"
        - 440 - "Session canceled, sending time exceeded"
        - 445 - "Verification error, no valid invoices"

- **Invoice shipping status**

    - `AcquisitionDate` has been added - the date of assigning the KSeF number.
    - The `ReceiveDate` field has been replaced with `InvoicingDate` – the date of receipt of the invoice into the KSeF system.

- **Sending invoices in the session**

    - Added [validation of](faktury/weryfikacja-faktury.md#ograniczenia-ilo%C5%9Bciowe) zip package size (100 MB) and number of packages (50) in batch session
    - Added invoice count [validation](faktury/weryfikacja-faktury.md#ograniczenia-ilo%C5%9Bciowe) in interactive and batch sessions.
    - Changed the "Processing" status code from 300 to 150.

- **Authentication using XAdES signature**

    - Fixed ContextIdentifier in AuthTokenRequest xsd. Use the corrected version [of the XSD schema](https://ksef-test.mf.gov.pl/docs/v2/schemas/authv2.xsd) . [Prepare XML document](uwierzytelnianie.md#1-przygotowanie-dokumentu-xml-authtokenrequest)
    - Added error code `21117` - "Invalid entity identifier for the specified context type."

- **Removing the endpoint for anonymous invoice downloads `invoices/download`**
     The functionality of downloading invoices without authentication has been removed; it is only available in the KSeF web-based invoice verification and download tool.

- **Test data - handling invoices with attachments**
     New endpoints have been added to enable testing the sending of invoices with attachments.

- **KSeF Certificates - Validation of key type and length in CSR**

    - The description of the POST `/certificates/enrollments` endpoint has been supplemented with requirements for private key types in CSR (RSA, EC),
    - Added new error code 25010 in response 400: "Invalid key type or length."

- **Public certificate format update**
     `/security/public-key-certificates` – returns Base64-encoded DER format certificates.

### Version 2.0.0 RC2

- **New endpoints for managing authentication sessions**
     They allow you to view and invalidate active authentication sessions.
     [Managing authentication sessions](auth/sesje.md)

- **New endpoint for downloading the list of invoice sending sessions**
     `/sessions` – allows you to download metadata for sending sessions (interactive and batch), with the option to filter by status, closing date, and session type, among others.
     [Downloading the session list](faktury/sesja-sprawdzenie-stanu-i-pobranie-upo.md#1-pobranie-listy-sesji)

- **Change in permission listing**
     `/permissions/query/authorizations/grants` – added query type (queryType) in [subject permissions](uprawnienia.md#pobranie-listy-uprawnie%C5%84-podmiotowych-do-obs%C5%82ugi-faktur) filtering.

- **Support for the new version of the FA(3) invoice schema**
     When opening an interactive and batch session, it is possible to select the FA(3) schema.

- **Adding invoiceFileName field in batch session response**
     `/sessions/{referenceNumber}/invoices` – added invoiceFileName field containing the invoice file name. This field is only available for batch sessions. [Retrieving information about submitted invoices](faktury/sesja-sprawdzenie-stanu-i-pobranie-upo.md#3-pobranie-informacji-na-temat-przes%C5%82anych-faktur)
