# Invoice verification

22/09/2025

An invoice submitted to the KSeF system is subject to a series of technical and semantic checks. Verification includes the following criteria:

## XSD schema compliance

The invoice must be prepared in XML format, encoded in UTF-8 without the BOM character (first 3 bytes 0xEF 0xBB 0xBF), consistent with the declared schema provided when opening the session.

## Uniqueness of the invoice

- KSeF detects duplicate invoices globally, based on data stored in the system. The duplicate identification criteria are a combination of:
    1. Seller's Tax Identification Number ( `Podmiot1:NIP` )
    2. Invoice type ( `RodzajFaktury` )
    3. Invoice number ( `P_2` )
- In case of a duplicate, error code 440 ("Duplicate invoice") is returned.
- The uniqueness of an invoice is maintained in KSeF for a period of 10 full years, counted from the end of the calendar year in which the invoice was issued.
- The uniqueness criterion always applies to the seller (Entity1: NIP). When invoices are issued on behalf of the same entity by different entities (e.g., branches, organizational units of local government units, other authorized entities), they must agree on numbering rules to avoid duplicates.

## Date validation

The invoice issue date ( `P_1` ) cannot be later than the date of acceptance of the document into the KSeF system.

## File size

- Maximum invoice size without attachments: **1 MB *** (1,000,000 bytes).
- Maximum invoice size with attachments: **3 MB *** (3,000,000 bytes).

## Quantitative restrictions

- The maximum number of invoices in one session (both interactive and batch) is 10,000 *.
- You can upload a maximum of 50 ZIP files in a batch upload; each file cannot exceed 100 MB (100,000,000 bytes) in size before encryption, and the total ZIP package size cannot exceed 5 GB (5,000,000,000 bytes).

## Correct encryption

- The invoice should be encrypted with the AES-256-CBC algorithm (256-bit symmetric key, 128-bit IV, with PKCS#7 padding).
- Symmetric key encrypted with the RSAES-OAEP algorithm (SHA-256/MGF1).

## Invoice metadata compliance in interactive session

- Calculation and verification of invoice digest along with file size.
- Calculation and verification of the hash of the encrypted invoice along with the file size.

## Attachment Restrictions

- Sending invoices with attachments is only allowed in batch mode.
     **Exception:** When submitting [a technical correction to an invoice offline,](../offline/korekta-techniczna.md) the use of an interactive session is permitted.
- The possibility of sending invoices with attachments requires prior notification of this option in the `e-Urząd Skarbowy` service.

## Permission requirements

Sending an invoice to KSeF requires having appropriate authorizations to issue it in the context of a given entity.

* **Note:** If the available [limits](../limity/limity.md) are insufficient in your organization's business scenarios, please contact the KSeF support department for an individual analysis and selection of the appropriate solution.
