# Report 3: Telemetry Generation and Security Investigation

## Detection Validation and Controlled Threat Simulation

## My Proof of Work

I generated, collected, and analysed Windows and Linux security telemetry in a controlled environment. I also completed authorised C2 simulations with Metasploit against my Windows and Ubuntu lab agents. This report records my actions, findings, investigation decisions, and supporting evidence.

## Project Overview

In this phase, I moved my lab from installation into active investigation. I generated account, authentication, process, network, registry, file-transfer, and shell activity on monitored systems. I then searched Wazuh and linked individual records into complete activity sequences.

## Tools Used

| Tool | Purpose |
|---|---|
| Wazuh Discover | Searched and expanded endpoint events |
| Windows Security auditing | Recorded account and authentication activity |
| Sysmon | Recorded process, network, and registry behaviour |
| Kali Linux | Hosted authorised simulation tools |
| `msfvenom` | Generated Windows and Linux lab payloads |
| `msfconsole` | Operated handlers and Meterpreter sessions |
| Meterpreter | Generated controlled post-compromise activity |
| `wget` and Python web server | Transferred the Linux test file |

## Skills I Demonstrated

- Event generation and validation
- Windows account investigation
- Authentication analysis
- Process and command-line analysis
- Registry persistence analysis
- Controlled C2 simulation
- Windows and Linux post-compromise investigation
- MITRE ATT&CK mapping
- Timeline reconstruction

## Attack and Telemetry Flow

```mermaid
flowchart TD
    A[Generate controlled payload] --> B[Transfer to lab endpoint]
    B --> C[Execute on Windows or Ubuntu]
    C --> D[Reverse connection to Kali]
    D --> E[Run discovery and persistence actions]
    E --> F[Endpoint creates telemetry]
    F --> G[Wazuh collects and indexes events]
    G --> H[Analyst correlates full sequence]
```

## Scope

I completed and documented:

- Windows account creation and deletion
- Successful Windows logons
- SID investigation
- Process and command-line activity
- Registry Run-key persistence
- SSH authentication failures
- Windows and Ubuntu Meterpreter sessions
- File transfer and reverse TCP callbacks
- Correlation of related endpoint events

## How I Generated and Investigated the Telemetry

For each scenario, I followed the same evidence-driven process:

1. I defined the action and expected evidence.
2. I recorded the source, target, user, and time.
3. I generated the action inside the isolated lab.
4. I confirmed that the endpoint produced a log.
5. I searched Wazuh by agent, event ID, process, user, or address.
6. I expanded the raw event and reviewed its fields.
7. I correlated the surrounding activity.
8. I recorded the 5Ws and How.
9. I classified the event as expected lab activity, suspicious activity, or failed test activity.

## Investigation 1: Windows Account Activity

### Findings

I created local accounts during controlled Windows testing. Wazuh recorded Event ID `4720` for creation of `student1` by `Grace Ogebe` on `DESKTOP-CKCPUSR`. Wazuh also recorded Event ID `4726` for deletion of `student1` and exposed the target account, subject account, SID, host, and timestamp context.

![Wazuh Event ID 4720 account creation](../images/30-wazuh-event-4720-account-created.png)

Wazuh recorded Event ID `4732` when `student1` was added to the local Administrators group.

![Wazuh Event ID 4732 local-group membership](../images/32-wazuh-event-4732-group-membership.png)

![Controlled account creation and group-change commands](../images/25-windows-post-compromise-actions.png)

![Windows account deletion event](../images/04-account-deletion-event-4726.png)

### Investigation Summary

The Windows endpoint generated a complete `student1` account-lifecycle sequence: creation, addition to the local Administrators group, and deletion. This was authorised lab activity. In production, the same sequence would require urgent review because it combines account creation, privilege assignment, and removal.

### 5Ws and How

