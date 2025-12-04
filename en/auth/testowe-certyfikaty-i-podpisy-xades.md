# XAdES Test Certificates and Signatures

This guide shows you how to **quickly launch** the [`KSeF.Client.Tests.CertTestApp`](https://github.com/CIRFMF/ksef-client-csharp) demo console application to:

- generating **a test (self-signed) certificate** for the KSeF test environment,
- building and **signing the XAdES** `AuthTokenRequest` document,
- sending the signed document to KSeF and **obtaining access tokens** (JWT).

> **Attention**
>
> - Self-signed certificates are **only allowed** in the **test** environment.
> - The data in the examples (NIP, reference number, tokens) are **fictitious** and are for demonstration purposes only.

---

## Prerequisites

- **.NET 10 SDK**
- Git
- Windows or Linux

---

## What does the app do?

- Gets **a challenge** from KSeF.
- Builds an `AuthTokenRequest` XML document.
- **Signs** an `AuthTokenRequest` document in **XAdES** format.
- Sends the signed document to KSeF and receives `referenceNumber` + `authenticationToken` .
- **Queries the status** of the authentication operation until successful.
- Upon success, it retrieves a pair of tokens: `accessToken` and `refreshToken` (JWT).
- Saves artifacts (including **test certificate** and **signed XML** ) to files if `file` output is selected.

---

## Windows

1. **Install .NET 10 SDK** :

    ```powershell
    winget install Microsoft.DotNet.SDK.10
    ```

    Alternatively: download the installer from the .NET website.

2. **Open a new terminal window** (PowerShell/CMD).

3. **Check installation** :

    ```powershell
    dotnet --version
    ```

    Expected version number: `10.xx` .

4. **Clone the repository and go to the project** :

    ```powershell
    git clone https://github.com/CIRFMF/ksef-client-csharp.git
    cd ksef-client-csharp/KSeF.Client.Tests.CertTestApp
    ```

5. **Run (random NIP by default, result on screen)** :

    ```powershell
    dotnet run --framework net10.0
    ```

6. **Startup with parameters** :

    - `--output` – `screen` (default) or `file` (save results to files),
    - `--nip` {nip_number} - e.g. `--nip 8976111986` ,
    - optional: `--no-startup-warnings` .

    ```powershell
    dotnet run --framework net10.0 --output file --nip 8976111986 --no-startup-warnings
    ```

---

## Linux (Ubuntu/Debian)

1. **Add Microsoft repository and update packages** :

    ```bash
    wget https://packages.microsoft.com/config/ubuntu/$(lsb_release -rs)/packages-microsoft-prod.deb -O packages-microsoft-prod.deb
    sudo dpkg -i packages-microsoft-prod.deb
    sudo apt-get update
    ```

2. **Install .NET 10 SDK** :

    ```bash
    sudo apt-get install -y dotnet-sdk-10.0
    ```

3. **Refresh your shell environment or open a new terminal** :

    ```bash
    source ~/.bashrc
    ```

4. **Check installation** :

    ```bash
    dotnet --version
    ```

    Expected version number: `10.xx` .

5. **Clone the repository and go to the project** :

    ```bash
    git clone https://github.com/CIRFMF/ksef-client-csharp.git
    cd ksef-client-csharp/KSeF.Client.Tests.CertTestApp
    ```

6. **Run (result on screen, random NIP)** :

    ```bash
    dotnet run --framework net10.0
    ```

7. **Startup with parameters** :

    - `--output` – `screen` (default) or `file` (save results to files),
    - `--nip` {nip_number} - e.g. `--nip 8976111986` ,
    - optional: `--no-startup-warnings` .

    ```bash
    dotnet run --framework net10.0 --output file --nip 8976111986 --no-startup-warnings
    ```

---

Related documents:

- [Authentication in KSeF](../uwierzytelnianie.md)
- [Signature XAdES](podpis-xades.md)

---
