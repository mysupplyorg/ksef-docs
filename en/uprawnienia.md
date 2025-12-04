## Right

10/07/2025

### Introduction – Business Context

The KSeF system introduces an advanced permissions management mechanism, which provides the foundation for secure and compliant use of the system by various entities. Permissions determine who can perform specific operations in KSeF – such as issuing invoices, viewing documents, granting further permissions, or managing subordinate units.

### Permission management goals:

- Data security – limiting access to information only to persons and entities that are formally authorized to do so.
- Compliance with regulations – ensuring that operations are performed by the appropriate entities in accordance with statutory requirements (e.g. VAT Act).
- Auditability – every operation related to granting or revoking permissions is recorded and can be analyzed.

### Who grants permissions?

Permissions can be granted by:

- entity owner - role (Owner),
- administrator of the subordinate entity,
- administrator of the subordinate unit,
- administrator of an EU entity,
- entity administrator, i.e. another entity or person with the CredentialsManage permission.

In practice, this means that every organization must manage the permissions of its employees, e.g. by granting permissions to an accounting employee when hiring a new employee or by revoking permissions when such an employee terminates their employment.

### When are permissions granted?

#### Examples:

- when starting cooperation with a new employee,
- if a company enters into cooperation with, for example, an accounting office, it should grant permissions to read invoices to that accounting office so that the office can settle the company's invoices,
- in connection with changes in the relationships between entities.

### Structure of granted permissions:

Permissions are granted:

1. Natural persons identified by PESEL, NIP or certificate fingerprint – to work at KSeF:

    - in the context of the entity granting the authorization (authorizations granted directly) or
    - in the context of another entity or entities:
        - in the context of a subordinate entity identified by the Tax Identification Number (a subordinate local government unit or a member of a VAT group),
        - in the context of a subordinate unit identified by an internal identifier,
        - in the context of a complex EU VAT number linking a Polish entity with an EU entity authorized to self-invoice on behalf of that Polish entity,
        - in the context of the indicated entity identified by the Tax Identification Number – the client of the entity granting the authorizations (selective authorizations granted indirectly),
        - in the context of all entities – clients of the entity granting the authorizations (general authorizations granted indirectly).

2. Other entities – identified by the Tax Identification Number:

    - as final recipients of the right to issue or view invoices,
    - as intermediaries - with the option to allow further delegation of authorizations enabled, so that the authorized entity can grant authorizations indirectly (see points IV and V above).

3. Other entities to act in their context on behalf of the authorizing entity (subjective rights):

    - tax representatives,
    - entities authorized to self-bille,
    - entities authorized to issue VAT RR invoices.

Access to system functions depends on the context in which authentication occurred and on the scope of permissions granted to the authenticated entity/person in that context.

## Glossary of terms (within the scope of KSeF's powers)

Deadline | Definition
--- | ---
**Powers** | Permission to perform specific operations in KSeF, e.g. `InvoiceWrite` , `CredentialsManage` .
**Owner** | Entity owner – a person who has full access to operations by default in the context of an entity with the same NIP identifier as the one stored in the authentication means used; the NIP-PESEL link also applies to the owner, so he or she can also authenticate using a means containing the associated PESEL number while retaining all the owner's privileges.
**Subordinate entity administrator** | A person with `CredentialsManage` permissions in the context of a subordinate entity. They can grant permissions (e.g., `InvoiceWrite` ). A subordinate entity can be, for example, a member of a VAT group.
**Subordinate Unit Administrator** | A person with `CredentialsManage` permissions in a subordinate entity. They can grant permissions (e.g., `InvoiceWrite` ).
**Administrator of an EU entity** | A person with `CredentialsManage` permissions in a complex context identified by NipVatUe. They can grant permissions (e.g., `InvoiceRead` ).
**Intermediary entity** | An entity that has been granted a permission with `canDelegate = true` flag and can delegate this permission, i.e., grant the permission indirectly. These permissions can only be `InvoiceWrite` and `InvoiceRead` .
**Target entity** | The entity in whose context a given right applies – e.g. a company served by an accounting office.
**Given directly** | A permission granted directly to a given user or entity by the owner or administrator.
**Indirectly transmitted** | Authorization granted by an intermediary to handle another entity – only for `InvoiceRead` and `InvoiceWrite` .
**`canDelegate`** | A technical flag ( `true` / `false` ) that allows for the delegation of authorizations. Only `InvoiceRead` and `InvoiceWrite` can have `canDelegate = true` . It can only be used when granting authorization to an entity to process invoices.
**`subjectIdentifier`** | Data identifying the recipient of the rights (person or entity): `Nip` , `Pesel` , `Fingerprint` .
**`targetIdentifier` / `contextIdentifier`** | Data identifying the context in which the granted authorization operates – e.g. customer's Tax Identification Number, Internal Identifier of the organizational unit.
**Fingerprint** | The result of calculating the SHA-256 hash function on a qualified certificate. This allows the recognition of a certificate from an entity holding the authorization granted based on the certificate's fingerprint. It is used, among other things, to identify foreign individuals or entities.
**InternalId** | Internal identifier of the subordinate unit in the KSeF system - a two-part identifier consisting of the NIP number and five digits `nip-5_cyfr` .
**NipVatUe** | A composite identifier, i.e. a two-part identifier consisting of the NIP number of the Polish entity and the EU VAT number of the EU entity, which are separated by the `nip-vat_ue` separator.
**Receiving** | The operation of revoking a previously granted permission.
**`permissionId`** | Technical identifier of the granted authorization – required, among others, for withdrawal operations.
**`operationReferenceNumber`** | Operation identifier (e.g. granting or revoking permissions), returned by the API, used to check status.
**Operation status** | Current status of the permission granting/revoking process: `100` , `200` , `400` , etc.

## Roles and permissions model (permission matrix)

The KSeF system allows for precise assignment of permissions, taking into account the types of activities performed by users. Permissions can be granted both directly and indirectly, depending on the access delegation mechanism.

