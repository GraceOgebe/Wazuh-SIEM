# Wazuh SIEM and Tines SOAR Lab

## Multi-Platform Detection, Investigation, Threat Enrichment, and Automated Response

## Objective

Build and validate a home Security Operations Centre lab that collects Windows and Linux telemetry, detects suspicious activity, supports investigation in Wazuh, enriches indicators through threat intelligence services, sends structured reports to Slack, and performs analyst-approved active response through Tines.

The project focuses on the full SOC workflow:

1. Generate controlled security events.
2. Collect endpoint telemetry.
3. Detect and investigate suspicious activity.
4. Enrich indicators with threat intelligence.
5. Notify the analyst through Slack.
6. Request human approval before containment.
7. Execute and verify the response.

> All activity was completed in an isolated virtual lab. The IP addresses shown are private RFC 1918 addresses used only within the lab.

## Project Overview

This project was completed by following the **MYDFIR Wazuh Challenge** and extending the challenge into a documented SOC detection and response portfolio project. It combines Wazuh SIEM with Windows Sysmon, Linux monitoring, custom detection rules, dashboards, Tines SOAR, VirusTotal, AbuseIPDB, Slack, and Wazuh Active Response.

The lab monitored account activity, authentication events, process execution, registry persistence, SSH failures, after-hours activity, and controlled command-and-control-style behaviour. Alerts were sent from Wazuh to Tines, analysed and enriched, formatted into a SOC triage report, and delivered to Slack. Where blocking was recommended, the workflow required an analyst decision before sending the active-response request to Wazuh.

![Wazuh dashboard overview](images/01-wazuh-dashboard-overview.png)

## Lab Architecture

![Wazuh SIEM and Tines SOAR lab architecture](images/00-lab-architecture.svg)

| System | Role | IP address |
|---|---|---|
| Wazuh server | Manager, indexer, dashboard, API, and central analysis | `192.168.66.128` |
| Windows 10 | Monitored endpoint with Wazuh agent and Sysmon | `192.168.66.131` |
| Ubuntu | Monitored Linux endpoint and SSH target | `192.168.66.132` |
| Kali Linux | Controlled attack simulation system | `192.168.66.133` |
| Tines | SOAR workflow, enrichment, approval, and response orchestration | Cloud service |
| Slack | SOC notification and triage channel | Cloud service |

## Tools Used

- Wazuh Manager, Indexer, Dashboard, and API
- Wazuh agents for Windows and Ubuntu
- Sysmon for Windows
- Linux Sysmon parser
- VMware Workstation
- Windows Event Viewer and PowerShell
- Ubuntu audit and authentication logs
- Kali Linux
- Tines SOAR
- VirusTotal
- AbuseIPDB
- Slack
- ngrok for temporary lab API connectivity
- MITRE ATT&CK

## Capabilities Demonstrated

- Windows and Linux log collection
- Sysmon process and network telemetry
- Windows account lifecycle monitoring
- Windows logon investigation
- SSH authentication monitoring
- Custom Wazuh rule creation
- Correlation of repeated authentication failures
- After-hours activity dashboards
- Indicator extraction and threat intelligence enrichment
- AI-assisted alert triage using alert evidence only
- Slack notification and case communication
- Human-in-the-loop containment approval
- Wazuh API authentication and active response
- Response verification using endpoint and Wazuh telemetry

## Environment Setup

### Agent Deployment

The Windows 10 and Ubuntu systems were enrolled as Wazuh agents. Agent connectivity was verified from the Wazuh dashboard before testing detection scenarios.

![Connected Windows and Ubuntu agents](images/02-connected-agents.png)

### Windows Telemetry

Sysmon was installed on the Windows endpoint using the Sysmon Modular configuration. Windows Security and Sysmon events were forwarded to Wazuh for process, account, authentication, registry, and network analysis.

![Windows event search in Wazuh](images/03-windows-event-search.png)

### Linux Telemetry

The Ubuntu agent collected authentication, SSH, system, and process activity. A Linux Sysmon decoder was added to the manager to improve parsing and field extraction for Linux events.

## Detection Scenarios

### 1. Windows Account Creation

A local Windows account was created during the simulation. Wazuh recorded Windows Security Event ID `4720`, which identifies the creation of a user account.

