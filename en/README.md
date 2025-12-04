# **KSeF 2.0 RC6.0 - Integrator's Guide**

02/12/2025

This document is a compendium of knowledge for developers, analysts, and system integrators who are integrating with the National e-Invoice System (KSeF) version 2.0. The guide focuses on the technical and practical aspects of communicating with the KSeF API.

## Introduction to KSeF 2.0

The National e-Invoice System (KSeF) is a central IT system used to issue and download structured invoices in electronic form.

## Contents

The guide is divided into thematic sections corresponding to key features and integration areas in the KSeF API:

- [Overview of key changes in KSeF 2.0](przeglad-kluczowych-zmian-ksef-api-2-0.md)
- [Changelog](api-changelog.md)
- Authentication
    - [Gaining access](uwierzytelnianie.md)
    - [Session management](auth/sesje.md)
- [Right](uprawnienia.md)
- [KSeF certificates](certyfikaty-KSeF.md)
- Offline modes
    - [Offline modes](tryby-offline.md)
    - [Technical correction](offline/korekta-techniczna.md)
- [QR codes](kody-qr.md)
- [Interactive session](sesja-interaktywna.md)
- [Batch session](sesja-wsadowa.md)
- Downloading invoices
    - [Downloading invoices](pobieranie-faktur/pobieranie-faktur.md)
    - [Incremental invoice download](pobieranie-faktur/przyrostowe-pobieranie-faktur.md)
- [KSeF token management](tokeny-ksef.md)
- [Limits](limity/limity.md)
- [Test data](dane-testowe-scenariusze.md)

For each area there is:

- detailed description of the operation,
- example call in C# and Java,
- links to the [OpenAPI](https://ksef-test.mf.gov.pl/docs/v2) specification and reference library code.

The code examples presented in the guide were prepared based on official open source libraries:

- [ksef-client-csharp](https://github.com/CIRFMF/ksef-client-csharp) – C# library
- [ksef-client-java](https://github.com/CIRFMF/ksef-client-java) – Java library

Both libraries are maintained and developed by teams at the Ministry of Finance and are publicly available on an open-source basis. They provide full support for the KSeF 2.0 API functionality, including support for all endpoints, data models, and sample call scenarios. Using these libraries significantly simplifies the integration process and minimizes the risk of API contract misinterpretation.

## System Environments

The list of KSeF API 2.0 environments is described in the [KSeF API 2.0 Environments](srodowiska.md) document.
