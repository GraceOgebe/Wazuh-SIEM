# Report 2: Agent Deployment and Sysmon Integration

## Cross-Platform Endpoint Monitoring

## My Proof of Work

I connected Windows and Ubuntu endpoints to Wazuh and added Sysmon telemetry. This work gave me the endpoint visibility I needed for detection, hunting, investigation, and response. The screenshots in this report show my completed configuration and validation results.

## Project Overview

I enrolled Windows 10 at `192.168.66.131` and Ubuntu at `192.168.66.132` with my Wazuh server at `192.168.66.128`. I collected identity, process, registry, and network evidence from Windows Security auditing and Sysmon. From Ubuntu, I collected SSH, authentication, process, privilege, and system evidence.

## Tools Used

| Tool | Purpose |
|---|---|
| Wazuh Windows agent | Forwarded Windows Security and Sysmon events |
| Wazuh Linux agent | Forwarded Ubuntu authentication and system events |
| Sysmon for Windows | Recorded process, network, file, and registry activity |
| Sysmon Modular | Supplied a structured Windows monitoring configuration |
| Linux Sysmon decoder | Improved Linux event parsing in Wazuh |
| Windows Event Viewer | Confirmed local event generation |
| PowerShell | Supported Windows validation and test activity |
| Linux command line | Supported agent setup, log review, and service checks |

## Skills I Demonstrated

- Cross-platform endpoint enrolment
- Windows Security log collection
- Sysmon deployment and validation
- Linux authentication monitoring
- Agent identity and health validation
- Field-level event analysis
- Configuration troubleshooting

## Telemetry Flow

```mermaid
flowchart TD
    A[Windows Security and Sysmon] --> C[Wazuh agent 001]
    B[Ubuntu auth and system logs] --> D[Wazuh agent 002]
    C --> E[Wazuh manager]
    D --> E
    E --> F[Wazuh indexer]
    F --> G[Dashboard search and investigation]
```

## Integration Record

| Area | Completed work |
|---|---|
| Owner | I configured and validated both Wazuh agents. |
| Endpoints | I monitored Windows 10 at `192.168.66.131` and Ubuntu at `192.168.66.132`. |
| Telemetry | I collected Windows Security and Sysmon events, plus Ubuntu authentication, SSH, system, and process evidence. |
| Period | I completed agent integration and validation during September 2026. |
| Network | Both endpoints used the isolated `192.168.66.0/24` network and sent data to Wazuh at `192.168.66.128`. |
| Validation | I checked agent status, recent activity, local services, and searchable events before detection testing. |

## Work I Completed

1. I connected the Windows endpoint to Wazuh.
2. I connected the Ubuntu endpoint to Wazuh.
3. I installed and configured Sysmon on Windows.
4. I collected Windows Security events.
5. I collected Ubuntu authentication and system events.
6. I verified each agent's identity, address, status, and recent activity.

## Endpoint Inventory

| Agent ID | Endpoint | IP address | Main telemetry |
|---|---|---|---|
| `001` | Windows 10 | `192.168.66.131` | Windows Security and Sysmon |
| `002` | Ubuntu | `192.168.66.132` | SSH, authentication, system, and Linux process activity |

![Connected Windows and Ubuntu agents](../images/02-connected-agents.png)

## Windows Agent Integration

I connected the Windows endpoint to the Wazuh manager. I used Windows Security auditing to collect identity and authentication evidence, including:

- Account creation
- Account deletion
- Account status changes
- Group membership changes
- Successful and failed logons
- Subject and target security identifiers
- Logon type and source address context

### Sysmon Integration

I used Sysmon to add detailed endpoint evidence beyond the standard Windows Security log. The fields I collected supported investigations of:

- Process creation
- Parent-child process relationships
- Command-line arguments
- Executable paths
- User context
- File hashes
- Registry modifications
- Network connections
- Destination addresses and ports
- UTC timestamps

![Windows event search in Wazuh](../images/03-windows-event-search.png)

### Windows Investigation Method

For each Windows event, the review process checked:

1. Agent ID and endpoint name
2. Windows Event ID or Sysmon Event ID
3. Subject and target accounts
4. Security identifier values
5. Process image and parent image
6. Command line and working directory
7. Source or destination address
8. Event time and surrounding activity
9. Expected lab action or unexplained behaviour

## Ubuntu Agent Integration

I configured the Ubuntu agent to collect evidence from authentication and system logs. This supported my investigation of:

- Failed SSH logons
- Successful SSH logons
- Source IP and source port review
- Target username review
- `sudo` and privilege-related activity
- Process and command activity
- Wazuh Active Response logging

A Linux Sysmon decoder was added to improve parsing and field extraction for Linux process events.

### Linux Investigation Method

For each Linux event, the review process checked:

1. Agent ID and endpoint address
2. Source IP and source port
3. Target account
4. Authentication result
5. Attempt frequency
6. Commands or processes after authentication
7. Privilege context
8. Expected lab activity or unexplained behaviour

## Agent Configuration Validation

The endpoint services were checked after configuration changes.

Ubuntu agent checks included:

```bash
sudo systemctl restart wazuh-agent
sudo systemctl status wazuh-agent
```

The Windows service, Sysmon service, and recent events were checked before later simulations.

## Data Sources

| Source | Platform | Key fields | SOC value |
|---|---|---|---|
| Windows Security | Windows | `eventID`, `targetUserName`, `subjectUserName`, `targetSid`, `ipAddress`, `logonType` | Account and authentication investigation |
| Sysmon Operational | Windows | `image`, `commandLine`, `parentImage`, `user`, `hashes`, `destinationIp`, `utcTime` | Process, registry, and network investigation |
| Authentication logs | Ubuntu | `srcip`, `srcport`, `dstuser`, `program_name`, `full_log` | SSH and identity investigation |
| Linux system logs | Ubuntu | `agent.id`, `location`, `decoder.name`, `rule.id` | System and process context |
| Agent metadata | Wazuh | `agent.id`, `agent.name`, `agent.ip` | Endpoint attribution |

## Validation Results

| Requirement | Validation | Outcome |
|---|---|---|
| Windows agent connected | Agent shown as active | Achieved |
| Ubuntu agent connected | Agent shown as active | Achieved |
| Windows Security data searchable | Event searches returned Windows records | Achieved |
| Sysmon data searchable | Process and command evidence returned | Achieved |
| Ubuntu authentication visible | SSH events appeared in Wazuh | Achieved |
| Agent identities remain consistent | IDs `001` and `002` mapped to the planned systems | Achieved |

## Challenges and Resolutions

| Challenge | Cause | Resolution |
|---|---|---|
| Ubuntu agent failed to start | Invalid XML closing tags and extra content in `ossec.conf` | Corrected the XML and restarted the service |
| Some Linux fields required clearer parsing | Raw Linux events did not expose all useful fields | Added the Linux Sysmon decoder |
| Windows events contained many routine records | High event volume reduced focus | Filtered by event ID, process name, user, and time |
| SID values required interpretation | Raw identifiers did not immediately show account context | Resolved the SID and reviewed related fields |

## Evidence

### Windows Account and Logon Telemetry

Wazuh captured account creation, successful logon, local-group membership, and account deletion events from the Windows agent.

![Windows account creation event 4720](../images/30-wazuh-event-4720-account-created.png)

![Windows successful logon event 4624](../images/31-wazuh-event-4624-successful-logon.png)

![Windows local-group membership event 4732](../images/32-wazuh-event-4732-group-membership.png)

### Account Deletion

![Windows account deletion event 4726](../images/04-account-deletion-event-4726.png)

### Event Interpretation References

I used Windows Event ID and logon-type references during analysis. These reference screenshots do not prove Wazuh ingestion.

![Event ID 4720 reference](../images/05-event-4720-reference.png)

![Windows logon-type reference](../images/06-logon-type-reference.png)

### SID Investigation

![Windows SID investigation](../images/07-sid-investigation.png)

## What I Achieved

- Two active Wazuh agents
- Windows Security log collection
- Windows Sysmon process, registry, and network visibility
- Ubuntu SSH and authentication visibility
- Reliable endpoint attribution using agent metadata
- Searchable evidence for later detection and investigation reports
- Verified Event IDs `4720`, `4624`, `4732`, and `4726` in Wazuh