### Examples of roles to map with permissions:

Role / entity | Role Description | Possible permissions
--- | --- | ---
**Owner of the entity** | This role is automatically assigned to the owner by default. To be recognized by the system as the owner, you must authenticate with a vector with the same Tax Identification Number (NIP) as the login context's Tax Identification Number (NIP) or a related PESEL number. | `Owner` role including all invoicing and administrative rights except `VatUeManage` .
**Entity Administrator** | A natural person who has the right to grant and revoke permissions to other users and/or appoint administrators of subordinate units/entities. | `CredentialsManage` , `SubunitManage` , `Introspection` .
**Operator (Accounting/Invoicing)** | The person responsible for issuing or reviewing invoices. | `InvoiceWrite` , `InvoiceRead` .
**Authorized entity** | Another business entity that has been granted specific rights to issue invoices on behalf of the entity, e.g., a tax representative. | `SelfInvoicing` , `RRInvoicing` , `TaxRepresentative`
**Intermediary entity** | An entity that has received permissions with the delegation option ( `canDelegate` ) and can grant them further. | `InvoiceRead` , `InvoiceWrite` with `canDelegate = true` flag.
**Administrator of an EU entity** | A person identifying themselves with a certificate who has the right to grant and revoke authorizations to other users within an EU entity associated with a given Polish entity. | `InvoiceWrite` , `InvoiceRead` , `VatUeManage` , `Introspection` .
**Representative of an EU entity** | A person identifying themselves with a certificate acting on behalf of an EU entity related to a given Polish entity. | `InvoiceWrite` , `InvoiceRead` .
**Subordinate Unit Administrator** | A user who has the ability to appoint administrators in subordinate units or entities. | `CredentialsManage` .

---

### Classification of permissions by type:

Permission type | Sample values | Possibility of sending indirectly | Operational description
--- | --- | --- | ---
**Invoice** | `InvoiceWrite` , `InvoiceRead` | ✔️ (if `canDelegate=true` ) | Invoice operations: sending, downloading
**Administrative** | `CredentialsManage` , `SubunitManage` , `VatUeManage` . | ❌ | Managing permissions and subordinate units
**Subjective** | `SelfInvoicing` , `RRInvoicing` , `TaxRepresentative` | ❌ | Authorizing other entities to act (issue invoices) in their own context on behalf of the authorizing entity
**Technical** | `Introspection` | ❌ | Access to operation and session history

---

## General and selective powers

The KSeF system allows you to grant selected permissions in a **general (general)** or **selective (individual)** manner, which allows you to flexibly manage access to the data of many business partners.

### Selective (individual) rights

Selective permissions are those granted by an intermediary entity (e.g., an accounting firm) with respect to **a specific target entity (partner)** . They allow access to be limited to a selected client or organizational unit.

**Example:**
 Accounting firm XYZ has been granted `InvoiceRead` permission by company ABC with the `canDelegate = true` flag. They can now delegate this permission to their employee, but only within the context of company ABC – other companies served by XYZ are not covered by this access.

**Selectivity features:**