| Question | Finding |
|---|---|
| **Who** | The lab analyst generated the actions on Windows agent `001`. The event fields identified the subject account and affected local account. |
| **What** | Wazuh recorded creation of `student1`, addition of that account to the local Administrators group, and later deletion of the account. |
| **When** | Wazuh displayed account creation at 9 September 2026 `11:33:19.932` and the Administrators-group addition at `11:51:49.667`. |
| **Where** | The activity occurred on Windows 10 at `192.168.66.131`. |
| **Why** | The test measured Wazuh visibility into local account lifecycle changes and prepared the lab for later account-manipulation detection. |
| **How** | I generated the account actions through the controlled Windows shell, then filtered Wazuh for Events `4720`, `4732`, and `4726` and reviewed the subject, target, SID, host, and group fields. |

### Key Questions Answered

| Question | Answer |
|---|---|
| Hosts involved | Windows agent `001`, `192.168.66.131` |
| Accounts changed | `student1` was created, added to Administrators, and deleted |
| File deleted | No. Event `4726` records account deletion, not file deletion |
| SSH abuse | Not part of this investigation |

### Recommendations

1. Correlate Events `4720`, `4732`, and `4726` by target SID, host, and initiating account.
2. Alert when a new account receives privileged membership shortly after creation.
3. Verify privileged account changes against approved requests.
4. Disable unauthorised accounts and review actions performed during their lifetime.

### Analyst Assessment

Account creation or deletion is not automatically malicious. I assessed the account name, initiating account, host, time, nearby process evidence, and surrounding remote-session, privilege-change, and persistence activity before assigning a disposition.

## Investigation 2: Windows Logon-Type Analysis

### Findings

I investigated Event ID `4624` in Wazuh. The retained event shows a successful service logon on `Windows-10` with Logon Type `5`. I used the event to demonstrate logon-type interpretation, but I did not classify it as unauthorised Valid Accounts activity.

![Wazuh Event ID 4624 successful service logon](../images/31-wazuh-event-4624-successful-logon.png)

![Windows logon-type reference](../images/06-logon-type-reference.png)

### Investigation Summary

Wazuh recorded a successful service logon under Event ID `4624`, Logon Type `5`. The evidence shows the machine account as the subject and `SYSTEM` as the new-logon account. I treated it as authentication telemetry requiring context, not as proof of account misuse.

### 5Ws and How

| Question | Finding |
|---|---|
| **Who** | The event shows subject account `DESKTOP-CKCPUSR$` and new-logon account `SYSTEM`. |
| **What** | Wazuh recorded a successful Event ID `4624` service logon with Logon Type `5`. |
| **When** | Wazuh displayed the event at 9 September 2026 `11:45:05.309`. |
| **Where** | The analysis applied to Windows agent `001`, `192.168.66.131`. |
| **Why** | I needed a repeatable method for distinguishing interactive, network, service, unlock, and remote logons. |
| **How** | I filtered Wazuh for Event ID `4624`, expanded the record, identified Logon Type `5`, and compared it with the logon-type reference. |

### Key Questions Answered

| Question | Answer |
|---|---|
| Hosts involved | Windows agent `001`, `192.168.66.131` |
| Accounts changed | None. This was a logon event |
| File deleted | No |
| SSH abuse | No. This was Windows service-logon telemetry |

### Recommendations

1. Baseline expected service accounts and Logon Type `5` activity.
2. Correlate unusual service logons with service installation, process, and network events.
3. Escalate only when the account, host, time, or supporting behaviour is unexpected.

### Logon Type Context

| Logon type | Meaning | Review focus |
|---|---|---|
| `2` | Interactive | Local user access |
| `3` | Network | Remote resource access |
| `5` | Service | Service account activity |
| `7` | Unlock | Existing session unlocked |
| `10` | Remote Interactive | Remote Desktop activity |

## Investigation 3: SID and Identity Resolution

### Findings

Some events displayed a SID where the account name required more context. The SID investigation connected the identifier to its related account and event.

![SID investigation](../images/07-sid-investigation.png)

### Investigation Summary

I used the SID to connect account records referring to the same Windows identity. This supported correlation of the `student1` lifecycle without relying only on an account name that could be changed or reused.

### 5Ws and How