## Recommendations Based on My Findings

1. Review agent health daily.
2. Alert when an agent becomes disconnected or stops sending events.
3. Keep Sysmon rules focused on high-value activity.
4. Test event collection after configuration or agent upgrades.
5. Record the expected hostname, IP address, agent ID, and operating system for each endpoint.
6. Protect agent enrolment keys and manager communication paths.
7. Confirm local event generation before troubleshooting Wazuh searches.

## What I Learned

1. Agent health and event ingestion require separate checks.
2. Sysmon supplies command-line and parent-process context needed for stronger investigations.
3. Linux and Windows expose different fields, but both support identity, process, network, and timeline analysis.
4. Configuration validation prevents avoidable monitoring gaps.
5. Event IDs require user, host, time, process, and source context before classification.

## Complete Screenshot Evidence

The following images preserve the original agent, Sysmon, decoder, service, and endpoint-configuration evidence in sequence.

| Evidence 011 | Evidence 012 |
|---|---|
| ![Evidence 011](../evidence/02-agents-sysmon/evidence-011.png) | ![Evidence 012](../evidence/02-agents-sysmon/evidence-012.png) |

| Evidence 013 | Evidence 014 |
|---|---|
| ![Evidence 013](../evidence/02-agents-sysmon/evidence-013.png) | ![Evidence 014](../evidence/02-agents-sysmon/evidence-014.png) |

| Evidence 015 | Evidence 016 |
|---|---|
| ![Evidence 015](../evidence/02-agents-sysmon/evidence-015.png) | ![Evidence 016](../evidence/02-agents-sysmon/evidence-016.png) |

| Evidence 017 | Evidence 018 |
|---|---|
| ![Evidence 017](../evidence/02-agents-sysmon/evidence-017.png) | ![Evidence 018](../evidence/02-agents-sysmon/evidence-018.png) |

| Evidence 019 | Evidence 020 |
|---|---|
| ![Evidence 019](../evidence/02-agents-sysmon/evidence-019.png) | ![Evidence 020](../evidence/02-agents-sysmon/evidence-020.png) |

| Evidence 054 | Evidence 055 |
|---|---|
| ![Evidence 054](../evidence/02-agents-sysmon/evidence-054.png) | ![Evidence 055](../evidence/02-agents-sysmon/evidence-055.png) |

| Evidence 056 | Evidence 057 |
|---|---|
| ![Evidence 056](../evidence/02-agents-sysmon/evidence-056.png) | ![Evidence 057](../evidence/02-agents-sysmon/evidence-057.png) |

| Evidence 058 | Evidence 059 |
|---|---|
| ![Evidence 058](../evidence/02-agents-sysmon/evidence-058.png) | ![Evidence 059](../evidence/02-agents-sysmon/evidence-059.png) |

| Evidence 060 | Evidence 061 |
|---|---|
| ![Evidence 060](../evidence/02-agents-sysmon/evidence-060.png) | ![Evidence 061](../evidence/02-agents-sysmon/evidence-061.png) |

| Evidence 062 | Evidence 063 |
|---|---|
| ![Evidence 062](../evidence/02-agents-sysmon/evidence-062.png) | ![Evidence 063](../evidence/02-agents-sysmon/evidence-063.png) |

| Evidence 064 | Evidence 065 |
|---|---|
| ![Evidence 064](../evidence/02-agents-sysmon/evidence-064.png) | ![Evidence 065](../evidence/02-agents-sysmon/evidence-065.png) |

| Evidence 066 | Evidence 067 |
|---|---|
| ![Evidence 066](../evidence/02-agents-sysmon/evidence-066.png) | ![Evidence 067](../evidence/02-agents-sysmon/evidence-067.png) |

| Evidence 068 | Evidence 069 |
|---|---|
| ![Evidence 068](../evidence/02-agents-sysmon/evidence-068.png) | ![Evidence 069](../evidence/02-agents-sysmon/evidence-069.png) |



## Next Report

[Report 3: Telemetry Generation and Security Investigation](03-Telemetry-Generation-and-Security-Investigation.md)