- It is necessary to indicate `targetIdentifier` (e.g. partner's `Nip` ).
- The recipient of the permission acts only in the context of the indicated entity.
- It does not provide access to the data of other partners of the intermediary entity.

---

### General powers

General permissions are those that are granted without specifying a specific partner, which means that the recipient gains access to operations in the context **of all entities whose data are processed by the intermediary entity** .

**Example:** Entity A has `InvoiceRead` permission with `canDelegate = true` for multiple customers. This grants employee B the general `InvoiceRead` permission – B can now act on behalf of any of A's customers (e.g., view invoices for all contractors).

**Features of generality:**

- `targetIdentifier` type is `AllPartners` .
- Access covers all entities served by the intermediary.
- Used for mass integration, large shared service centers or accounting systems.

---

### Technical Notes and Limitations

- The mechanism only applies to `InvoiceRead` and `InvoiceWrite` permissions granted indirectly.
- In practice, the difference is the presence (selective) or absence (general) of the `targetIdentifier` entity in the body of `POST /permissions/indirect/grants` request.
- The system does not allow combining general and selective broadcasts in one call – separate operations must be performed.
- General permissions should be used with caution, especially in production environments, due to their broad scope.

---

### Permission assignment structure:

1. **Direct assignment** – e.g. the administrator of entity A assigns `InvoiceWrite` permission to a user in the context of entity A.
2. **Granting with the possibility of further delegation** – e.g. the administrator of entity A grants entity B (intermediary) `InvoiceRead` permission with `canDelegate=true` , which allows the administrator of entity B to grant `InvoiceRead` to entity/person C.
3. **Granting indirectly** – using a dedicated endpoint /permissions/indirect/grants, where the administrator of intermediary entity B, who has received delegated permissions from entity A, grants permissions on behalf of the entity/person C to handle the target entity A.

---

### Example of a permission matrix:

User / Entity | InvoiceWrite | InvoiceRead | CredentialsManage | SubunitManage | TaxRepresentative
--- | --- | --- | --- | --- | ---
Anna Kowalska (PESEL) | ✅ | ✅ | ❌ | ❌ | ❌
XYZ Accounting Office (NIP) | ✅ (with delegation) | ✅ (with delegation) | ❌ | ❌ | ❌
Jan Nowak (Certificate Identifier) | ✅ | ✅ | ❌ | ❌ | ❌
Accounting Department Admin (PESEL) | ❌ | ❌ | ✅ | ✅ | ❌
Mother Company, i.e. owner (NIP) | ✅ | ✅ | ✅ | ✅ | ✅
VAT group admin (PESEL) | ❌ | ❌ | ❌ | ✅ | ❌
Tax representative (NIP) | ❌ | ❌ | ❌ | ❌ | ✅

---

### Roles or permissions required to grant permissions

Granting permissions: | Required role or permission
--- | ---
a natural person to work at KSeF | `Owner` or `CredentialsManage`
entity for handling invoices | `Owner` or `CredentialsManage`
subjective | `Owner` or `CredentialsManage`
to process invoices – indirectly | `Owner` or `CredentialsManage`
to the administrator of the subordinate unit | `SubunitManage`
the administrator of an EU entity | `Owner` or `CredentialsManage`
representative of an EU entity | `VatUeManage`

---

### Identifier constraints ( `subjectIdentifier` , `contextIdentifier` )

Identifier type | Identified | Comments
--- | --- | ---
`Nip` | National entity | For entities registered in Poland and individuals
`Pesel` | Natural person | Required, among others, when granting authorizations to employees using a trusted profile or a qualified certificate with a PESEL number
`Fingerprint` | Certificate owner | Used when the qualified certificate does not contain the Tax Identification Number (NIP) or Personal Identification Number (PESEL) and when identifying administrators or representatives of EU entities
`NipVatUe` | EU entities related to Polish entities | Required when granting permissions to administrators and representatives of EU entities
`InternalId` | Subordinate units | Used in entities with a structure composed of subordinate units

---

### API functional limitations

- You cannot grant the same permission twice – the API may return an error or ignore the duplicate.
- Performing the operation of granting permissions does not result in immediate access – the operation is asynchronous and must be correctly processed by the system (the status of the operation must be checked).

---

### Time constraints

- The granted permissions remain active until they are revoked.
- Implementing a time restriction requires logic on the client system side (e.g., a schedule for revoking the permission).

## Granting permissions

### Granting authorizations to individuals to work at KSeF.

Organizations using KSeF can grant permissions to specific individuals—for example, accounting or IT staff. Permissions are assigned to individuals based on their identifier (PESEL, NIP, or fingerprint). Permissions can cover both operational activities (e.g., invoicing) and administrative tasks (e.g., permission management). This section describes how to grant such permissions using the API and the permission requirements for the granter.

POST [/permissions/persons/grants](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Nadawanie-uprawnien/paths/~1api~1v2~1permissions~1persons~1grants/post)

Field | Value
:-- | :--
`subjectIdentifier` | Identifier of an entity or individual. `"Nip"` , `"Pesel"` , `"Fingerprint"`
`permissions` | Permissions to grant. `"CredentialsManage"` , `"CredentialsRead"` , `"InvoiceWrite"` , `"InvoiceRead"` , `"Introspection"` , `"SubunitManage"` , `"EnforcementOperations"`
`description` | Text value (description)

List of rights that can be granted to an individual:

Powers | Description
:-- | :--
`CredentialsManage` | Permission management
`CredentialsRead` | Viewing permissions
`InvoiceWrite` | Issuing invoices
`InvoiceRead` | Viewing invoices
`Introspection` | Viewing session history
`SubunitManage` | Subordinate unit management
`EnforcementOperations` | Carrying out enforcement operations

C# example: [KSeF.Client.Tests.Core\E2E\Permissions\PersonPermission\PersonPermissionE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/Permissions/PersonPermission/PersonPermissionE2ETests.cs)

```csharp
GrantPermissionsPersonRequest request = GrantPersonPermissionsRequestBuilder
    .Create()
    .WithSubject(subject)
    .WithPermissions(
        StandardPermissionType.InvoiceRead,
        StandardPermissionType.InvoiceWrite)
    .WithDescription(description)
    .Build();

OperationResponse response =
    await KsefClient.GrantsPermissionPersonAsync(request, accessToken, CancellationToken);
```

Java example: [PersonPermissionIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/PersonPermissionIntegrationTest.java)

```java

GrantPersonPermissionsRequest request = new GrantPersonPermissionsRequestBuilder()
        .withSubjectIdentifier(new PersonPermissionsSubjectIdentifier(PersonPermissionsSubjectIdentifier.IdentifierType.PESEL, personValue))
        .withPermissions(List.of(PersonPermissionType.INVOICEWRITE, PersonPermissionType.INVOICEREAD))
        .withDescription("e2e test grant")
        .build();

OperationResponse response = ksefClient.grantsPermissionPerson(request, accessToken);
```

Permissions can be granted by someone who is:

- owner
- has `CredentialsManage` permission
- administrator of subunits who has `SubunitManage`
- the administrator of an EU entity that has `VatUeManage`

---

### Giving entities the right to process invoices

KSeF allows you to grant permissions to entities that will process invoices on behalf of a given organization—e.g., accounting offices, shared service centers, or outsourcing companies. InvoiceRead and InvoiceWrite permissions can be granted directly, and if necessary, with the option of further delegation (the `canDelegate` flag). This section discusses the mechanism for granting these permissions, the required roles, and sample implementations.

POST [/permissions/entities/grants](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Nadawanie-uprawnien/paths/~1api~1v2~1permissions~1entities~1grants/post)

- **InvoiceWrite** : This permission allows you to upload XML invoice files to the KSeF system. Once successfully verified and assigned a KSeF number, these files become structured invoices.
- **InvoiceRead** : With this permission, the entity can download lists of invoices within a given context, download invoice content, invoices, report abuse, as well as generate and view bulk payment identifiers.
- **The InvoiceWrite** and **InvoiceRead** permissions can be granted directly to entities by the granting entity. An API client granting these permissions directly must have **the CredentialsManage** permission or the **Owner** role. When granting permissions to entities, it is possible to set the `canDelegate` flag to `true` for **InvoiceRead** and **InvoiceWrite** , which allows for further, indirect delegation of this permission.

Field | Value
:-- | :--
`subjectIdentifier` | Entity identifier. `"Nip"`
`permissions` | Permissions to send. `"InvoiceWrite"` , `"InvoiceRead"`
`description` | Text value (description)

C# example: [KSeF.Client.Tests.Core\E2E\Permissions\EntityPermission\EntityPermissionE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/Permissions/EntityPermission/EntityPermissionE2ETests.cs)

```csharp
GrantPermissionsEntityRequest request = GrantEntityPermissionsRequestBuilder
    .Create()
    .WithSubject(subject)
    .WithPermissions(
        Permission.New(StandardPermissionType.InvoiceRead, true),
        Permission.New(StandardPermissionType.InvoiceWrite, false)
    )
    .WithDescription(description)
    .Build();

OperationResponse response = await KsefClient.GrantsPermissionEntityAsync(request, accessToken, CancellationToken);
```

Java example: [EntityPermissionIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/EntityPermissionIntegrationTest.java)

```java
GrantEntityPermissionsRequest request = new GrantEntityPermissionsRequestBuilder()
        .withPermissions(List.of(
                new EntityPermission(EntityPermissionType.INVOICE_READ, true),
                new EntityPermission(EntityPermissionType.INVOICE_WRITE, false)))
        .withDescription(DESCRIPTION)
        .withSubjectIdentifier(new SubjectIdentifier(SubjectIdentifier.IdentifierType.NIP, targetNip))
        .build();

OperationResponse response = ksefClient.grantsPermissionEntity(request, accessToken);
```

---

### Granting subjective rights

For selected invoicing processes, KSeF provides so-called entity permissions, which apply to invoicing on behalf of another entity ( `TaxRepresentative` , `SelfInvoicing` , `RRInvoicing` ). These permissions can only be granted by the owner or administrator with `CredentialsManage` . This section describes how they are granted, their use, and their technical limitations.

POST [/permissions/authorizations/grants](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Nadawanie-uprawnien/paths/~1api~1v2~1permissions~1authorizations~1grants/post)

It is used to grant so-called subject authorizations, such as `SelfInvoicing` (self-invoicing), `RRInvoicing` (RR self-invoicing) or `TaxRepresentative` (tax representative operations).

Nature of rights:

These are entity-specific permissions, meaning they are important when the entity sends invoice files and are verified during the validation process. The relationship between the entity and the invoice data is verified. They can be changed during a session.

Required permissions to grant permissions: `CredentialsManage` or `Owner` .

Field | Value
:-- | :--
`subjectIdentifier` | Entity identifier. `"Nip"`
`permissions` | Authorizations to assign. `"SelfInvoicing"` , `"RRInvoicing"` , `"TaxRepresentative"`
`description` | Text value (description)

C# example: [KSeF.Client.Tests.Core\E2E\Permissions\ProxyPermission\AuthorizationPermissionsE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/Permissions/ProxyPermission/AuthorizationPermissionsE2ETests.cs)

```csharp
GrantAuthorizationPermissionsRequest grantPermissionAuthorizationRequest =
    GrantAuthorizationPermissionsRequestBuilder
    .Create()
    .WithSubject(Fixture.SubjectIdentifier)
    .WithPermission(StandardPermissionType.SelfInvoicing)
    .WithDescription("E2E test grant")
    .Build();

OperationResponse operationResponse = await KsefClient
    .GrantsAuthorizationPermissionAsync(grantPermissionAuthorizationRequest,
    accessToken, CancellationToken);
```

Java example: [ProxyPermissionIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/ProxyPermissionIntegrationTest.java)

```java
GrantAuthorizationPermissionsRequest request = new GrantAuthorizationPermissionsRequestBuilder()
        .withSubjectIdentifier(new SubjectIdentifier(SubjectIdentifier.IdentifierType.NIP, subjectNip))
        .withPermission(InvoicePermissionType.SELF_INVOICING)
        .withDescription("e2e test grant")
        .build();

OperationResponse response = ksefClient.grantsPermissionsProxyEntity(request, accessToken);
```

---

### Granting permissions indirectly

The indirect authorization mechanism enables the operation of a so-called intermediary entity, which – based on previously obtained delegations – can transfer selected authorizations further, within the context of another entity. This most commonly applies to accounting firms that serve multiple clients. This section describes the conditions that must be met to use this functionality and presents the data structure required to perform such assignment.

`InvoiceWrite` and `InvoiceRead` permissions are the only permissions that can be granted indirectly. This means that an intermediary entity can grant these permissions to another entity (the authorized party), which will apply to the target entity (partner). These permissions can be selective (for a specific partner) or general (for all partners of the intermediary entity). For selective granting, the target entity identifier must be assigned with the type `"Nip"` and the value of the specific TAXpayer Identification Number. For general granting, the target entity identifier must be assigned with the type `"AllPartners"` without the `value` field.

POST [/permissions/indirect/grants](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Nadawanie-uprawnien/paths/~1api~1v2~1permissions~1indirect~1grants/post)

Field | Value
:-- | :--
`subjectIdentifier` | Individual identifier. `"Nip"` , `"Pesel"` , `"Fingerprint"`
`targetIdentifier` | Target entity identifier. `"Nip"` or `null`
`permissions` | Permissions to send. `"InvoiceRead"` , `"InvoiceWrite"`
`description` | Text value (description)

C# example: [KSeF.Client.Tests.Core\E2E\Permissions\IndirectPermission\IndirectPermissionE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/Permissions/IndirectPermission/IndirectPermissionE2ETests.cs)

```csharp
GrantPermissionsIndirectEntityRequest request = GrantIndirectEntityPermissionsRequestBuilder
    .Create()
    .WithSubject(subject)
    .WithContext(new TargetIdentifier { Type = TargetIdentifierType.Nip, Value = targetNip })
    .WithPermissions(StandardPermissionType.InvoiceRead, StandardPermissionType.InvoiceWrite)
    .WithDescription(description)
    .Build();

OperationResponse grantOperationResponse = await KsefClient.GrantsPermissionIndirectEntityAsync(request, accessToken, CancellationToken);
```

Java example: [IndirectPermissionIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/IndirectPermissionIntegrationTest.java)

```java
GrantIndirectEntityPermissionsRequest request = new GrantIndirectEntityPermissionsRequestBuilder()
        .withSubjectIdentifier(new SubjectIdentifier(SubjectIdentifier.IdentifierType.NIP, subjectNip))
        .withTargetIdentifier(new TargetIdentifier(TargetIdentifier.IdentifierType.NIP, targetNip))
        .withPermissions(List.of(INVOICE_WRITE))
        .withDescription("E2E indirect grantE2E indirect grant")
        .build();

OperationResponse response = ksefClient.grantsPermissionIndirectEntity(request, accessToken);

```

---

### Granting administrator privileges to a subordinate entity

An entity's organizational structure may include subordinate units or entities—e.g., branches, departments, subsidiaries, VAT group members, and local government units. KSeF allows you to assign management privileges for such units. `SubunitManage` privilege is required. This section describes how to assign administrative privileges within the context of a subordinate unit or entity, taking into account the `InternalId` or `Nip` .

POST [/permissions/subunits/grants](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Nadawanie-uprawnien/paths/~1api~1v2~1permissions~1subunits~1grants/post)

Required broadcasting permissions:

- The user who wants to grant these permissions must have the `SubunitManage` permission.

Field | Value
:-- | :--
`subjectIdentifier` | Identifier of a natural person or entity. `"Nip"` , `"Pesel"` , `"Fingerprint"`
`contextIdentifier` | Subordinate entity identifier. `"Nip"` , `InternalId`
`description` | Text value (description)

C# example: [KSeF.Client.Tests.Core\E2E\Permissions\SubunitPermission\SubunitPermissionsE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/Permissions/SubunitPermission/SubunitPermissionsE2ETests.cs)

```csharp
GrantPermissionsSubUnitRequest grantPermissionsSubUnitRequest =
    GrantSubUnitPermissionsRequestBuilder
    .Create()
    .WithSubject(Fixture.SubUnit)
    .WithContext(Fixture.Unit)
    .WithDescription("E2E test grant sub-unit")
    .Build();

OperationResponse operationResponse = await KsefClient
    .GrantsPermissionSubUnitAsync(grantPermissionsSubUnitRequest, accessToken, CancellationToken);
```

Java example:

[SubUnitPermissionIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/SubUnitPermissionIntegrationTest.java)

```java
SubunitPermissionsGrantRequest request = new SubunitPermissionsGrantRequestBuilder()
        .withSubjectIdentifier(new SubjectIdentifier(SubjectIdentifier.IdentifierType.NIP, subjectNip))
        .withContextIdentifier(new ContextIdentifier(ContextIdentifier.IdentifierType.INTERNALID, contextNip))
        .withDescription("e2e subunit test")
        .withSubunitName("test")
        .build();

OperationResponse response = ksefClient.grantsPermissionSubUnit(request, accessToken);

```

---

### Granting administrator rights to an EU entity

Granting administrator privileges to an EU entity in KSeF allows the entity or person designated by the EU entity with the right to self-billing to be authorized on behalf of the Polish entity granting the authorization. This operation allows the authorized person to log in to the `NipVatUe` complex context, which binds the Polish entity granting the authorization to the EU entity with the right to self-billing. After granting administrator privileges to an EU entity, such person will be able to perform invoice operations and manage the authorizations of other individuals (so-called representatives of the EU entity) within this complex context.

POST [/permissions/eu-entities/administration/grants](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Nadawanie-uprawnien/paths/~1api~1v2~1permissions~1eu-entities~1administration~1grants/post)

Field | Value
:-- | :--
`subjectIdentifier` | Identifier of a natural person or entity. `"Nip"` , `"Pesel"` , `"Fingerprint"`
`contextIdentifier` | A two-part identifier consisting of the Tax Identification Number (NIP) and the VAT-EU number `{nip}-{vat_ue}` . `"NipVatUe"`
`description` | Text value (description)

C# example: [KSeF.Client.Tests.Core\E2E\Permissions\EuEntityPermission\EuEntityPermissionE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/Permissions/EuEntityPermission/EuEntityPermissionE2ETests.cs)

```csharp
GrantPermissionsRequest grantPermissionsRequest = GrantEUEntityPermissionsRequestBuilder
    .Create()
    .WithSubject(TestFixture.EuEntity)
    .WithSubjectName(EuEntitySubjectName)
    .WithContext(contextIdentifier)
    .WithDescription(EuEntityDescription)
    .Build();

OperationResponse operationResponse = await KsefClient
    .GrantsPermissionEUEntityAsync(grantPermissionsRequest, accessToken, CancellationToken);
```

Java example: [EuEntityPermissionIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/EuEntityPermissionIntegrationTest.java)

```java
EuEntityPermissionsGrantRequest request = new GrantEUEntityPermissionsRequestBuilder()
        .withSubject(new SubjectIdentifier(SubjectIdentifier.IdentifierType.FINGERPRINT, euEntity))
        .withEuEntityName("Sample Subject Name")
        .withContext(new ContextIdentifier(ContextIdentifier.IdentifierType.NIP_VAT_UE, nipVatUe))
        .withDescription("E2E EU Entity Permission Test")
        .build();

OperationResponse response = ksefClient.grantsPermissionEUEntity(request, accessToken);

```

---

### Granting the rights of a representative of an EU entity

An EU entity representative is a person acting on behalf of an entity registered in the EU who requires access to the KSeF to view or issue invoices. This authorization can only be granted by the EU VAT administrator. This section presents the data structure and how to invoke the appropriate endpoint.

POST [/permissions/eu-entities/grants](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Nadawanie-uprawnien/paths/~1api~1v2~1permissions~1eu-entities~1grants/post)

Field | Value
:-- | :--
`subjectIdentifier` | Entity identifier. `"Fingerprint"`
`permissions` | Permissions to send. `"InvoiceRead"` , `"InvoiceWrite"`
`description` | Text value (description)

C# example: [KSeF.Client.Tests.Core\E2E\Permissions\EuAdministrationPermission\EuRepresentativePermissionE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/Permissions/EuAdministrationPermission/EuRepresentativePermissionE2ETests.cs)

```csharp
GrantPermissionsEUEntityRepresentativeRequest grantRepresentativePermissionsRequest = GrantEUEntityRepresentativePermissionsRequestBuilder
    .Create()
    .WithSubject(new Client.Core.Models.Permissions.EUEntityRepresentative.SubjectIdentifier
    {
        Type = Client.Core.Models.Permissions.EUEntityRepresentative.SubjectIdentifierType.Fingerprint,
        Value = euRepresentativeEntityCertificateFingerprint
    })
    .WithPermissions(
        StandardPermissionType.InvoiceWrite,
        StandardPermissionType.InvoiceRead
        )
    .WithDescription("Representative for EU Entity")
    .Build();

OperationResponse grantRepresentativePermissionResponse = await KsefClient.GrantsPermissionEUEntityRepresentativeAsync(grantRepresentativePermissionsRequest,
    euAuthInfo.AccessToken.Token,
    CancellationToken.None);
```

Java example: [EuEntityRepresentativePermissionIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/EuEntityRepresentativePermissionIntegrationTest.java)

```java
GrantEUEntityRepresentativePermissionsRequest request = new GrantEUEntityRepresentativePermissionsRequestBuilder()
        .withSubjectIdentifier(new SubjectIdentifier(SubjectIdentifier.IdentifierType.FINGERPRINT, fingerprint))
        .withPermissions(List.of(EuEntityPermissionType.INVOICE_WRITE, EuEntityPermissionType.INVOICE_READ))
        .withDescription("Representative for EU Entity")
        .build();

OperationResponse response = ksefClient.grantsPermissionEUEntityRepresentative(request, accessToken);


```

## Revoking permissions

The process of revoking permissions in KSeF is just as important as granting them – it ensures access control and enables rapid response to situations such as a change in employee role, termination of cooperation with an external partner, or a breach of security policies. Revoking permissions can be performed for any recipient category: individual, entity, subordinate entity, EU representative, or EU administrator. This section discusses the methods for revoking different types of permissions and the required identifiers.

### Revocation of privileges

A standard method for revoking permissions that can be used for most cases: individuals, national entities, subordinate entities, as well as EU representatives or EU administrators. This operation requires knowledge `permissionId` and possession of the appropriate permission.

DELETE [/permissions/common/grants/{permissionId}](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Odbieranie-uprawnien/paths/~1api~1v2~1permissions~1common~1grants~1%7BpermissionId%7D/delete)

This method is used to revoke permissions such as:

- granted to natural persons to work at KSeF,
- assigned to entities for handling invoices,
- given indirectly,
- administrator of the subordinate entity,
- administrator of an EU entity,
- representative of an EU entity.

Example in C#:

```csharp
OperationResponse operationResponse = await KsefClient.RevokeCommonPermissionAsync(permission.Id, accessToken, CancellationToken);
```

Java example: [EntityPermissionIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/EntityPermissionIntegrationTest.java)

```java
OperationResponse response = ksefClient.revokeCommonPermission(permissionId, accessToken);
```

---

### Revocation of subjective rights

For entity-type permissions ( `SelfInvoicing` , `RRInvoicing` , `TaxRepresentative` ), a separate method of revocation applies – using an endpoint dedicated to authorization operations. These permissions are not transferable, so their revocation has immediate effect and ends the ability to perform invoice operations in that mode. Knowledge `permissionId` is required.

DELETE [/permissions/authorizations/grants/{permissionId}](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Odbieranie-uprawnien/paths/~1api~1v2~1permissions~1authorizations~1grants~1%7BpermissionId%7D/delete)

This method is used to revoke permissions such as:

- self-billing,
- self-billing RR,
- tax representative operations.

C# example: [KSeF.Client.Tests.Core\E2E\Permissions\EuEntityPermission\EuEntityPermissionE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/Permissions/EuEntityPermission/EuEntityPermissionE2ETests.cs)

```csharp
await ksefClient.RevokeAuthorizationsPermissionAsync(permissionId, accessToken, cancellationToken);
```

Java example: [ProxyPermissionIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/ProxyPermissionIntegrationTest.java)

```java
OperationResponse response = ksefClient.revokeAuthorizationsPermission(operationId, accessToken);
```

## Searching for granted permissions

KSeF provides a set of endpoints that allow querying the list of active permissions granted to users and entities. These mechanisms are essential for auditing, access status review, and building administrative interfaces (e.g., for managing the organization's access structure). This section provides an overview of search methods, broken down by permission category.

---

### Downloading a list of your own permissions

This query retrieves a list of permissions held by the authenticated entity. This list includes permissions:

- given directly in the current context
- granted by a superior entity
- given indirectly, where the context is the intermediary or target entity
- assigned to an entity for handling invoices ( `"InvoiceRead"` and `"InvoiceWrite"` ) by another entity, if the authenticated entity has ownership rights ( `"Owner"` )

POST [/permissions/query/personal/grants](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Wyszukiwanie-nadanych-uprawnien/paths/~1api~1v2~1permissions~1query~1personal~1grants/post)

Java example: [SearchPersonalGrantPermissionIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/SearchPersonalGrantPermissionIntegrationTest.java)

```java
QueryPersonalGrantResponse response = ksefClient.searchPersonalGrantPermission(request, pageOffset, pageSize, token.accessToken());

```

---

### Downloading the list of authorizations to work in KSeF granted to individuals or entities

The query allows you to retrieve a list of permissions granted to individuals or entities—e.g., company employees. Filtering is possible by permission type, status ( `Active` / `Inactive` ), and sender and recipient IDs. This endpoint is often used for onboarding, auditing, and monitoring personnel permissions.

POST [/permissions/query/persons/grants](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Wyszukiwanie-nadanych-uprawnien/paths/~1api~1v2~1permissions~1query~1persons~1grants/post)

Field | Description
:-- | :--
`authorIdentifier` | Identifier of the entity granting the authorization. Tax Identification `Nip` , Personal Identification `Pesel` ), `Fingerprint` , `System`
`authorizedIdentifier` | Identifier of the entity to which the authorization was granted. Tax Identification `Nip` , Personal Identification `Pesel` ), `Fingerprint`
`targetIdentifier` | Target entity identifier (for indirect authorizations). `Nip` , `AllPartners`
`permissionTypes` | Permission types to filter. `"CredentialsManage"` , `"CredentialsRead"` , `"InvoiceWrite"` , `"InvoiceRead"` , `"Introspection"` , `"SubunitManage"` , `"EnforcementOperations"`
`permissionState` | Permission Status: `Active` / `Inactive`

