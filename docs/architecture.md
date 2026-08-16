# Lab Architecture

## Overview

The lab uses two primary virtual machines hosted in VirtualBox:

```text
┌───────────────────────────────┐
│ Windows 11 endpoint           │
│                               │
│ - Wazuh agent                 │
│ - Sysmon                      │
│ - Windows Event Logs          │
└───────────────┬───────────────┘
                │
                │ Private lab network
                ▼
┌───────────────────────────────┐
│ Ubuntu Wazuh server           │
│                               │
│ - Wazuh manager               │
│ - Wazuh indexer               │
│ - Wazuh dashboard             │
│ - Filebeat                    │
└───────────────────────────────┘
```

## Network design

The endpoint and Wazuh server communicate over an isolated VirtualBox host-only network. A separate NAT adapter provides Internet connectivity where needed without making the monitoring path dependent on the NAT network.

The design provides a simple separation between:

- lab-internal security telemetry;
- management traffic between the endpoint and Wazuh;
- external package/download access.

Exact private addresses are intentionally omitted from the public documentation because they are not relevant to reproducing the architecture.

## Endpoint telemetry

The Windows 11 endpoint runs the Wazuh agent and Sysmon.

Sysmon writes detailed endpoint telemetry to:

```text
Microsoft-Windows-Sysmon/Operational
```

The Wazuh agent is configured to collect that event channel in addition to standard Windows Application, Security, and System events.

The monitored endpoint therefore provides process-creation and related endpoint context that can include:

- executable image paths;
- command lines;
- parent/child process relationships;
- execution users;
- integrity levels;
- hashes;
- selected network and file activity, depending on the Sysmon configuration.

## Wazuh pipeline

```text
Endpoint activity
   ↓
Sysmon / Windows Event Log
   ↓
Wazuh agent
   ↓
Wazuh manager decoding and rules
   ↓
Wazuh indexer
   ↓
Wazuh dashboard / Threat Hunting
```

The lab was validated end-to-end by generating known endpoint actions, confirming corresponding Sysmon events locally, confirming Wazuh collection of the Sysmon channel, and then locating resulting Wazuh alerts in the dashboard.

## Preserved Wazuh capabilities

Sysmon integration was added without intentionally replacing the platform's other endpoint capabilities. The final working configuration retained:

- Security Configuration Assessment (SCA);
- File Integrity Monitoring (FIM);
- rootcheck;
- standard Windows event-channel collection.

## Safety boundary

This is a personal defensive-security lab. Controlled exercises use inert commands and scripts designed to produce observable telemetry without deploying malware or creating persistence.
