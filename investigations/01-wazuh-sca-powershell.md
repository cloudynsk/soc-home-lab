# Investigation 01: PowerShell Temporary-File Deletion During Wazuh Assessment

## Status

**Closed - Benign / Expected Security-Tool Activity**

## Alert

- **Wazuh rule:** 92021
- **Description:** PowerShell was used to delete files or directories
- **Level:** 3
- **MITRE ATT&CK:** T1070.004 - File Deletion
- **Tactic:** Defense Evasion
- **Telemetry:** Sysmon Event ID 1, Process Create

## Observed process

```text
Image:
C:\Windows\SysWOW64\WindowsPowerShell\v1.0\powershell.exe

Parent:
C:\Program Files (x86)\ossec-agent\wazuh-agent.exe

User:
NT AUTHORITY\SYSTEM

Current directory:
C:\Program Files (x86)\ossec-agent\
```

The command performed three relevant actions:

```text
1. Export the local Windows security policy to a temporary secpol.cfg file.
2. Read the temporary file and inspect the LockoutBadCount setting.
3. Delete the temporary secpol.cfg file.
```

## Initial concern

PowerShell file deletion can be associated with evidence removal or defense-evasion behavior. The rule description and MITRE mapping therefore warranted inspection rather than immediate closure.

## Analysis

Process ancestry showed that PowerShell was launched directly by `wazuh-agent.exe` under the SYSTEM account. The working directory was the Wazuh agent installation directory.

The command content was consistent with configuration assessment: it exported the local security policy, extracted the account-lockout threshold, and removed the temporary export afterward.

The event occurred during the same period in which the Wazuh Security Configuration Assessment module started and loaded its Windows 11 CIS assessment policy.

The deletion behavior was therefore real, but its purpose was legitimate cleanup performed by the monitoring platform itself.

## Disposition

**Benign / expected Wazuh security-assessment activity.**

- Escalation: Not required.
- Remediation: None required.

## Analyst takeaway

The alert could not be classified safely from the rule description alone. Process ancestry, execution account, command-line content, working directory, and timeline context were necessary to distinguish legitimate security-tool activity from potentially malicious file deletion.