C# example: [KSeF.Client.Tests.Core\E2E\Permissions\SubunitPermission\SubunitPermissionsE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/Permissions/SubunitPermission/SubunitPermissionsE2ETests.cs)

```csharp
PagedPermissionsResponse<Client.Core.Models.Permissions.PersonPermission> response =
    await KsefClient
    .SearchGrantedPersonPermissionsAsync(
        personPermissionsQueryRequest,
        accessToken,
        pageOffset: 0,
        pageSize: 10,
        CancellationToken);
```

Java example: [PersonPermissionIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/PersonPermissionIntegrationTest.java)

```java
PersonPermissionsQueryRequest request = new PersonPermissionsQueryRequestBuilder()
        .withQueryType(PersonPermissionQueryType.PERMISSION_GRANTED_IN_CURRENT_CONTEXT)
        .build();

QueryPersonPermissionsResponse response = ksefClient.searchGrantedPersonPermissions(request, pageOffset, pageSize, accessToken);


```

---

### Downloading the list of permissions of unit and subordinate entity administrators

This endpoint is used to retrieve information about administrators of subordinate units or entities (e.g., branches, VAT groups). It allows you to monitor who has management privileges for a given subordinate structure, identified by `InternalId` or `Nip` .

POST [/permissions/query/subunits/grants](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Wyszukiwanie-nadanych-uprawnien/paths/~1api~1v2~1permissions~1query~1subunits~1grants/post)