![Windows account creation event 4720](images/05-account-creation-event-4720.png)

### 2. Windows Account Deletion

The test account was deleted to validate account lifecycle monitoring. Wazuh recorded Windows Security Event ID `4726` and exposed the subject, target account, security identifier, host, and timestamp for investigation.

![Windows account deletion event 4726](images/04-account-deletion-event-4726.png)

### 3. Successful Logon Investigation

Windows Security Event ID `4624` was reviewed to identify successful authentication activity. Logon type, account name, source address, process, workstation, and security identifier were used to determine the nature of the logon.

![Successful Windows logon event 4624](images/06-successful-logon-event-4624.png)

Security identifiers were also resolved during the investigation to link raw SID values to Windows accounts.

![SID investigation](images/07-sid-investigation.png)

### 4. Repeated SSH Authentication Failures

Repeated failed SSH logins were generated against the Ubuntu endpoint. A custom correlation rule detected three failures from the same source within a 120-second period.

```xml
<group name="local,syslog,sshd,authentication_failed,">
  <rule id="100101" level="10" frequency="3" timeframe="120">
    <if_matched_sid>5760</if_matched_sid>
    <same_source_ip />
    <description>Multiple SSH login failures observed from the same source IP</description>
    <mitre>
      <id>T1110</id>
    </mitre>
    <group>authentication_failed,ssh_bruteforce,credential_access,</group>
  </rule>
</group>
```

![SSH alert investigation](images/08-ssh-alert-investigation.png)

### 5. Activity Outside Working Hours

Custom dashboards were created to highlight Windows and Linux authentication or account activity outside the expected period of 06:00 to 20:00. This supports faster review of events occurring at unusual times.

![Outside working hours dashboard](images/09-outside-working-hours-dashboard.png)

![Linux activity outside working hours](images/10-linux-activity-table.png)

### 6. Registry Run Key Persistence

A controlled Windows persistence test added `calc.exe` to the `HKLM\Software\Microsoft\Windows\CurrentVersion\Run` registry path. Sysmon telemetry allowed the command and related process activity to be reviewed in Wazuh.

```cmd
reg add "HKLM\Software\Microsoft\Windows\CurrentVersion\Run" /v CalcPersist /t REG_SZ /d "C:\Windows\System32\calc.exe" /f
```

![Controlled account and registry persistence test](images/13-registry-run-key-command.png)

### 7. Process Execution Investigation

Process telemetry was searched for commands such as `whoami` to validate that Sysmon events reached Wazuh with the process name, user, parent process, endpoint, and execution time.

![Whoami process telemetry](images/14-whoami-process-telemetry.png)

## Custom Detection Engineering

Local rules were developed and tested for account management and authentication activity. Examples included enabling the Windows Guest account and correlating repeated SSH failures.

![Custom Wazuh rules](images/11-custom-wazuh-rules.png)

Key rule design considerations included:

- Use a unique local rule ID range.
- Match the correct parent rule or Windows Event ID.
- Filter on meaningful fields such as account name and source IP.
- Assign severity based on business context.
- Map behaviour to MITRE ATT&CK.
- Test XML syntax before restarting the manager.
- Validate the rule with real events in the dashboard.

## Tines SOAR Workflow

### Workflow Design

The automation flow used the following sequence:

1. Wazuh sends the complete alert to a Tines webhook.
2. The AI agent reviews only the `all_fields` alert object.
3. Indicator values are extracted from the output.
4. Public indicators are enriched through VirusTotal and AbuseIPDB.
5. A structured SOC report is posted to Slack.
6. The workflow decides whether blocking is recommended.
7. A human approval page presents the response decision.
8. Approved actions are sent to the Wazuh Active Response API.
9. Wazuh executes the endpoint firewall script.
10. The result is verified through connectivity testing and Wazuh logs.

### IOC Extraction

The Event Transform separated `recommend_block`, `ioc_type`, and `ioc_value` from the AI output. The IOC was normalised into one clean scalar value before being passed to Wazuh.

![Tines IOC extraction](images/15-tines-ioc-extraction.png)

### SOC Triage Report

