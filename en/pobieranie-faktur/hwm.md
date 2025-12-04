# High Water Mark (HWM)

25/11/2025

The High Water Mark (HWM) mechanism describes how KSeF manages data completeness over time for the `PermanentStorage` date.

At any time, the system knows the point in time ( `HWM` ) by which it is sure that all invoices have been saved and that no new documents will appear with a `PermanentStorage` date earlier or equal to that point in time.

![HWM](hwm.png)

- For time ≤ `HWM` - all invoices with a `PermanentStorage` date in this interval have already been permanently saved in KSeF. The system guarantees that no new invoices with a `PermanentStorage` date ≤ `HWM` will appear in the future.
- In the interval ( `HWM` , `Teraz` ):
    - some invoices are already visible and can be returned in the inquiry,
    - due to the asynchronous and multi-threaded nature of the writing process, new invoices may still appear in this interval, i.e. with a `PermanentStorage` date within the range ( `HWM` , `Teraz` ].

Application:

- everything that ≤ `HWM` can be treated as a **closed** and **complete** set,
- anything &gt; `HWM` is **potentially incomplete** and needs to be handled carefully when synchronizing.

## Scenario 1 - "HWM only" synchronization

![HWM-1](hwm-1.png)

With each query, the system retrieves invoices **from the "last known point" only up to the current `HWM` value** . The new `HWM` value becomes the starting point for the next interval.

Advantages:

- `HWM` data is definitive - there is no need to check the same interval again,
- the number of duplicates between subsequent downloads is minimal.

Consequences:

- some of the latest invoices from the scope `(HWM, Teraz]` are not visible in the local system - they will appear only after `HWM` is moved in the next cycle.

This scenario is recommended for incremental, automatic data synchronization where traffic optimization and minimizing duplicates are more important than immediate availability of the latest invoices.

## Scenario 2 - "Now" synchronization

![HWM-2](hwm-2.png)

The system integrating with KSeF performs cyclical, incremental queries **from the last starting point up to `Teraz`** and saves all returned invoices, also from the interval `(HWM, Teraz]` .

Since the data in this range may be incomplete, **the next query repeats part of the range** - at least from the previous `HWM` to the new `Teraz` . Deduplication is necessary on the local system side (e.g. after the KSeF number).

Advantages:

- the local system (and user) sees the latest invoices as quickly as possible, without waiting for `HWM` to "catch up" with them.

Consequences:

- the range `(HWM, Teraz]` must be checked again on the next query,
- duplicates will appear and need to be deleted on the local system side.

The same mechanism can also be used **ad hoc** , when the user manually requests a data refresh - the system then downloads "here and now" the latest available invoices from the last known date up to `Teraz` .

## Related documents

- [Incremental invoice download](przyrostowe-pobieranie-faktur.md)
