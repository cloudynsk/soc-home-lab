# Investigation 02: Account Discovery Using net.exe and net1.exe

## Status

**Closed - Benign / Expected Security-Assessment Activity**

## Alert

- **Wazuh rule:** 92031
- **Description:** Discovery activity executed
- **Level:** 3
- **MITRE ATT&CK:** T1087 - Account Discovery
- **Tactic:** Discovery
- **Telemetry:** Sysmon Event ID 1, Process Create

## Observed process chain

```text
wazuh-agent.exe
  -> net.exe "net user"
     -> net1.exe "net1 user"
```

The child event showed:

```text
Image:
C:\Windows\SysWOW64\net1.exe

Command line:
C:\WINDOWS\system32\net1 user

Parent image:
C:\Windows\SysWOW64\net.exe

Parent command line:
net user

User:
NT AUTHORITY\SYSTEM

Current directory:
C:\Program Files (x86)\ossec-agent\
```

A separate Sysmon Process Create event for the matching `net.exe` PID was then inspected to verify the missing ancestry. Its parent was the Wazuh agent.

## Initial concern

`net user` enumerates user accounts and is legitimate account-discovery behavior. In a production environment, the same command can be used by administrators, security products, scripts, or an attacker conducting discovery after execution.

The behavior therefore required contextual attribution.

## Analysis

The matching process records established that the command originated from the Wazuh agent and ran under the SYSTEM account from the Wazuh agent directory.

The timing aligned with the Wazuh Security Configuration Assessment workflow. This converted the initial hypothesis from unknown account-enumeration activity to verified security-tool behavior.

The Sysmon configuration attached a T1018 Remote System Discovery tag to the process, while Wazuh classified the resulting behavior as T1087 Account Discovery. The command line itself, `net user`, supported the Wazuh T1087 interpretation more directly. This demonstrated why ATT&CK metadata should be treated as investigative context rather than unquestioned ground truth.

## Disposition

**Benign / expected Wazuh Security Configuration Assessment activity.**

- Escalation: Not required.
- Remediation: None required.

## Analyst takeaway

Discovery commands are not malicious by themselves. The final disposition depended on process ancestry and operational context. Following the exact PID relationship was necessary to confirm attribution instead of assuming that nearby Wazuh activity caused the alert.