Field | Description
:-- | :--
`subjectIdentifier` | Subordinate entity identifier. `InternalId` or `Nip` Identification Number

C# example: [KSeF.Client.Tests.Core\E2E\Permissions\SubunitPermission\SubunitPermissionsE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/Permissions/SubunitPermission/SubunitPermissionsE2ETests.cs)

```csharp
SubunitPermissionsQueryRequest subunitPermissionsQueryRequest = new SubunitPermissionsQueryRequest();
PagedPermissionsResponse<Client.Core.Models.Permissions.SubunitPermission> response =
    await KsefClient
    .SearchSubunitAdminPermissionsAsync(
        subunitPermissionsQueryRequest,
        accessToken,
        pageOffset: 0,
        pageSize: 10,
        CancellationToken);
```

Java example: [SubUnitPermissionIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/SubUnitPermissionIntegrationTest.java)

```java
SubunitPermissionsQueryRequest request = new SubunitPermissionsQueryRequestBuilder()
        .withSubunitIdentifier(new SubunitPermissionsSubunitIdentifier(SubunitPermissionsSubunitIdentifier.IdentifierType.INTERNALID, subUnitNip))
        .build();

QuerySubunitPermissionsResponse response = ksefClient.searchSubunitAdminPermissions(request, pageOffset, pageSize, accessToken);


```

