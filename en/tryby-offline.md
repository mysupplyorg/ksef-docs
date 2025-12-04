## Offline modes

10/07/2025

## Entry

The KSeF system offers two basic modes of issuing invoices:

- `online` mode - invoice issued and sent in real time to the KSeF system,
- `offline` mode - invoice issued and sent to KSeF at a later, statutory date.

In offline mode, invoices are issued electronically, in accordance with the applicable FA(3) structure template. Key technical aspects:

- When sending an invoice – both in interactive and batch mode – `offlineMode: true` parameter must be set.
- For invoices sent online (offlineMode: false), the KSeF system can automatically assign them offline mode based on a comparison of the issue date with the receipt date. Mechanism details: [Automatic determination of offline sending mode](offline/automatyczne-okreslanie-trybu-offline.md) .
- The KSeF system only accepts the value contained in the `P_1` field of the e-invoice structure as the date of issue.
- The date of receipt of the invoice is the date of assignment of the KSeF number or, in the case of making it available outside KSeF, the date of its actual receipt.
- After issuing an invoice in offline mode, the client application should generate two [QR codes](kody-qr.md) to visualize the invoice:
    - **CODE I** – enables invoice verification in the KSeF system,
    - **CODE II** – confirms the identity of the issuer.
- The corrective invoice is sent only after the KSeF number has been assigned to the original document.
- If a submitted offline invoice is rejected for technical reasons, it is possible to use the [technical correction](/offline/korekta-techniczna.md) mechanism.

### Summary of invoicing modes in KSeF – offline24, offline and emergency

Mode | Responsibility side | Circumstances of launch | Deadline for submission to KSeF | Legal basis
--- | --- | --- | --- | ---
**offline24** | Client | No restrictions (taxpayer discretion) | by the next business day after the date of issue | Article 106nda of the VAT Act (KSeF 2.0 project)
**offline** | KSeF system | System unavailability (announced in the BIP and interface software) | until the next business day after the end of the unavailability | Article 106nh of the VAT Act (from 1 February 2026)
**emergency** | KSeF system | KSeF failure (message in BIP MF and in interface software) | up to 7 business days from the end of the failure (the counter is counted again with the next message) | Article 106nf of the VAT Act (from 1 February 2026)

### Deadline for sending the invoice to KSeF for subsequent events

In offline24 and offline modes, if a KSeF failure is announced during the expected invoice sending period (message in the BIP MF or in the interface software), the sending deadline is postponed and is counted from the date of end of the last announced failure, no longer than 7 business days.

In emergency mode, if another failure message appears during the seven-day period for sending the invoice, the deadline counter resets and runs from the date of the end of that next failure.

Declaring a total failure during any of the above modes results in the abolition of the obligation to send invoices to KSeF.

#### Example: offline24 mode with announced KSeF failure

1. 2025-07-08 (Wednesday)
    - The taxpayer generates an invoice in offline24 mode (offlineMode = true).
    - The deadline for submission to KSeF is set for 2025-07-09 (next business day).
2. 2025-07-09 (Thursday)
    - The Ministry of Finance publishes a notice about the KSeF failure (BIP and API interface).
    - According to the rule: the original deadline is extended and the new one is counted from the date the failure ends.
3. 2025-07-12 (Saturday)
    - The failure is removed – the system is available again.
    - The 7 business day period for submitting the overdue invoice begins.
4. 2025-07-22 (Tuesday)
    - The deadline is 7 business days from the end of the failure.
    - The application has time until this date to send the invoice to KSeF with offlineMode = true set.

### Total failure mode

In the event of a total failure announcement (social media: TV, radio, press, Internet):

- The invoice can be issued in paper or electronic form, without the obligation to use the FA(3) template.
- There is no obligation to send an invoice to KSeF after the failure has ceased.
- The transfer to the buyer takes place via any channel (in person, e-mail, other).
- The date of issue is always the actual date indicated on the invoice and the date of receipt is the date of actual receipt of the purchase invoice.
- Invoices from this mode are not provided with QR codes.
- A corrective invoice during an ongoing KSeF failure is issued analogously – outside the KSeF, with the actual date.

## Related documents

- [Offline technical invoice correction](offline/korekta-techniczna.md)
- [QR codes](kody-qr.md)
