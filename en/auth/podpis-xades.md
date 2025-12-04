## Signature XAdES

https://www.w3.org/TR/XAdES/

Acceptable signature formats:

- surrounded
- surrounding

Signature in external format (detached) is not accepted.

Allowable transforms included in the XAdES signature:

- http://www.w3.org/TR/1999/REC-xpath-19991116 - not(ancestor-or-self::ds:Signature)
- http://www.w3.org/2002/06/xmldsig-filter2
- http://www.w3.org/2000/09/xmldsig#enveloped-signature
- http://www.w3.org/2000/09/xmldsig#base64
- http://www.w3.org/2006/12/xml-c14n11
- http://www.w3.org/2006/12/xml-c14n11#WithComments
- http://www.w3.org/2001/10/xml-exc-c14n#
- http://www.w3.org/2001/10/xml-exc-c14n#WithComments
- http://www.w3.org/TR/2001/REC-xml-c14n-20010315
- http://www.w3.org/TR/2001/REC-xml-c14n-20010315#WithComments

### Permitted certificate types

Allowed certificate types in XAdES signature:

- Qualified certificate of a natural person – containing the PESEL or NIP number of the person authorized to act on behalf of the company,
- Qualified certificate of the organization (so-called company stamp) - containing the Tax Identification Number,
- Trusted Profile (ePUAP) – enables signing a document; used by natural persons,
- KSeF Internal Certificate – issued by the KSeF system. This certificate is not a qualified certificate, but is recognized in the authentication process.

**Qualified certificate** – a certificate issued by a qualified trust service provider, entered into the [EU Trusted List (EUTL)](https://eidas.ec.europa.eu/efda/trust-services/browse/eidas/tls) , in accordance with the eIDAS Regulation. The KSeF accepts qualified certificates issued in Poland and other European Union member states.

### Required attributes of qualified certificates

#### Qualified signature certificates (issued to individuals)

Required entity attributes:<br>

Identifier (OID) | Name | Meaning
--- | --- | ---
2.5.4.42 | givenName | name
2.5.4.4 | surname | last name
2.5.4.5 | serialNumber | serial number
2.5.4.3 | commonName | common name of the certificate owner
2.5.4.6 | countryName | country name, ISO 3166 code

Recognized patterns of `serialNumber` attribute:<br> **(PNOPL|PESEL).*?(?<number> \d{11})</number>**<br> **(TINPL|NIP).*?(?<number> \d{10})</number>**<br>

#### Qualified Seal Certificates (issued for organizations)

Required entity attributes:<br>

Identifier (OID) | Name | Meaning
--- | --- | ---
2.5.4.10 | organizationName | full formal name of the entity for which the certificate is issued
2.5.4.97 | organizationIdentifier | entity identifier
2.5.4.3 | commonName | common name of the organization
2.5.4.6 | countryName | country name, ISO 3166 code

Unacceptable entity attributes:

Identifier (OID) | Name | Meaning
--- | --- | ---
2.5.4.42 | givenName | name
2.5.4.4 | surname | last name

Recognized patterns of `organizationIdentifier` attribute:<br> **(VATPL).*?(?<number> \d{10})</number>**<br>

### Certificate fingerprint

In the case of qualified certificates that do not have the appropriate identifiers stored in the OID.2.5.4.5 entity attribute, it is possible to authenticate with such a certificate after granting permissions to the SHA-256 hash (so-called fingerprint) of this certificate.