| Question | Finding |
|---|---|
| **Who** | The subject and target SID fields identified the initiating and affected identities. |
| **What** | I correlated related Windows account records by SID. |
| **When** | The analysis followed the account events recorded on 9 September 2026. |
| **Where** | Windows agent `001`, `192.168.66.131`. |
| **Why** | SID correlation gives stronger identity continuity than display names alone. |
| **How** | I searched the SID in Wazuh and compared account, event, host, and target fields. |

### Key Questions Answered

| Question | Answer |
|---|---|
| Hosts involved | Windows agent `001` |
| Accounts changed | `student1` |
| File deleted | No |
| SSH abuse | No |

### Recommendations

1. Include subject and target SIDs in account-change alerts.
2. Correlate renamed or reused accounts by SID and host.
3. Preserve SID-to-account mappings in the case record.

### Investigation Value

- Distinguishes built-in, local, domain, and service identities
- Links account events across name changes
- Helps identify unexpected administrative activity
- Reduces the risk of relying only on a display name

## Controlled Command-and-Control Simulation

I used `msfvenom`, `msfconsole`, multi/handler, and Meterpreter to generate known command-and-control-style behaviour. My goal was to observe the activity from both sides:

- Kali recorded the action taken by the simulation operator.
- Windows and Ubuntu generated endpoint evidence.
- Wazuh recorded what each monitored endpoint observed.

The simulation used only the private lab systems. No external target was involved.

## Investigation 4: Windows C2 Simulation

### Findings

Kali Linux generated `grace.exe` with `msfvenom`. The Windows endpoint executed the file and connected to a Metasploit handler on Kali. Meterpreter supported controlled process, identity, network, account, group, and registry actions.

![Windows payload generated with msfvenom](../images/22-msfvenom-windows-payload.png)

![Windows Meterpreter session](../images/23-msfconsole-windows-session.png)

### Investigation Summary

I executed `grace.exe` inside the isolated lab and received a reverse Meterpreter session on Kali. I then performed controlled discovery, account, group, privilege-related, and persistence-style actions. Endpoint and Metasploit evidence proves the session and commands. Wazuh directly confirms selected activity such as `whoami`, but I do not claim a separate retained Wazuh event for every action.

### 5Ws and How

| Question | Finding |
|---|---|
| **Who** | Kali `192.168.66.133` received the connection from Windows `192.168.66.131`. The Windows session ran as `DESKTOP-QKCPUSR\Grace Ogebe`. |
| **What** | `grace.exe` created a reverse TCP session. The controlled session ran discovery commands, created a test account, changed group membership, and added a registry Run-key value. |
| **When** | The first callback occurred on 11 September 2026 at `13:21:41 -0400`. Extended testing continued on 12 September 2026. |
| **Where** | The file ran from `C:\Users\Grace Ogebe\Downloads\grace.exe` and connected to `192.168.66.133:4444`. |
| **Why** | The authorised simulation tested Wazuh visibility across a realistic sequence of execution, callback, discovery, account manipulation, privilege-related activity, and persistence. |
| **How** | `msfvenom` generated the payload and `msfconsole` hosted the handler. The endpoint and Metasploit screenshots prove execution, shell activity, account changes, persistence commands, and the reverse session. The retained Wazuh screenshot directly confirms `whoami` telemetry. It does not independently prove every expected registry and network event. |

### Key Questions Answered

| Question | Answer |
|---|---|
| Hosts involved | Windows `192.168.66.131` and Kali `192.168.66.133` |
| Accounts changed | Controlled `attacker` account created and added to Administrators through command evidence |
| File deleted | No confirmed file deletion |
| SSH abuse | No. The channel was reverse TCP Meterpreter |

### Recommendations

1. Isolate the endpoint if equivalent activity is unexpected.
2. Quarantine the payload and collect its hash, process tree, command line, and network destination.
3. Review created accounts, privileged groups, and persistence locations.
4. Hunt across endpoints for the same executable, callback, commands, and Run-key value.
5. Rebuild the host if post-compromise scope cannot be established confidently.

### Commands and Behaviours Observed

