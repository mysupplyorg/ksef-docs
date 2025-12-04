# KSeF number – structure and validation

The KSeF number is a unique invoice identifier assigned by the system. It is always **35 characters** long and globally unique – it uniquely identifies each invoice in the KSeF.

## General structure of the issue

```
9999999999-RRRRMMDD-FFFFFFFFFFFF-FF
```

Where:

- `9999999999` – seller's tax identification number (10 digits),
- `RRRRMMDD` – date of invoice acceptance (year, month, day) for further processing,
- `FFFFFFFFFFFF` – technical part consisting of 12 characters in hexadecimal notation, only [0–9 A–F], uppercase letters,
- `FF` – CRC-8 checksum - 2 hexadecimal characters, only [0–9 A–F], uppercase letters.

## Example

```
5265877635-20250826-0100001AF629-AF
```

- `5265877635` - seller's tax identification number,
- `20250826` - date of receipt of the invoice for further processing,
- `0100001AF629` - technical part,
- `AF` - CRC-8 checksum.

## KSeF number validation

The validation process includes:

1. Checking if the number is **exactly 35 characters** long.
2. Separation of the data part (32 characters) and the checksum part (2 characters).
3. Calculating the checksum from part of the data **using the CRC-8 algorithm** .
4. Comparing the calculated sum with the value contained in the number.

## CRC-8 algorithm

To calculate the checksum, the **CRC-8** algorithm is used with the following parameters:

- **Polinom:** `0x07`
- **Initial value:** `0x00`
- **Result format:** 2-character hexadecimal notation (HEX, uppercase)

Example: if the calculated checksum is `0x46` , `"46"` will be added to the KSeF number.

## Example in C#:

```csharp
using KSeF.Client.Core;

bool isValid = KsefNumberValidator.IsValid(ksefNumber, out string message);
```

## Java example:

[OnlineSessionIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/OnlineSessionIntegrationTest.java)

```java
KSeFNumberValidator.ValidationResult result = KSeFNumberValidator.isValid(ksefNumber);

```
