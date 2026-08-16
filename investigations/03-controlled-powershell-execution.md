# Investigation 03: Controlled PowerShell Execution from a Temporary Directory

## Status

**Closed - True-Positive Detection / Authorized Benign Lab Activity**

## Controlled test

A harmless script was created under the Windows user temporary directory and executed with PowerShell using `-ExecutionPolicy Bypass`.

The script content was intentionally inert:

```powershell
Write-Output "SOC-LAB-CONTROLLED-TEST"
```

The test then removed the temporary script.

## Alert

- **Wazuh rule:** 92029
- **Description:** Powershell executed script from suspicious location
- **Level:** 6
- **MITRE ATT&CK:** T1059.001 - PowerShell
- **Tactic:** Execution
- **Telemetry:** Sysmon Event ID 1, Process Create

## Observed process

```text
Image:
C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe

Command line:
powershell.exe -NoProfile -ExecutionPolicy Bypass \
  -File C:\Users\<LAB-USER>\AppData\Local\Temp\soc-lab-test.ps1

Parent image:
C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe

Execution context:
Interactive lab user
Integrity level: High

Current directory:
C:\SOC\Sysmon\bin\
```

## Timeline correlation

The underlying Sysmon Process Create event occurred within the two-second controlled test window. The process command line contained the expected `soc-lab-test.ps1` path and the expected `-ExecutionPolicy Bypass` argument, tying the alert directly to the authorized exercise.

The Wazuh alert was indexed shortly afterward, demonstrating the expected delay between endpoint event creation and searchable alert availability.

## Initial concern

Executing a PowerShell script from a temporary user directory with execution-policy bypass is behavior that deserves investigation in a production environment. Temporary directories are frequently used for staged scripts and payloads, and execution-policy bypass can be associated with attempts to run scripts outside normal policy constraints.

## Analysis

The detection itself was accurate. Wazuh correctly identified the suspicious execution pattern.

However, the surrounding evidence established that the event was generated intentionally as part of the controlled SOC lab exercise. The script contained only a `Write-Output` statement and produced the expected marker text. Process ancestry matched the test procedure: an existing interactive PowerShell session launched the child PowerShell process that executed the temporary script.

No evidence from the investigated event indicated persistence, credential access, malicious payload execution, or unauthorized activity.

This makes the alert a **true-positive detection of suspicious behavior**, while the underlying activity remains **authorized and benign**.

## Disposition

**Authorized benign lab activity. Close after validation.**

- Escalation: Not required.
- Remediation: None required.

## Analyst takeaway

A true-positive detection does not mean the underlying activity is necessarily malicious. The rule correctly detected a behavior worth triaging; command-line inspection, process ancestry, timing, and known test context established the final disposition.