| Action | Expected evidence | ATT&CK mapping |
|---|---|---|
| Execute `grace.exe` | Process image, path, user, hash, and parent process | Execution |
| Reverse TCP connection | Destination `192.168.66.133:4444` | `T1095` |
| Run `whoami` | Process and command line | `T1033` |
| Run `ipconfig` | Process and command line | `T1016` |
| Start PowerShell | PowerShell process evidence | `T1059.001` |
| Create local `attacker` account | Event ID `4720` | `T1136.001` |
| Add account to Administrators | Event ID `4732` | `T1098` |
| Add `CalcPersist` Run key | Registry modification | `T1547.001` |
| Test UAC bypass | Process and registry evidence | `T1548.002` |

![Windows process listing and shell](../images/24-meterpreter-windows-process-shell.png)

![Windows account and persistence activity](../images/25-windows-post-compromise-actions.png)

### Local Privilege and Persistence Assessment

I ran Metasploit's local exploit suggester against Windows session `1`. It tested 352 of 2,654 checks and returned several results marked potentially vulnerable. These results were suggestions, not proof of successful exploitation. I separately tested `bypassuac_fodhelper`. Metasploit confirmed UAC was enabled, identified the session as part of Administrators, prepared the registry and stager values, launched `fodhelper.exe`, and created another Meterpreter session.

![Windows local exploit suggester results, part 1](../images/46-windows-local-exploit-suggester-1.png)

![Windows local exploit suggester results, part 2](../images/47-windows-local-exploit-suggester-2.png)

![Windows handler and controlled UAC test](../images/48-windows-handler-and-uac-test.png)

The Windows command evidence also records account creation, Administrators-group changes, command-syntax corrections, and successful creation of the `CalcPersist` Run-key value.

![Windows account and Run-key commands](../images/50-windows-account-and-run-key-commands.png)

### Wazuh Evidence

A Wazuh search for `whoami` returned 25 events on 12 September 2026. The expanded event identified:

- Agent `001`
- Agent address `192.168.66.131`
- Endpoint name `Windows-10`
- Image `C:\Windows\SysWOW64\whoami.exe`
- Original filename `whoami.exe`
- Process and parent-process identifiers
- Command and working-directory context

![Wazuh whoami telemetry](../images/28-wazuh-whoami-telemetry.png)

![Additional Wazuh whoami event view](../images/49-wazuh-windows-whoami-event.png)

## Investigation 5: Ubuntu C2 Simulation

### Findings

Kali generated a Linux x64 payload named `ogebe.elf`. Ubuntu downloaded the file from a temporary lab web server, marked it executable, and ran it. The executable created a reverse TCP Meterpreter session to Kali on port `4445`.

![Ubuntu payload transfer and execution](../images/26-linux-payload-transfer-execution.png)

![Ubuntu Meterpreter session](../images/27-msfconsole-linux-session.png)

### Investigation Summary

Ubuntu downloaded `ogebe.elf` from Kali, changed its permissions, executed it, and established a reverse Meterpreter session. Endpoint and Metasploit evidence confirms the transfer, execution, callback, shell access, and discovery. Wazuh retained Linux process and session telemetry, but this report does not claim a distinct alert for every action.

### 5Ws and How

| Question | Finding |
|---|---|
| **Who** | Kali `192.168.66.133` interacted with Ubuntu `192.168.66.132`. The session ran as `graceogebe`. |
| **What** | Ubuntu downloaded and ran `ogebe.elf`. The process opened a reverse session and generated user, group, account-file, OS, and shell discovery activity. |
| **When** | The file was downloaded on 12 September 2026 at `10:56:37` in the endpoint's displayed time. Metasploit recorded session `3` at `07:02:16 -0400`. |
| **Where** | The file was retrieved from `192.168.66.133:9999` and connected to `192.168.66.133:4445`. The activity occurred on Ubuntu agent `002`. |
| **Why** | The test measured cross-platform visibility into file transfer, execution, reverse connection, user discovery, account discovery, and shell activity. |
| **How** | `msfvenom` generated the ELF. Ubuntu used `wget` and `chmod` before execution, and `msfconsole` received the callback. The retained Ubuntu and Metasploit screenshots prove these actions. A clear Wazuh event for every Linux step was not retained. |

![Wazuh Linux whoami process telemetry](../images/35-wazuh-linux-whoami-process.png)