The AI prompt was restricted to evidence present in the Wazuh alert. Missing fields were reported as unavailable instead of being inferred. The Slack report included findings, an investigation summary, 5W1H, recommendations, and the block decision.

![SOC triage report in Slack](images/16-slack-triage-report.png)

### Human Approval

Containment was not performed immediately after an AI recommendation. The workflow presented a Yes or No approval page so the analyst retained control over the final action.

![Human approval page](images/21-human-approval-page.png)

## Active Response

The Wazuh API request targeted the agent that generated the alert. For the Ubuntu endpoint, the `firewall-drop` script received the extracted IP through the `srcip` field.

```json
{
  "arguments": [],
  "command": "!firewall-drop",
  "alert": {
    "data": {
      "srcip": "<<event_transform.ioc_value[0][0]>>"
    }
  }
}
```

The exclamation mark instructs Wazuh to treat the command as an executable script. Windows endpoints require a Windows-compatible response script such as `netsh.exe`; Linux and Unix endpoints use `firewall-drop`.

The Wazuh agent exposed its available response commands before the containment test.

![Available active response commands](images/12-active-response-commands.png)

The incoming webhook confirmed the affected Ubuntu agent as ID `002` with address `192.168.66.132`.

![Wazuh webhook event in Tines](images/17-tines-wazuh-webhook-event.png)

### Response Validation

Connectivity from the Windows test source to the Ubuntu endpoint was confirmed before the response.

![Connectivity before active response](images/18-connectivity-before-block.png)

After the approved firewall response, the same connectivity test timed out, showing the endpoint firewall action had taken effect.

![Connectivity after active response](images/19-connectivity-after-block.png)

Wazuh recorded the active-response program and the source IP passed to the script.

![Wazuh active response log](images/20-active-response-log.png)

## SOC Investigation Report

### Findings

- Custom rule `100101`, level 10, detected repeated SSH authentication failures against Ubuntu agent `002` at `192.168.66.132`.
- The source address was `192.168.66.131`, the Windows lab endpoint.
- The test targeted the Ubuntu account `graceogebe`.
- Three failed attempts occurred within the configured correlation window.
- The rule mapped the activity to MITRE ATT&CK `T1110`, Brute Force.
- VirusTotal and AbuseIPDB identified the source as a private RFC 1918 address, so public reputation results were not meaningful for this internal lab IOC.
- Tines produced a structured triage report and requested analyst approval.
- Wazuh Active Response executed `firewall-drop` on the Ubuntu agent.
- Post-response connectivity testing showed timeouts, and Wazuh logged the response action.

### Investigation Summary

The Ubuntu endpoint received repeated failed SSH authentication attempts from the Windows lab system. Wazuh correlated the failures and generated a high-priority custom alert. Tines received the full alert, extracted the indicator, enriched the available context, and sent a triage report to Slack. The workflow recommended containment and presented the decision to the analyst. Following approval, Tines called the Wazuh Active Response API and instructed the Ubuntu agent to block the source IP. Endpoint testing and Wazuh telemetry confirmed the response.

### 5W1H

| Question | Answer |
|---|---|
| Who | Source endpoint `192.168.66.131` targeted user `graceogebe` on Ubuntu agent `002`. |
| What | Multiple failed SSH authentication attempts triggered custom rule `100101`. |
| When | Events were generated and investigated during the September 2026 lab exercises. Exact event timestamps remain visible in the supporting screenshots. |
| Where | The activity occurred between the Windows endpoint and Ubuntu endpoint on the isolated `192.168.66.0/24` lab network. |
| Why | The activity was intentionally generated to validate SSH brute-force detection, triage, enrichment, approval, and containment. |
| How | Repeated SSH failures were correlated by Wazuh, sent to Tines, reviewed through the SOAR workflow, and contained through Wazuh Active Response. |

## MITRE ATT&CK Mapping

| Technique | Name | Lab evidence |
|---|---|---|
| `T1110` | Brute Force | Repeated SSH authentication failures |
| `T1021.004` | Remote Services: SSH | SSH access and authentication testing |
| `T1098` | Account Manipulation | Local account creation and account changes |
| `T1078` | Valid Accounts | Successful logon investigation |
| `T1547.001` | Registry Run Keys / Startup Folder | Controlled Run key persistence test |
| `T1059.001` | PowerShell | PowerShell process activity in Windows telemetry |
| `T1105` | Ingress Tool Transfer | Simulated staged download alert used in the SOAR test payload |
| `T1070.004` | File Deletion | File deletion monitoring and investigation workflow |

