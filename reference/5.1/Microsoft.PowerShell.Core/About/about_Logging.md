---
description: PowerShell logs internal operations from the engine, providers, and cmdlets.
Locale: en-US
ms.date: 01/09/2024
online version: https://learn.microsoft.com/powershell/module/microsoft.powershell.core/about/about_logging?view=powershell-5.1&WT.mc_id=ps-gethelp
schema: 2.0.0
title: about_Logging
---

# about_Logging

## Short description
PowerShell logs internal operations from the engine, providers, and cmdlets to
the Windows event log.

## Long description

PowerShell logs details about PowerShell operations, such as starting and
stopping the engine and providers, and executing PowerShell commands.

> [!NOTE]

> Windows PowerShell 5.1 (and older) include **EventLog** cmdlets for the
> Windows event logs. In those versions, to display the list of **EventLog**
> cmdlets type: `Get-Command -Noun EventLog`. For more information, see the
> cmdlet documentation and [about_EventLogs][02] for your
> version of Windows PowerShell.

For information about logging in PowerShell 7, see the following articles:

- [about_Logging_Windows][04]
- [about_Logging_Non-Windows][03]

## Viewing the PowerShell event log entries on Windows

PowerShell logs can be viewed using the Windows Event Viewer. The event log is
located in the **Application and Services Logs** group and is named
**Microsoft-Windows-PowerShell**. The associated ETW provider `GUID` is
`{A0C1853B-5C40-4B15-8766-3CF1C58F985A}`.

When Script Block Logging is enabled, PowerShell logs the following events to
the **Microsoft-Windows-PowerShell/Operational** log:

|  Field  |       Value       |
| ------- | ----------------- |
| EventId | `4104` / `0x1008` |
| Channel | `Operational`     |
| Level   | `Verbose`         |
| Opcode  | `Create`          |
| Task    | `CommandStart`    |
| Keyword | `Runspace`        |

## Enabling Script Block Logging

When you enable Script Block Logging, PowerShell records the content of all
script blocks that it processes. Once enabled, any new PowerShell session logs
this information.

> [!NOTE]
> To ensure that sensitive data is protected, enable Protected Event Logging
> when using Script Block Logging for anything other than diagnostics purposes.

Script Block Logging can be enabled via Group Policy or a registry setting.

### Using Group Policy

To enable automatic transcription, enable the **Turn on PowerShell Script Block
Logging** feature in Group Policy through **Administrative Templates** ->
**Windows Components** -> **Windows PowerShell**.

### Using the Registry

Run the following function:

```powershell
function Enable-PSScriptBlockLogging {
    $basePath = @(
        'HKLM:\Software\Policies\Microsoft\Windows'
        'PowerShell\ScriptBlockLogging'
    ) -join '\'

    if (-not (Test-Path $basePath)) {
        $null = New-Item $basePath -Force
    }

    Set-ItemProperty $basePath -Name EnableScriptBlockLogging -Value "1"
}
```

## Protected Event Logging

Increasing the level of logging on a system increases the possibility that
logged content may contain sensitive data. For example, with script logging
enabled, credentials or other sensitive data used by a script can be written to
the event log. When a machine that has logged sensitive data is compromised,
the logs can provide an attacker with information needed to extend their reach.

To protect this information, Windows 10 introduces Protected Event Logging.
Protected Event Logging lets participating applications encrypt sensitive data
written to the event log. Later, you can decrypt and process these logs on a
more secure and centralized log collector.

Event log content is protected using the IETF Cryptographic Message Syntax
(CMS) standard. CMS uses public key cryptography. The keys used to encrypt
content and decrypt content are kept separate.

The public key can be shared widely and isn't sensitive data. Any content
encrypted with this public key can only be decrypted by the private key. For
more information about Public Key Cryptography, see
[Wikipedia - Public Key Cryptography][06].

To enable a Protected Event Logging policy, deploy a public key to all machines
that have event log data to protect. The corresponding private key is used to
post-process the event logs at a more secure location such as a central event
log collector, or [SIEM][07] aggregator. You can set up SIEM in Azure. For more
information, see [Generic SIEM integration][01].

### Enabling Protected Event Logging via Group Policy

To enable Protected Event Logging, enable the `Enable Protected Event Logging`
feature in Group Policy through `Administrative Templates -> Windows Components
-> Event Logging`. This setting requires an encryption certificate, which you
can provide in one of several forms:

- The content of a base-64 encoded X.509 certificate (for example, as offered
  by the `Export` option in Certificate Manager).
- The thumbprint of a certificate that can be found in the Local Machine
  certificate store (can be deployed by PKI infrastructure).
- The full path to a certificate (can be local, or a remote share).
- The path to a directory containing a certificate or certificates (can be
  local, or a remote share).
- The subject name of a certificate that can be found in the Local Machine
  certificate store (can be deployed by PKI infrastructure).

The resulting certificate must have `Document Encryption` as an enhanced key
usage (`1.3.6.1.4.1.311.80.1`), and either `Data Encipherment` or
`Key Encipherment` key usages enabled.

> [!WARNING]
> The private key shouldn't be deployed to the machines logging events. It
> should be kept in a secure location where you decrypt the messages.

### Decrypting Protected Event Logging messages

The following script retrieves and decrypts events, assuming that you have the
private key:

```powershell
Get-WinEvent Microsoft-Windows-PowerShell/Operational |
    Where-Object Id -EQ 4104 |
    Unprotect-CmsMessage
```

## See also

- [PowerShell the Blue Team][05]
- [Generic SIEM integration][01]

<!-- link references -->
[01]: /cloud-app-security/siem
[02]: about_Eventlogs.md
[03]: /powershell/module/microsoft.powershell.core/about/about_logging_non-windows
[04]: /powershell/module/microsoft.powershell.core/about/about_logging_windows
[05]: https://devblogs.microsoft.com/powershell/powershell-the-blue-team/
[06]: https://wikipedia.org/wiki/Public-key_cryptography
[07]: https://wikipedia.org/wiki/Security_information_and_event_management