---

### Getting the entity role list

Endpoint returns a set of roles assigned to the context in which we are authenticated (i.e., the one on whose behalf the query is being executed). This function is primarily used for automatic access checking by client systems.

GET [/permissions/query/entities/roles](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Wyszukiwanie-nadanych-uprawnien/paths/~1api~1v2~1permissions~1query~1entities~1roles/get)

C# example: [KSeF.Client.Tests.Core\E2E\Permissions\SubunitPermission\SubunitPermissionsE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/Permissions/SubunitPermission/SubunitPermissionsE2ETests.cs)

```csharp
PagedRolesResponse<EntityRole> response =
    await KsefClient
    .SearchEntityInvoiceRolesAsync(
        accessToken,
        pageOffset: 0,
        pageSize: 10,
        CancellationToken);
```

Java example: [SearchEntityInvoiceRoleIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/SearchEntityInvoiceRoleIntegrationTest.java)

```java
QueryEntityRolesResponse response = ksefClient.searchEntityInvoiceRoles(0, 10, token);
```

---

### Downloading the list of subordinate entities

This allows you to obtain information about related subordinate entities for the context in which you are authenticated (i.e., the one on whose behalf the query is being executed). This function is primarily used to verify the structure of local government units or VAT groups.

POST [/permissions/query/subordinate-entities/roles](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Wyszukiwanie-nadanych-uprawnien/paths/~1api~1v2~1permissions~1query~1subordinate-entities~1roles/post)

