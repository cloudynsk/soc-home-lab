# SOC Home Lab

A completed Windows/Linux security-monitoring lab built to practice entry-level SOC analyst workflows with **Wazuh** and **Sysmon**.

The project goes beyond installation: it demonstrates endpoint telemetry collection, alert triage, process-tree analysis, command-line investigation, MITRE ATT&CK interpretation, and documented disposition decisions.

## Lab goals

- Deploy a functional Wazuh-based monitoring environment.
- Enroll and monitor a Windows 11 endpoint.
- Add Sysmon telemetry for richer process and endpoint visibility.
- Validate the full telemetry path from endpoint activity to searchable Wazuh alerts.
- Investigate alerts using evidence rather than rule descriptions alone.
- Document analyst findings, escalation decisions, and final dispositions.

## Architecture

```text
Windows 11 endpoint
  ├─ Wazuh agent
  └─ Sysmon
        │
        │ Windows event telemetry
        ▼
Ubuntu Wazuh manager
        │
        ▼
Wazuh indexer
        │
        ▼
Wazuh dashboard / Threat Hunting
```

The lab was hosted in VirtualBox using an isolated private lab network. Internet access and lab-internal monitoring traffic were separated through distinct virtual adapters.

## What was implemented

- Ubuntu-based Wazuh manager, indexer, dashboard, and Filebeat stack.
- Windows 11 Wazuh endpoint agent enrollment and connectivity validation.
- Sysmon integration using the Windows Sysmon event channel.
- Wazuh collection of `Microsoft-Windows-Sysmon/Operational` telemetry.
- File Integrity Monitoring, Security Configuration Assessment, and rootcheck preserved alongside Sysmon collection.
- End-to-end validation from Windows process activity through Wazuh detection and dashboard search.
- Controlled alert-generation exercises for investigation practice.

## Investigation workflow

Each investigation follows the same basic analyst process:

```text
Alert
  ↓
Validate source telemetry
  ↓
Inspect process / command line / user context
  ↓
Correlate process ancestry and timeline
  ↓
Review MITRE ATT&CK mapping
  ↓
Determine benign, suspicious, or malicious context
  ↓
Document disposition and escalation decision
```

## Documented investigations

### 1. PowerShell temporary-file deletion during Wazuh assessment

A Wazuh rule detected PowerShell deleting a temporary file and mapped the activity to **T1070.004 - File Deletion**. Process ancestry and command-line analysis showed that `wazuh-agent.exe` launched PowerShell during a Security Configuration Assessment to export the local security policy, inspect the account-lockout setting, and delete its temporary file.

**Disposition:** Benign / expected security-tool activity.

[Read Investigation 01](investigations/01-wazuh-sca-powershell.md)

### 2. Account discovery using `net.exe` and `net1.exe`

Wazuh detected account-discovery behavior associated with `net user`. Sysmon process telemetry was used to reconstruct the chain from `wazuh-agent.exe` to `net.exe` and then `net1.exe`, confirming that the account enumeration occurred during the Wazuh assessment workflow.

**Disposition:** Benign / expected security-assessment activity.

[Read Investigation 02](investigations/02-account-discovery.md)

### 3. Controlled PowerShell execution from a temporary directory

A harmless test script was created under the Windows temporary directory and executed with PowerShell using `-ExecutionPolicy Bypass`. Wazuh generated a **Level 6** alert for suspicious PowerShell script execution and mapped the behavior to **T1059.001 - PowerShell**.

The detection was technically correct, while investigation context established that the action was an authorized lab test.

**Disposition:** True-positive detection of suspicious behavior / authorized benign lab activity.

[Read Investigation 03](investigations/03-controlled-powershell-execution.md)

## Key lessons

- Alert severity is a triage signal, not a final verdict.
- MITRE ATT&CK mappings describe observed behavior but do not establish malicious intent by themselves.
- Process ancestry and command-line context can turn an apparently suspicious alert into a defensible benign disposition.
- A true-positive detection can still represent authorized activity.
- Security tools themselves generate activity that may trigger detections and must be distinguished from unauthorized execution.

## Technologies

- Wazuh 4.14.x
- Sysmon
- Windows 11
- Ubuntu Server
- VirtualBox
- PowerShell
- Windows Event Logs
- MITRE ATT&CK

## Scope and safety

This repository documents a **controlled personal lab**. It contains no production credentials, private keys, API tokens, VM images, or raw sensitive logs. User-specific identifiers and unnecessary environment details are intentionally omitted or generalized.

## Status

**Completed.** The infrastructure is stable, Sysmon telemetry is flowing end-to-end, and three investigations have been documented. Future lab use is optional practice rather than required project completion.
