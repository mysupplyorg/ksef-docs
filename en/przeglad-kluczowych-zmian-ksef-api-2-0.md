# KSeF API 2.0 – Key Changes Overview

09/06/2025

# Introduction

This document is aimed at technical teams and developers experienced with version 1.0 of the KSeF API. It provides an overview of the key changes introduced in version 2.0, along with a discussion of new capabilities and practical integration improvements.

The purpose of the document is:

- Indication of the main differences from version 1.0
- Presenting the benefits of migrating to version 2.0
- Facilitating integration preparation for system requirements implemented on February 1, 2026.

---

## Documentation and tools supporting integration with KSeF API 2.0

To facilitate the transition to the new API version and ensure correct implementation, a set of official materials and tools has been made available to support integrators:

**Technical documentation (OpenAPI)**

Version 2.0 of the KSeF API is described in the OpenAPI standard, which enables both easy documentation browsing by developers and automatic generation of integration code.

- **Documentation** (interactive online version): A documentation interface in the form of a technical portal, containing descriptions of methods, data structures, parameters, and examples. Intended for use by developers and integration analysts.<br> Test Environment (TE): [ [link](https://ksef-test.mf.gov.pl/docs/v2/index.html) ]<br> Production Environment (PRD)

- **Specification** (OpenAPI JSON file):
     A raw OpenAPI specification file in JSON format, intended for use in integration automation tools (e.g., code generators, API contract validators).<br> Test Environment (TE): [ [link](https://ksef-test.mf.gov.pl/docs/v2/openapi.json) ]<br> Production Environment (PRD)

**Official KSeF 2.0 Client Integration Library (open source)**

A public, open-source library, developed in parallel with subsequent API releases and maintained in full compliance with the specification. It is a recommended integration tool, enabling ongoing change tracking and reducing the risk of incompatibility with current system releases.

- **C#:** [ [link](https://github.com/CIRFMF/ksef-client-csharp) ]

- **Java:** [ [link](https://github.com/CIRFMF/ksef-client-java) ]

**Published packages**

The KSeF 2.0 Client library will be available in official package repositories for the most popular programming languages. It will be published as a NuGet package for the .NET platform and as a Maven Central artifact for Java. Publishing it in these repositories will allow for easy integration of the library into projects and automatic tracking of updates to future API versions.

**Step-by-step guide**

- **Integration guide/tutorial:**
     Practical step-by-step instructions with code snippets illustrating how to use key system endpoints.<br> [ [link](https://github.com/CIRFMF/ksef-docs) ]

# Key changes in API 2.0

## New authentication model based on JWT

In version 1.0, authentication was closely linked to opening an interactive session, which introduced many limitations and complicated integration.

In version 2.0:

- Authentication has been **separated as a separate process** , independent of session initialization.

- **Standard JWT tokens** have been introduced and are used to authorize all protected operations.

Benefits:

- compliance with market practices,
- the ability to reuse the token to create multiple sessions,
- **support for refreshing and invalidating tokens** .

Details of the authentication process: [ [link](https://github.com/CIRFMF/ksef-docs/blob/main/uwierzytelnianie.md) ]

## Unified initialization process for batch and interactive sessions

In API 2.0, the session opening process has been standardized and made independent of the operating mode. After obtaining an authentication token, you can open either an interactive session: POST /sessions/online, or a batch session: POST /sessions/batch.

In both cases, a simple JSON is passed containing:

- form code (formCode),

- encrypted AES key for encrypting invoice data (encryptionKey).

In the case of batch sending, a list of partial files with metadata included in the package is also transferred.

Details and examples of use:

- interactive shipping [ [link](https://github.com/CIRFMF/ksef-docs/blob/main/sesja-interaktywna.md) ]
- batch shipping [ [link](https://github.com/CIRFMF/ksef-docs/blob/main/sesja-wsadowa.md) ]

## Mandatory encryption of all invoices

In version 1.0, invoice encryption was mandatory only in batch mode. In interactive mode, encryption was available but optional.

In version 2.0, encryption of all invoices – both in batch and interactive mode – **is required** .

Each invoice or batch of invoices must be encrypted locally by the customer using **an AES key** that:

- is generated individually for each session,

- transferred to the system when opening the session (encryptionKey).

## Asymmetric encryption

Version 2.0 introduced `RSA-OAEP` with `SHA-256` and `MGF1-SHA256` . KSeF token encryption is implemented with a separate key than the symmetric key encryption used for invoices.

Current **public key certificates** are returned by the public endpoint: GET `/security/public-key-certificates`

## Consistency and New Naming Convention in API 2.0

One of the key changes in KSeF API 2.0 is the standardization and simplification of naming conventions for resources, parameters, and JSON models. Version 1.0 of the API contained a number of inconsistencies and excess complexity resulting from the evolution of the system.

In version 2.0:

- **Endpoints** have gained clear, RESTful naming (e.g. sessions/online, auth/token, permissions/entities/grants).

- **Operation names** have been simplified to reflect the actual action (e.g. grant, revoke, refresh).

- The structure **of headers, parameters, and data formats** has been cleaned up to be consistent and in line with REST API design best practices.

- **Data structures** are flat and clear – identifier and authorization types have explicitly defined enum types (Tax Identification Number, Personal Identification Number, Fingerprint), without the need to analyze subtypes.

Changes in version 2.0 also include updated names and data structures. While a full roadmap of these changes is not presented here, it is available in the OpenAPI v2 documentation and in the code samples on the official GitHub repository.

It should be emphasized that the changes are not drastic – **they do not affect** the overall logic of the KSeF system, but only organize and simplify the naming and structures, making the API more transparent and intuitive to use.

Migrating to version 2.0 should be treated as a change to the integration contract and requires adapting the implementation on the external systems' side. It's recommended to use the official **KSeF 2.0 Client** integration library, developed and maintained by the API team. This library implements all available endpoints and data models, significantly simplifying the migration process and ensuring stable support in future system versions.

## New module for managing internal certificates

KSeF API 2.0 introduces mechanisms for issuing and managing internal **KSeF certificates** [documentation link]. These certificates will enable authentication in KSeF and are required for offline invoicing [documentation link].

Entities that successfully complete the authentication process will be able to:

- submit an application for issuing an internal KSeF certificate containing selected attributes from the signature certificate used during authentication,

- download the issued certificate in digital form,

- check the status of the submitted certificate application,

- download the metadata list of issued certificates,

- check your certificate limit.

## Streamlining the batch shipping process

KSeF API 2.0 introduces significant improvements to batch session processing. The previous solution available in API 1.0 was inefficient – if even a single invoice in a batch contained an error, the entire shipment was rejected. This approach effectively limited integrators' use of batch mode and caused significant operational difficulties.

In the new solution, when sending a package of documents:

- each invoice is processed independently,

- any errors only affect specific invoices, not the entire shipment,

- the number of incorrect invoices is returned for the session status,

- a dedicated endpoint is available to download the detailed status of incorrectly processed invoices along with information about any errors.

This change significantly improves the reliability and efficiency of batch mode and is based on the same parcel transfer model without the risk of losing an entire parcel due to single errors.

## Verification of duplicate invoices

The method of detecting duplicates has been changed – now the invoice business data (Podmiot1:NIP, Podstawy Inwestycji, P_2) is checked, not the file hash. Details – [Duplicate verification](faktury/weryfikacja-faktury.md) .

## Changes to the Permissions module

Changes to the authorization module involve changes to some aspects of their functioning logic in KSeF 2.0.

In response to feedback, version 2.0 of the system introduced a mechanism for granting indirect authorizations, replacing the previous principle of inheriting authorizations for viewing and issuing invoices. The new interface allows for separating the viewing of client (partner) invoices and the ability to issue invoices on their behalf from the viewing and issuing of invoices by the entity (e.g., an accounting firm).

The mechanism involves a client granting an entity permission to view or issue invoices, with a special option enabled that allows the authorized entity to further transfer this permission. After receiving this permission, the entity can grant it to, for example, its employees. After completing these actions, these employees will be able to service the specified client within the scope of the granted permissions.

It is also possible for an entity to grant so-called general authorizations, which allow an employee authorized in this way to serve all of the entity's clients – of course, to the extent to which these clients have authorized the entity and taking into account the scope of the employee's authorizations (to view and/or issue invoices).

Thanks to this mechanism, the granting and functioning of authorizations to view and issue invoices within the entity itself are not linked to authorizations to process customer invoices. This gives entities better opportunities to profile employee authorizations than the previous authorization inheritance mechanism in KSeF. This involved granting authorizations to employees to view and issue invoices within the entity, and if the entity had the appropriate authorizations from customers, these authorizations were automatically transferred to the employees (inherited). As a result, an employee could only serve customers if they also had the right to view and/or issue the entity's invoices. In many cases, this was a redundant authorization, which could cause problems within organizations.

Additionally, the system has introduced a new authorization type and new login options, enabling self-billing for EU entities. Login is now possible in a context defined by a Polish entity (identified by its Tax Identification Number) and an entity from an EU country identified by its VAT number. In this context, authenticated representatives of a designated EU entity can issue self-billed invoices on behalf of the designated Polish entity.

Within the API definition, all permissions have been organized into logical groups corresponding to individual functional areas of the system.

## API call limits (rate limiting)

KSeF API 2.0 introduces a precise and predictable rate limit mechanism, replacing the existing solutions from version 1.0. Each endpoint in the system is subject to a request limit within specified time intervals: per second, minute, or hour.

The limit ranges and values are:

- publicly available: [API limits](limity/limity-api.md) ,
- varied depending on the environment (the test environment has limits that are less restrictive than the production environment),
- adapted to the nature of the operation:
    - for protected endpoints – limits are applied per context and IP address,
    - for open endpoints – limits are applied per IP address.

The new limits model is designed to allow for unimpeded testing of applications that integrate with the system. This solution provides greater transparency, predictability, and improved system resilience in both test and production environments.

## Helper API for generating test data (test environment)

The KSeF API 2.0 test environment will provide a dedicated **auxiliary API for generating test data** , enabling the rapid creation of companies, organizational structures, and contexts necessary for conducting integration tests.

Thanks to this solution, it will be possible, among other things:

- **simulating the establishment of a new business entity** ,

- **simulation of granting authorizations by ZAW-FA** ,

- **creation of units within the structure of local government units** ,

- creating **VAT tax groups** (GVAT) with related entities,

- **defining enforcement authorities** **and court bailiffs** .

Typically, the process of registering companies and granting authorizations to real entities in a production environment requires formal steps (e.g., a visit to the tax office). In a test environment, such data does not exist. Therefore, **a supporting API is an essential tool** , enabling integrators to independently create test entities on which they can freely implement and verify full application scenarios.