Field | Description
:-- | :--
`subordinateEntityIdentifier` | Identifier of the entity to which the authorization was granted. `Nip`

C# example: [KSeF.Client.Tests.Core\E2E\Permissions\SubunitPermission\SubunitPermissionsE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/Permissions/SubunitPermission/SubunitPermissionsE2ETests.cs)

```csharp
SubordinateEntityRolesQueryRequest subordinateEntityRolesQueryRequest = new SubordinateEntityRolesQueryRequest();
PagedRolesResponse<SubordinateEntityRole> response =
    await KsefClient
    .SearchSubordinateEntityInvoiceRolesAsync(
        subordinateEntityRolesQueryRequest,
        accessToken,
        pageOffset: 0,
        pageSize: 10,
        CancellationToken);
```

Java example: [SearchSubordinateQueryIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/SearchSubordinateQueryIntegrationTest.java)

```java
SubordinateEntityRolesQueryResponse response = ksefClient.searchSubordinateEntityInvoiceRoles(queryRequest, pageOffset, pageSize,accessToken);
```

---

### Downloading the list of entity authorizations for handling invoices

This endpoint is used to review all granted subject permissions granted by the context in which we are authenticated or granted to the context in which we are authenticated. It supports filtering by permission type and recipient.

POST [/permissions/query/authorizations/grants](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Wyszukiwanie-nadanych-uprawnien/paths/~1api~1v2~1permissions~1query~1authorizations~1grants/post)