## Investigation Challenges and Resolutions

| Challenge | Resolution |
|---|---|
| Wazuh manager failed after configuration edits | XML syntax was validated and mismatched or duplicated tags were corrected. |
| Linux agent failed to start | Incorrect XML closing tags and extra content were removed from `ossec.conf`. |
| Active-response agent ID was blank | The correct `agent.id` field was passed to the `agents_list` query parameter. |
| IOC arrived as a nested array with escaped quotation marks | The Event Transform regex was narrowed to the IPv4 value and the scalar element was selected. |
| API accepted the request but did not block | The correct endpoint script, clean `srcip`, target agent, and `wazuh-execd` execution path were verified. |
| Public threat intelligence returned no result for the source | The address was recognised as an internal RFC 1918 IOC and investigated using internal telemetry. |
| Risk of automatic false-positive containment | A human approval step was placed before Active Response. |

## Recommendations

1. Keep human approval for disruptive containment actions until detection quality and false-positive rates are measured.
2. Use separate platform-specific response paths for Windows and Linux endpoints.
3. Validate IOC type and format before sending an active-response request.
4. Exclude loopback, broadcast, gateway, DNS, Wazuh manager, and approved infrastructure addresses from automatic blocking.
5. Avoid sending private IP addresses to public reputation services unless the integration explicitly supports internal intelligence.
6. Add alert deduplication and response timeouts to prevent repeated blocks.
7. Store response decisions, approver identity, timestamp, target agent, IOC, and API result for audit purposes.
8. Add recovery actions for false positives, including controlled unblock procedures.
9. Monitor `active-responses.log` and generate alerts for failed response execution.
10. Expand the workflow to isolate endpoints, disable accounts, or quarantine files only after platform-specific testing.

## Security Operations Takeaways

- Detection engineering depends on clean telemetry and accurate field mapping.
- A successful API response does not prove the endpoint action succeeded. Response verification is essential.
- Private IP indicators require internal context rather than public reputation alone.
- AI-generated triage needs evidence restrictions and structured output.
- Human approval reduces the risk of automated containment affecting trusted systems.
- Threat intelligence adds context, while SOC telemetry confirms what occurred inside the environment.
- Linking Wazuh, Tines, Slack, and Active Response creates a repeatable detection-to-containment process.

## Repository Structure

```text
Wazuh-SIEM-Tines-SOAR-Lab/
├── README.md
└── images/
    ├── 01-wazuh-dashboard-overview.png
    ├── 02-connected-agents.png
    ├── ...
    └── 21-human-approval-page.png
```

## References

- [Wazuh documentation](https://documentation.wazuh.com/current/)
- [Wazuh API reference](https://documentation.wazuh.com/current/user-manual/api/reference.html)
- [Wazuh default active response scripts](https://documentation.wazuh.com/current/user-manual/capabilities/active-response/default-active-response-scripts.html)
- [Sysmon Modular](https://github.com/olafhartong/sysmon-modular)
- [MITRE ATT&CK](https://attack.mitre.org/)
- [Tines documentation](https://www.tines.com/docs/)
- [Microsoft Windows security auditing documentation](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/)

## Acknowledgement

This project was completed as part of the **MYDFIR Wazuh Challenge**. The challenge provided the practical foundation for building the Wazuh lab, testing detections, investigating alerts, and connecting Wazuh to a SOAR workflow.

The presentation style and high-level report organisation were informed by Justin Jude Fernandes' public SOC investigation portfolio:

[Microsoft Defender XDR Phishing-Led Multi-Stage Attack Simulation and End-to-End SOC Investigation](https://github.com/justinjudefernandes/Microsoft-Defender-XDR-Phishing-Led-Multi-Stage-Attack-Simulation-End-to-End-SOC-Investigation)

The technical work, screenshots, analysis, and Wazuh and Tines implementation documented here are from my own lab.

## Author

Eneattah Grace Ogebe
