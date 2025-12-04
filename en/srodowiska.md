## KSeF API 2.0 Environments

01/09/2025

Information about public environments is summarized below.

Abbreviation | Environment | Description | Address (API) | Availability date | Acceptable invoice formats for shipping
--- | --- | --- | --- | --- | ---
**THESE** | Test<br> (Release Candidate) | An RC environment for early verification of planned changes before deployment to production. It enables testing using both the Taxpayer Application and custom API calls. | [https://ksef-test.mf.gov.pl/api/v2](https://ksef-test.mf.gov.pl/api/v2) | 30/09/2025 | FA(2), FA(3), FA_PEF (3), FA_KOR_PEF (3)
**TR** | Pre-production (Demo/Preprod) | An environment corresponding to a production configuration, intended for final integration validation in production-like conditions. | [https://ksef-demo.mf.gov.pl/api/v2](https://ksef-demo.mf.gov.pl/api/v2) | 15/10/2025 | FA(3), FA_PEF (3), FA_KOR_PEF (3)
**PRD** | Production | An environment for issuing and receiving invoices with full legal validity, a guaranteed SLA and relevant production data. | [https://ksef.mf.gov.pl/api/v2](https://ksef.mf.gov.pl/api/v2) | 01/02/2026 | FA(3), FA_PEF (3), FA_KOR_PEF (3)

Documentation for each environment is available at `/docs/v2` .

> <font color="red">Note:</font> The TE (test) environment is for testing integration with the KSeF API only. **Production invoices** or actual entity data should not be uploaded to it.

In the test environment, authentication using self-signed certificates is permitted, which in practice means that multiple integrators can [authenticate](uwierzytelnianie.md#proces-uwierzytelniania) against the same company. Therefore, data entered in the TE environment is not isolated and can be shared between integrators. Random Tax Identifiers (TINs) should be used for testing, avoiding any real data.

### Service work on test environments

Due to the planned, systematic development of the National e-Invoice System (KSeF 2.0), **from 1 October 2025,** periodic maintenance work may be carried out on the System's test environments.

This work will be carried out from **4:00 PM to 6:00 PM** . During this time, there may be temporary disruptions in access to the test environments.

After the work is completed **, only changes that impact integration** will be published in [the changelog](api-changelog.md) , e.g. changes to API behavior, modifications to contracts, XSD schemas, limits, etc. Changes that do not impact the API and are unnoticeable from an integration perspective, e.g. internal quality, performance, or security improvements **, may not be communicated** or will be presented collectively.