Field | Description
:-- | :--
`authorizingIdentifier` | Identifier of the entity granting the authorization. `Nip`
`authorizedIdentifier` | Identifier of the entity to which the authorization was granted. `Nip`
`queryType` | Query type. Specifies whether we are querying for granted or received permissions. `Granted` `Received`
`permissionTypes` | Permission types to filter. `"SelfInvoicing"` , `"TaxRepresentative"` , `"RRInvoicing"` ,

C# example: [KSeF.Client.Tests.Core\E2E\Permissions\SubunitPermission\SubunitPermissionsE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/Permissions/SubunitPermission/SubunitPermissionsE2ETests.cs)

```csharp
PagedAuthorizationsResponse<AuthorizationGrant> response =
        await KsefClient
        .SearchEntityAuthorizationGrantsAsync(
            entityAuthorizationsQueryRequest,
            accessToken,
            pageOffset: 0,
            pageSize: 10,
            CancellationToken);
```

Java example: [ProxyPermissionIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/ProxyPermissionIntegrationTest.java)

```java
        EntityAuthorizationPermissionsQueryRequest request = new EntityAuthorizationPermissionsQueryRequestBuilder()
        .withQueryType(QueryType.GRANTED)
        .build();

QueryEntityAuthorizationPermissionsResponse response = ksefClient.searchEntityAuthorizationGrants(request, pageOffset, pageSize, accessToken);


```

---

### Downloading the list of rights of administrators or representatives of EU entities entitled to self-billing

EU entities can also be assigned permissions to use the KSeF. In this section, you can download information about their access, including EU VAT identifiers and certificate fingerprints.

POST [/permissions/query/eu-entities/grants](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Wyszukiwanie-nadanych-uprawnien/paths/~1api~1v2~1permissions~1query~1eu-entities~1grants/post)

Field | Description
:-- | :--
`vatUeIdentifier` | EU VAT ID.
`authorizedFingerprintIdentifier` | Authorized entity certificate fingerprint.
`permissionTypes` | Permission types to filter. Possible values are: `VatUeManage` , `InvoiceWrite` , `InvoiceRead` , `Introspection` .

C# example: [KSeF.Client.Tests.Core\E2E\Permissions\SubunitPermission\SubunitPermissionsE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/Permissions/SubunitPermission/SubunitPermissionsE2ETests.cs)

```csharp
PagedPermissionsResponse<Client.Core.Models.Permissions.EuEntityPermission> response =
    await KsefClient
    .SearchGrantedEuEntityPermissionsAsync(
        euEntityPermissionsQueryRequest,
        accessToken,
        pageOffset: 0,
        pageSize: 10,
        CancellationToken);
```

Java example: [EuEntityPermissionIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/EuEntityPermissionIntegrationTest.java)

```java
EuEntityPermissionsQueryRequest request = new EuEntityPermissionsQueryRequestBuilder()
   .withAuthorizedFingerprintIdentifier(subjectContext)
   .build();

QueryEuEntityPermissionsResponse response = createKSeFClient().searchGrantedEuEntityPermissions(request, pageOffset, pageSize, accessToken);
```

## Operations

The National e-Invoice System enables tracking and verification of the status of operations related to authorization management. Each authorization granting or revoking is performed as an asynchronous operation, the status of which can be monitored using a unique reference identifier ( `operationReferenceNumber` ). This section presents a mechanism for retrieving the operation status and its interpretation in the context of automation and validation of administrative activities in the National e-Invoice System.

### Getting the operation status

After granting or revoking permission, the system returns an operation reference number ( `operationReferenceNumber` ). This identifier allows you to check the current status of the request: whether it was successful, whether an error occurred, or whether processing is still ongoing. This information can be crucial in supervisory systems, automatic retry logic, or administrative reporting. This section presents an example API call for retrieving the status of an operation.

GET [/permissions/operations/{operationReferenceNumber}](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Operacje/paths/~1api~1v2~1permissions~1operations~1%7BoperationReferenceNumber%7D/get)

Each permission granting operation returns an operation identifier that should be used to check the status of that operation.

C# example: [KSeF.Client.Tests.Core\E2E\Permissions\SubunitPermission\SubunitPermissionsE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/Permissions/SubunitPermission/SubunitPermissionsE2ETests.cs)

```csharp
var operationStatus = await ksefClient.OperationsStatusAsync(referenceNumber, accessToken, cancellationToken);
```

Java example: [EuEntityPermissionIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/EuEntityPermissionIntegrationTest.java)

```java
PermissionStatusInfo status = ksefClient.permissionOperationStatus(referenceNumber, accessToken);
```

### Checking the status of consent to issue invoices with attachments

Consent is required to issue invoices with attachments and is valid within the current context ( `ContextIdentifier` ) used for authentication. Consent is granted outside the API, only in the e-Tax Office service, and applications can be submitted from January 1, 2026. The API does not provide the consent submission process.

GET [/permissions/attachments/status](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Operacje/paths/~1api~1v2~1permissions~1attachments~1status/get)

Returns the consent status for the current context. If consent is not active, the invoice with an attachment sent to the KSeF API will be rejected.

Java example: [PermissionAttachmentStatusIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/PermissionAttachmentStatusIntegrationTest.java)

```java
PermissionAttachmentStatusResponse trueResponse = ksefClient.checkPermissionAttachmentInvoiceStatus(token.accessToken());
```

**Test environment**
 The test environment includes a POST `/testdata/attachment` endpoint, which allows a specified entity to send invoices with attachments. This endpoint is used solely to simulate consent in tests and operates within the current context.