![Wazuh Linux session telemetry](../images/36-wazuh-linux-session-telemetry.png)

### Retained Ubuntu Command Evidence

The additional evidence records the full endpoint sequence: `wget` downloaded `ogebe.elf` from Kali, `chmod` made it executable, the handler received session `3`, and `getuid` identified `graceogebe`. The shell then recorded operating-system, user, UID, group, and account-file discovery.

![Ubuntu payload download and execution](../images/51-ubuntu-payload-download-and-execution.png)

![Ubuntu Meterpreter session details](../images/52-ubuntu-meterpreter-session-details.png)

![Ubuntu operating-system and user discovery](../images/53-ubuntu-os-and-user-discovery.png)

![Ubuntu account and group discovery](../images/54-ubuntu-account-and-group-discovery.png)

### Key Questions Answered

| Question | Answer |
|---|---|
| Hosts involved | Ubuntu `192.168.66.132` and Kali `192.168.66.133` |
| Accounts changed | None established. The session ran as `graceogebe` |
| File deleted | No confirmed file deletion |
| SSH abuse | No. This was file transfer, execution, and reverse TCP activity |

### Recommendations

1. Isolate the endpoint if the payload or callback is unauthorised.
2. Quarantine the ELF, calculate its hashes, and hunt for matching files and executions.
3. Review persistence, scheduled tasks, services, account changes, and outbound connections.
4. Block the callback address and port after validation.
5. Rebuild the endpoint if its integrity cannot be established.

## Investigation 6: SSH Authentication Failures

### Findings

The retained source-event screenshot shows an earlier failed SSH attempt from Windows `192.168.66.131` against the non-existent Ubuntu user `fakeuser`. Wazuh recorded the event under rule `5710`. This image demonstrates the source fields used by the later correlation rule, but it is not the 24 September custom rule `100101` alert. The later `graceogebe` alert and response appear in Report 4.

![SSH authentication source event](../images/08-ssh-authentication-source-event.png)

The retained evidence also shows an accepted password for `graceogebe` from `192.168.66.131`, a session-start event, and an interactive SSH session. This proves successful authentication during the authorised test, not hostile access.

![Wazuh successful SSH authentication](../images/33-wazuh-successful-ssh-authentication.png)

![Successful SSH session from Windows](../images/34-windows-successful-ssh-session.png)

### Investigation Summary

The SSH tests produced failures against `fakeuser` and later successful authentication for `graceogebe`. Wazuh exposed the source address, target account, source port, agent, rule, and session context. Report 4 covers the later custom-rule `100101` correlation, Tines triage, approval, and response.

### 5Ws and How

| Question | Finding |
|---|---|
| **Who** | Source `192.168.66.131` targeted non-existent user `fakeuser` on Ubuntu agent `002`. |
| **What** | Wazuh recorded an SSH attempt against a non-existent user under rule `5710`. |
| **When** | The selected Wazuh screenshot displays 10 September 2026. |
| **Where** | The activity moved from the Windows lab system to Ubuntu at `192.168.66.132`. |
| **Why** | The test measured authentication visibility and exposed the fields required for later brute-force correlation. |
| **How** | The SSH attempt generated an Ubuntu authentication record containing the source IP, source port, target user, agent, and rule information. |

### Key Questions Answered

| Question | Answer |
|---|---|
| Hosts involved | Windows `192.168.66.131` and Ubuntu `192.168.66.132`, agent `002` |
| Accounts changed | None. `fakeuser` was non-existent and `graceogebe` was the later authentication target |
| File deleted | No |
| SSH abuse | Repeated failures were observed as a controlled brute-force simulation. A later successful login was also recorded |

### Recommendations

1. Correlate failures by source IP, target account, and target host.
2. Escalate when a successful login follows repeated failures, especially outside working hours.
3. Prefer key-based authentication and restrict SSH to approved administration networks.
4. Review commands, processes, file changes, and outbound connections from successful sessions.

## Windows Simulation Timeline

