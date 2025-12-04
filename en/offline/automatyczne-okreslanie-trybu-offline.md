## Automatically determine offline shipping mode

04/10/2025

In the case of invoices sent online ( `offlineMode: false` ), the KSeF system may assign them offline mode - based on a comparison of the date of issue with the date of acceptance for processing.

## Mechanism algorithm

For invoices sent as `offlineMode: false` the system compares:

- invoice **issue date** ( `issueDate` , e.g. `P_1` for an invoice compliant with FA(3)),
- **date of receipt** of the invoice in the KSeF system for further processing ( `invoicingDate` ).

Rules:

- If the calendar day of `issueDate` is earlier than the calendar day of `invoicingDate` (comparison by date, not by time), the system automatically marks the invoice as **offline** , even if it was not declared as such.
- If `issueDate` and `invoicingDate` are the same, the invoice remains **online** .

The value `invoicingDate` depends on the shipping mode:

- **batch session** - `invoicingDate` is the moment the session was opened,
- **interactive session** - `invoicingDate` is the moment the invoice was sent.

This means that if, for example, an invoice was issued on 2025-10-03 ( `P_1` ) and sent on 2025-10-04 at 00:00:01, it will be marked as an offline invoice despite offlineMode: false.

## Examples

**Batch session** opened at 11:59:59 PM October 3: Even if the batch is sent after midnight, the invoices will remain online – because `invoicingDate` is October 3 (date the session was opened).

**Interactive session** started at 23:59:59 on October 3 and invoices were submitted after midnight: If `P_1` = 2025-10-03, the system will mark them as offline – because `P_1` is earlier than the submission day.

## Related documents

- [Offline modes](../tryby-offline.md)
