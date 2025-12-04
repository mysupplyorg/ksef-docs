## Sample scenarios

05/08/2025

### Scenario No. 1 – Bailiff

If we want to use the KSeF system in the test environment as an individual with bailiff privileges, we should add such a person using the `/v2/testdata/person` endpoint, setting *the isBailiff* flag to **true** .

Sample JSON:

```json
{
  "nip": "7980332920",
  "pesel": "30112206276",
  "description": "Komornik",
  "isBailiff": true
}
```

As a result of this operation, the person logging in in the context of the provided NIP number, using the PESEL number or NIP number, will receive ownership rights ( **Owner** ) and enforcement rights ( **EnforcementOperations** ), which will enable the use of the system from the perspective of a bailiff.

---

### Scenario No. 2 – JDG

If we want to use the KSeF system as a sole proprietorship in the test environment, we should add such a person using the `/v2/testdata/person` endpoint, setting *the isBailiff* flag to **false** .

Sample JSON:

```json
{
  "nip": "7980332920",
  "pesel": "30112206276",
  "description": "JDG",
  "isBailiff": false
}
```

As a result of this operation, the person logging in in the context of the provided NIP, using the PESEL number or NIP, will receive ownership rights ( **Owner** ), which will enable the use of the system from the perspective of a single-family home owner (JDG).

---

### Scenario No. 3 – VAT Group

If we want to create a VAT group structure in the test environment and grant permissions to the group administrator and administrators of its members, in the first step we should create a structure of entities using the endpoint `/v2/testdata/subject` , indicating the Tax Identification Number of the parent unit and the child units.

Sample JSON:

```json
{
  "subjectNip": "3755747347",
  "subjectType": "VatGroup",
  "description": "Grupa VAT",
  "subunits": [
    {
      "subjectNip": "4972530874",
      "description": "NIP 4972530874: członek grupy VAT dla 3755747347"
    },
    {
      "subjectNip": "8225900795",
      "description": "NIP 8225900795: członek grupy VAT dla 3755747347"
    }
  ]
}
```

As a result, the specified entities and the links between them will be created in the system. Next, the person must be granted permissions regarding the VAT group's Tax Identification Number (NIP), in accordance with ZAW-FA rules. This operation can be performed using the `/v2/testdata/permissions` method.

Sample JSON for an authorized person in the context of a VAT group:

```json
{
  "contextIdentifier": {
    "value": "3755747347",
    "type": "nip"
  },
  "authorizedIdentifier": {
    "value": "38092277125",
    "type": "pesel"
  },
  "permissions": [
    {
      "permissionType": "InvoiceRead",
      "description": "praca w kontekście 3755747347: uprawniony PESEL: 38092277125, Adam Abacki"
    },
    {
      "permissionType": "InvoiceWrite",
      "description": "praca w kontekście 3755747347: uprawniony PESEL: 38092277125, Adam Abacki"
    },
    {
      "permissionType": "Introspection",
      "description": "praca w kontekście 3755747347: uprawniony PESEL: 38092277125, Adam Abacki"
    },
    {
      "permissionType": "CredentialsRead",
      "description": "praca w kontekście 3755747347: uprawniony PESEL: 38092277125, Adam Abacki"
    },
    {
      "permissionType": "CredentialsManage",
      "description": "praca w kontekście 3755747347: uprawniony PESEL: 38092277125, Adam Abacki"
    },
    {
      "permissionType": "SubunitManage",
      "description": "praca w kontekście 3755747347: uprawniony PESEL: 38092277125, Adam Abacki"
    }
  ]
}
```

This operation can be performed both for VAT groups (as above) and for VAT group members. It should be noted that while this is the only way to grant initial permissions for a VAT group, this is not necessary for group members. This can be done using the standard /v2/permissions/subunit/grants endpoint, appointing administrators for VAT group members.

Alternatively, you can use the endpoint described above to create test data. Sample JSON for granting `CredentialsManage` permission to a group member administrator:

```json
{
  "contextIdentifier": {
    "value": "4972530874",
    "type": "nip"
  },
  "authorizedIdentifier": {
    "value": "3388912629",
    "type": "nip"
  },
  "permissions": [
    {
      "permissionType": "CredentialsManage",
      "description": "praca w kontekście 4972530874: uprawniony NIP: 3388912629, Bogdan Babacki"
    }
  ]
}
```

Thanks to this operation, the representative of a VAT group member gains the ability to grant authorizations to himself or other people (e.g. employees) in a standard manner, via the KSeF system.

### Scenario No. 4 – Enabling the option to send invoices with attachments

In the test environment, you can simulate an entity that has the ability to send invoices with attachments enabled. This operation should be performed using the /testdata/attachment endpoint.

```json
{
  "nip": "4972530874"
}
```

As a result, the entity with NIP 4972530874 will be able to send invoices containing attachments.

### Scenario No. 5 – Disabling the ability to send invoices with attachments

To test a situation in which a given entity is no longer able to send invoices with attachments, use the /testdata/attachment/revoke endpoint.

```json
{
  "nip": "4972530874"
}
```

As a result, the entity with NIP 4972530874 loses the ability to send invoices containing attachments