| Date and time | Event |
|---|---|
| 11 Sep 2026, 12:40 local lab time | `grace.exe` generated on Kali |
| 11 Sep 2026, 13:21:41 -0400 | Windows Meterpreter session opened |
| 12 Sep 2026, 06:31:00 -0400 | Extended Windows session opened |
| 12 Sep 2026, 06:31:36 -0400 | Controlled UAC-bypass session opened |
| 12 Sep 2026 | Wazuh returned related `whoami` telemetry |

## Ubuntu Simulation Timeline

| Date and time | Event |
|---|---|
| 12 Sep 2026, 10:56:37 endpoint time | Ubuntu downloaded `ogebe.elf` |
| 12 Sep 2026, 07:02:16 -0400 | Ubuntu Meterpreter session opened |

## SSH Detection Timeline

| Date and time | Event |
|---|---|
| 10 Sep 2026, displayed Wazuh time | Wazuh recorded an SSH attempt against `fakeuser` under rule `5710` |
| 24 Sep 2026, approximately 16:28 UTC | The later `graceogebe` sequence generated custom rule `100101`, documented in Report 4 |

## Results

| Objective | Outcome |
|---|---|
| Create a controlled Windows account | Achieved. Wazuh Event ID `4720` captured creation of `student1`. |
| Investigate a successful Windows logon | Achieved. Wazuh Event ID `4624` showed a Logon Type `5` service logon. |
| Resolve identity using SID context | Achieved |
| Generate Windows process and network activity | Achieved |
| Generate a registry persistence action | Achieved through successful command evidence. Matching Wazuh registry event not isolated. |
| Generate Linux transfer and execution activity | Achieved |
| Establish controlled Windows and Linux callbacks | Achieved |
| Capture SSH authentication failures | Achieved |
| Search and correlate selected evidence in Wazuh | Achieved for retained account creation, account deletion, group membership, successful logon, SSH, FIM, Active Response, and `whoami` evidence. |

## Threat Hunting Queries

```text
agent.id:001 AND data.win.eventdata.image:*whoami.exe
```

```text
agent.id:001 AND data.win.system.eventID:(4720 OR 4726 OR 4732)
```

```text
agent.id:001 AND data.win.eventdata.destinationIp:192.168.66.133
```

```text
agent.id:001 AND data.win.eventdata.targetObject:*\\CurrentVersion\\Run*
```

```text
agent.id:002 AND (full_log:*wget* OR full_log:*ogebe.elf*)
```

```text
agent.id:002 AND rule.groups:sshd
```

## Recommendations Based on My Findings

1. Alert on new executable files followed by outbound connections.
2. Correlate discovery commands with unusual parent processes.
3. Monitor local account creation and administrator-group changes.
4. Alert on registry Run-key modifications outside approved software changes.
5. Record file hashes for every suspicious executable.
6. Restrict unnecessary outbound connections from endpoints.
7. Search for the same destination address, hash, user, and command across all agents.
8. Remove test accounts, payloads, listeners, and persistence values after validation.

## What I Learned

1. One event rarely explains the full incident.
2. I learned to analyse process, network, identity, account, and registry evidence together.
3. Failed commands still show intent and belong in the timeline.
4. Source timestamps require timezone context.
5. I learned to check Wazuh visibility at raw-event and alert levels.
6. A known lab action still requires the same evidence-based investigation method used for a live alert.

## Complete Screenshot Evidence

The following images preserve the original account, authentication, process, registry, network, payload, Meterpreter, and C2 investigation evidence in sequence.

| Evidence 021 | Evidence 022 |
|---|---|
| ![Evidence 021](../evidence/03-telemetry-c2/evidence-021.png) | ![Evidence 022](../evidence/03-telemetry-c2/evidence-022.png) |

| Evidence 023 | Evidence 024 |
|---|---|
| ![Evidence 023](../evidence/03-telemetry-c2/evidence-023.png) | ![Evidence 024](../evidence/03-telemetry-c2/evidence-024.png) |

| Evidence 025 | Evidence 026 |
|---|---|
| ![Evidence 025](../evidence/03-telemetry-c2/evidence-025.png) | ![Evidence 026](../evidence/03-telemetry-c2/evidence-026.png) |

| Evidence 027 | Evidence 028 |
|---|---|
| ![Evidence 027](../evidence/03-telemetry-c2/evidence-027.png) | ![Evidence 028](../evidence/03-telemetry-c2/evidence-028.png) |

| Evidence 029 | Evidence 030 |
|---|---|
| ![Evidence 029](../evidence/03-telemetry-c2/evidence-029.png) | ![Evidence 030](../evidence/03-telemetry-c2/evidence-030.png) |

| Evidence 031 | Evidence 032 |
|---|---|
| ![Evidence 031](../evidence/03-telemetry-c2/evidence-031.png) | ![Evidence 032](../evidence/03-telemetry-c2/evidence-032.png) |

| Evidence 033 | Evidence 034 |
|---|---|
| ![Evidence 033](../evidence/03-telemetry-c2/evidence-033.png) | ![Evidence 034](../evidence/03-telemetry-c2/evidence-034.png) |

| Evidence 035 | Evidence 036 |
|---|---|
| ![Evidence 035](../evidence/03-telemetry-c2/evidence-035.png) | ![Evidence 036](../evidence/03-telemetry-c2/evidence-036.png) |

| Evidence 037 | Evidence 038 |
|---|---|
| ![Evidence 037](../evidence/03-telemetry-c2/evidence-037.png) | ![Evidence 038](../evidence/03-telemetry-c2/evidence-038.png) |

| Evidence 039 | Evidence 040 |
|---|---|
| ![Evidence 039](../evidence/03-telemetry-c2/evidence-039.png) | ![Evidence 040](../evidence/03-telemetry-c2/evidence-040.png) |

| Evidence 070 | Evidence 071 |
|---|---|
| ![Evidence 070](../evidence/03-telemetry-c2/evidence-070.png) | ![Evidence 071](../evidence/03-telemetry-c2/evidence-071.png) |

| Evidence 072 | Evidence 073 |
|---|---|
| ![Evidence 072](../evidence/03-telemetry-c2/evidence-072.png) | ![Evidence 073](../evidence/03-telemetry-c2/evidence-073.png) |

| Evidence 074 | Evidence 075 |
|---|---|
| ![Evidence 074](../evidence/03-telemetry-c2/evidence-074.png) | ![Evidence 075](../evidence/03-telemetry-c2/evidence-075.png) |

| Evidence 076 | Evidence 077 |
|---|---|
| ![Evidence 076](../evidence/03-telemetry-c2/evidence-076.png) | ![Evidence 077](../evidence/03-telemetry-c2/evidence-077.png) |

| Evidence 078 | Evidence 079 |
|---|---|
| ![Evidence 078](../evidence/03-telemetry-c2/evidence-078.png) | ![Evidence 079](../evidence/03-telemetry-c2/evidence-079.png) |

| Evidence 080 | Evidence 081 |
|---|---|
| ![Evidence 080](../evidence/03-telemetry-c2/evidence-080.png) | ![Evidence 081](../evidence/03-telemetry-c2/evidence-081.png) |

| Evidence 082 | Evidence 083 |
|---|---|
| ![Evidence 082](../evidence/03-telemetry-c2/evidence-082.png) | ![Evidence 083](../evidence/03-telemetry-c2/evidence-083.png) |

| Evidence 084 | Evidence 085 |
|---|---|
| ![Evidence 084](../evidence/03-telemetry-c2/evidence-084.png) | ![Evidence 085](../evidence/03-telemetry-c2/evidence-085.png) |

| Evidence 086 | Evidence 087 |
|---|---|
| ![Evidence 086](../evidence/03-telemetry-c2/evidence-086.png) | ![Evidence 087](../evidence/03-telemetry-c2/evidence-087.png) |

| Evidence 088 | Evidence 089 |
|---|---|
| ![Evidence 088](../evidence/03-telemetry-c2/evidence-088.png) | ![Evidence 089](../evidence/03-telemetry-c2/evidence-089.png) |

| Evidence 090 |  |
|---|---|
| ![Evidence 090](../evidence/03-telemetry-c2/evidence-090.png) |  |



## Next Report

[Report 4: Detection Engineering, Dashboard, Response, and Final Investigation](04-Detection-Dashboard-Active-Response-and-Final-Investigation.md)
