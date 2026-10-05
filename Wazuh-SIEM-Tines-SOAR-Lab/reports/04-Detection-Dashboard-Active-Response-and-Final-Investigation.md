# Report 4: Detection Engineering, Dashboard, Active Response, and Final Investigation

## Detection Engineering, Investigation, SOAR, and Response Validation

## My Proof of Work

I built the SOC dashboards, configured file integrity monitoring, created custom detections, investigated the resulting alerts, and implemented analyst-approved active response. I also built a SOAR workflow with Tines, VirusTotal, AbuseIPDB, Slack, and the Wazuh API. This report is evidence of the complete detection-to-response workflow I delivered.

## Project Overview

In this phase, I turned endpoint telemetry into a repeatable SOC workflow. I used Wazuh to detect and correlate activity and built dashboards to support prioritisation. I configured Tines to parse and enrich alerts, deliver analyst-facing reports, request approval, and send approved containment actions to Wazuh.

## Tools Used

| Tool | Purpose |
|---|---|
| Wazuh Dashboard | Visualisation, search, alert review, and investigation |
| Wazuh local rules | Custom account and SSH detections |
| File Integrity Monitoring | Detected changes to monitored files and paths |
| Tines | Orchestrated alert triage and response |
| VirusTotal | Supplied available IOC reputation context |
| AbuseIPDB | Supplied available IP reputation context |
| Slack | Received the structured SOC triage report |
| ngrok | Exposed the lab API through a temporary controlled tunnel |
| Wazuh API | Accepted the approved active-response request |
| `firewall-drop` | Blocked the selected source on the affected agent |

## Skills I Demonstrated

- Dashboard design
- File integrity monitoring
- Custom detection engineering
- Frequency-based SSH correlation
- IOC extraction and normalisation
- Threat intelligence enrichment
- Evidence-limited AI triage
- Slack reporting
- Human-in-the-loop approval
- Wazuh API integration
- Active-response validation

## Detection and Response Flow

```mermaid
flowchart TD
    A[Endpoint activity] --> B[Wazuh rule match]
    B --> C[Tines webhook]
    C --> D[Parse and enrich IOC]
    D --> E[Send SOC report to Slack]
    E --> F{Analyst approval}
    F -->|Approved| G[Wazuh API]
    F -->|Rejected| H[Document and close]
    G --> I[Agent firewall-drop]
    I --> J[Validate logs and connectivity]
```

My evidence covers the dashboards, custom detection rules, alert triage, Tines orchestration, enrichment, analyst approval, Wazuh Active Response, containment validation, and final SOC investigation I completed.

## Work I Completed

1. I turned raw telemetry into useful SOC views.
2. I created custom detection rules.
3. I detected repeated SSH failures and unusual Windows account changes.
4. I forwarded alerts to Tines.
5. I extracted indicators and limited public enrichment to supported public values.
6. I sent structured investigation reports to Slack.
7. I required analyst approval before blocking.
8. I executed Wazuh Active Response.
9. I validated the response using Tines, Wazuh, connectivity, and Ubuntu firewall evidence.

## File Integrity Monitoring View

I configured Wazuh FIM for `/opt/CompanyData` and reviewed the resulting Linux file events. The retained dashboard evidence shows `test.txt` modification and deletion activity.

![Ubuntu FIM configuration](../images/09-fim-ossec-configuration.png)

![Wazuh FIM events dashboard](../images/10-fim-events-dashboard.png)

I also reviewed a Windows FIM alert showing a modification to `C:\CompanyData\payroll.txt` on agent `001`.

![Windows FIM event](../images/40-windows-fim-event.png)

## SOC Activity and Outside-Hours Dashboards

I built a Basic SOC Activity Overview to bring Windows failed-logon counts, account-change activity, Linux authentication activity, and outside-hours events into one analyst view.

![Basic SOC Activity Overview](../images/37-basic-soc-activity-dashboard.png)

For the outside-hours view, I added the scripted field `outside_working_hours`. The field returned true when the event hour was before `07:00` or at or after `18:00`. The resulting panel displayed SSH, PAM, and syslog events for the Ubuntu endpoint.

![Outside-working-hours scripted field](../images/39-outside-working-hours-scripted-field.png)

![Linux security activity outside working hours](../images/38-linux-outside-hours-dashboard.png)

This view supports prioritisation. It does not prove malicious activity by itself. I still reviewed the user, source address, endpoint, event type, and surrounding timeline before deciding whether an event required escalation.

## Custom Detection Engineering

### SSH Brute-Force Correlation Rule

Rule `100101` correlated three SSH authentication failures from the same source IP within 120 seconds.

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

![Wazuh custom rule 100101 alert](../images/45-wazuh-custom-rule-100101-alert.png)

### Guest Account Rule

I configured rule `100200` to match Event ID `4722` when the built-in Guest account becomes enabled. I generated the matching action and confirmed the custom alert message in Wazuh.

```xml
<group name="windows,windows_security,account_changed,adduser">
  <rule id="100200" level="12">
    <if_sid>60103</if_sid>
    <field name="win.system.eventID">^4722$</field>
    <field name="win.eventdata.targetUserName">^Guest$</field>
    <description>Grace Windows Guest account was enabled.</description>
    <mitre>
      <id>T1098</id>
    </mitre>
    <group>windows,windows_account_management,account_enabled,guest_account,</group>
  </rule>
</group>
```

![Custom Wazuh rules](../images/11-custom-wazuh-rules.png)

![Guest account rule in the Wazuh editor](../images/43-wazuh-custom-rule-editor.png)

![Triggered Guest account custom alert](../images/42-wazuh-guest-account-alert.png)

### Rule Testing Process

1. Back up the current rules.
2. Add one rule change.
3. Validate XML syntax.
4. Test the rule and decoder relationship.
5. Restart the manager.
6. Generate the matching event.
7. Confirm the expected rule ID and level.
8. Review extracted fields.
9. Record false-positive and response considerations.

## File Integrity and Persistence Monitoring

The lab monitored changes linked to persistence and file activity. A controlled registry Run-key test added the `CalcPersist` value under:

```text
HKLM\Software\Microsoft\Windows\CurrentVersion\Run
```

The value launched `C:\Windows\System32\calc.exe`, providing safe evidence of a persistence-style registry change.

![Registry Run-key command](../images/13-registry-run-key-command.png)

Wazuh and Sysmon recorded discovery and process activity such as `whoami`. The supporting telemetry appears in the telemetry investigation report.

![Whoami process telemetry](../images/28-wazuh-whoami-telemetry.png)

## SOAR Extension

The bonus SOAR stage extended Wazuh from alert generation into triage, enrichment, communication, approval, and containment. Tines received the Wazuh event and coordinated the workflow while Wazuh remained the detection and endpoint-response platform.

### Wazuh to Tines Integration

I created a custom Wazuh integration by copying the supplied Shuffle integration and adapting it as `custom-tines`. I set ownership to `root:wazuh`, added the integration to `ossec.conf`, selected JSON alert format, and limited forwarding to rule `100101`.

![Custom Tines integration files](../images/55-custom-tines-integration-files.png)

![Custom Tines integration file ownership](../images/56-custom-tines-file-ownership.png)

![Wazuh Tines integration configuration](../images/57-wazuh-tines-integration-config.png)

## Tines SOAR and Active Response

### Workflow

I built and tested the response workflow using the following sequence:

1. Wazuh generated an alert.
2. Wazuh sent the alert to a Tines webhook.
3. Tines parsed the event.
4. An Event Transform extracted the source indicator.
5. VirusTotal and AbuseIPDB supplied available context.
6. An evidence-limited triage summary was created.
7. Slack received the SOC report.
8. The analyst reviewed an approval page.
9. Approval triggered a request to the Wazuh API.
10. Wazuh executed `firewall-drop` on the selected agent.
11. Connectivity and Wazuh logs validated the outcome.

![Wazuh webhook event in Tines](../images/16-tines-wazuh-webhook-event.png)

The retained webhook payload includes the Wazuh alert title, rule ID, timestamp, affected agent, previous SSH failures, full log, and source authentication data.

![Tines Wazuh webhook payload](../images/58-tines-wazuh-webhook-payload.png)

### IOC Extraction

I found that the source IP initially arrived as nested data with escaped quotation marks. Passing the whole structure created malformed JSON. I corrected the workflow so it selected one clean scalar IPv4 value before building the API request.

![Tines Event Transform configuration](../images/14-tines-event-transform.png)

I configured separate extraction fields for the block recommendation, IOC type, and IOC value.

![Tines IOC extraction fields](../images/59-tines-ioc-extraction-fields.png)

I then routed the event through a recommendation condition, which separated block and do-not-block outcomes.

![Tines block recommendation condition](../images/60-tines-block-recommendation-condition.png)

The retained image proves the transform configuration. The clean `srcip` value is supported by the later Wazuh Active Response event and Ubuntu firewall rule.

### Slack Notification

The Slack message contained findings, an investigation summary, the 5Ws and How, indicator context, and a recommended action.

![Slack triage report](../images/20-slack-triage-report.png)

### Human Approval

I configured the workflow to require an analyst decision before containment. This reduced the risk of blocking an administrative or trusted system because of a parsing error or incomplete alert.

![Human approval page](../images/21-human-approval-page.png)

The approval-page settings restricted access to team members and allowed anonymous submissions only after removing identifying browser and network data.

![Tines approval page settings](../images/61-tines-approval-page-settings.png)

### Slack Integration

I configured the `Wazuh-SOC` Slack application with `chat:write`, `chat:write.public`, and `users:read`, reviewed the requested permissions, and authorised it for the SOC workspace.

![Slack bot token scopes](../images/62-slack-bot-token-scopes.png)

![Slack bot token scope confirmation](../images/63-slack-bot-token-scopes-confirmation.png)

![Slack application authorisation](../images/64-slack-app-authorization.png)

The final Slack message presented the alert findings, rule details, source and target context, MITRE mapping, threat-intelligence result, and investigation summary.

![Slack SOC triage report](../images/65-slack-soc-triage-report.png)

The expanded Slack evidence records the source `192.168.66.131`, target account `graceogebe`, source port `50916`, three failed attempts, MITRE `T1110`, the private-address enrichment result, response recommendations, and the analyst approval link.

![Slack alert findings](../images/68-slack-alert-findings.png)

![Slack recommendations and approval link](../images/69-slack-recommendations-and-approval-link.png)

### Active Response Request

I sent the final request to Ubuntu agent `002` and passed the clean source address to the endpoint response script.

```json
{
  "arguments": [],
  "command": "!firewall-drop",
  "alert": {
    "data": {
      "srcip": "<<event_transform.ioc_value>>"
    }
  }
}
```

The final `ioc_value` must resolve to one IPv4 string. If the transform still returns an array, the transform must be corrected before this request runs. The response action must not receive escaped quotes, nested arrays, domains, hashes, usernames, or file paths through `srcip`.

I enabled the `firewall-drop` Active Response command for local execution on the selected Wazuh agent.

![Wazuh Active Response configuration](../images/44-wazuh-active-response-configuration.png)

![Ubuntu iptables block and controlled removal](../images/12-iptables-block-and-unblock.png)

## Final SSH Investigation

### Findings

- Rule `100101`, level 10, detected repeated SSH failures.
- Windows `192.168.66.131` was the source.
- Ubuntu `192.168.66.132`, agent `002`, was the target.
- The target account was `graceogebe`.
- Three failures occurred within the correlation window.
- The rule mapped the activity to `T1110`, Brute Force.
- Tines received and processed the alert.
- I identified the source as an RFC 1918 private address, excluded it from public reputation scoring, and relied on internal telemetry.
- The analyst approved containment.
- Wazuh recorded the Active Response program and source IP on the Ubuntu endpoint.
- Connectivity changed from successful replies to timeouts after the response request.

### 5Ws and How

| Question | Finding |
|---|---|
| **Who** | Windows `192.168.66.131` targeted user `graceogebe` on Ubuntu agent `002`. The lab analyst reviewed and approved the response. |
| **What** | Multiple failed SSH authentication attempts triggered custom correlation rule `100101`. Tines triaged the alert, and Wazuh processed the approved Active Response request. |
| **When** | The SSH sequence occurred on 24 September 2026 at approximately `16:28 UTC`. Final active-response validation occurred on 28 September 2026. |
| **Where** | The authentication attempts moved across the private lab network from Windows to Ubuntu at `192.168.66.132`. Wazuh at `192.168.66.128` analysed the event. |
| **Why** | The activity was intentionally generated to validate detection, investigation, enrichment, approval, containment, and response verification. |
| **How** | Ubuntu authentication logs recorded the failures. Wazuh correlated them. Tines processed the webhook and requested approval. After approval, the Wazuh API instructed agent `002` to run `firewall-drop` against `192.168.66.131`. |

### Classification

| Attribute | Value |
|---|---|
| Alert | Multiple SSH login failures observed from the same source IP |
| Rule ID | `100101` |
| Wazuh level | `10` |
| Category | Authentication failure correlation |
| ATT&CK tactic | Credential Access |
| ATT&CK technique | `T1110`, Brute Force |
| Source | `192.168.66.131` |
| Target | Ubuntu agent `002`, `192.168.66.132` |
| Target account | `graceogebe` |
| Disposition | True positive for the controlled lab scenario |
| Production impact | None |
| Response | Analyst-approved Active Response request with changed endpoint connectivity |

## Response Validation

### Before Containment

The Windows source reached the Ubuntu endpoint successfully.

![Connectivity before the response](../images/17-connectivity-before-response.png)

![Continuous connectivity before blocking](../images/70-connectivity-before-block.png)

### After Containment

The same connectivity test returned timeouts after the response.

![Connectivity transition after the response](../images/18-connectivity-response-transition.png)

The continuous test shows successful replies followed by repeated timeouts during the containment transition.

![Connectivity block transition](../images/66-connectivity-block-transition.png)

### Wazuh Evidence

Wazuh recorded the response program and the value passed to `data.srcip`.

![Wazuh active-response log](../images/19-wazuh-active-response-log.png)

The expanded Wazuh fields identify `active-response/bin/firewall-drop` and `data.srcip` value `192.168.66.131`.

![Wazuh firewall-drop event fields](../images/67-wazuh-firewall-drop-event-fields.png)

I assessed the response using three evidence sources:

1. Tines recorded the API action.
2. Wazuh recorded the response script and source IP.
3. Endpoint connectivity changed from replies to timeouts.

An HTTP success status alone did not prove containment. The combined evidence supports response execution, changed endpoint behaviour, and the presence of Ubuntu DROP rules for `192.168.66.131`. The screenshot also shows controlled removal from INPUT and FORWARD. A successful connectivity test after removal was not retained.

## Incident Timeline

| Date and time | Event |
|---|---|
| 24 Sep 2026, approximately 16:28 UTC | Repeated SSH failures reached Ubuntu |
| 24 Sep 2026, 16:28:46 UTC | Tines received the Wazuh webhook |
| 24 Sep 2026 | Tines extracted the source address and reviewed enrichment |
| 24 Sep 2026 | Slack received the SOC report |
| 28 Sep 2026 | Connectivity succeeded before containment |
| 28 Sep 2026 | Analyst approved the block |
| 28 Sep 2026 | Wazuh Active Response received `192.168.66.131` |
| 28 Sep 2026 | Connectivity timed out after containment |

## MITRE ATT&CK Mapping

| Technique | Name | Evidence |
|---|---|---|
| `T1110` | Brute Force | Repeated SSH failures |
| `T1021.004` | Remote Services: SSH | Remote authentication testing |
| `T1136.001` | Create Account: Local Account | Controlled local account creation |
| `T1098` | Account Manipulation | Local Administrators membership change |
| `T1547.001` | Registry Run Keys / Startup Folder | `CalcPersist` Run-key value |
| `T1059.001` | PowerShell | PowerShell process activity |
| `T1095` | Non-Application Layer Protocol | Controlled reverse TCP callbacks |
| `T1105` | Ingress Tool Transfer | Controlled Linux payload transfer. Synthetic SOAR payload fields were not treated as observed endpoint evidence. |

## Challenges and Resolutions

| Challenge | Resolution |
|---|---|
| Custom XML caused Wazuh service errors | Validated structure before restart |
| Agent ID was missing from the API URL | Passed `agent.id` to `agents_list` |
| IOC value contained quotes and nested arrays | Extracted one clean scalar IPv4 value |
| API request succeeded without blocking | Checked the command name, endpoint script, `wazuh-execd`, source value, and target agent |
| Reputation checks received a private IP | Treated public reputation as non-authoritative for RFC 1918 data |
| Automated blocking introduced business risk | Added analyst approval and infrastructure exclusions |

## Recommendations Based on My Findings

1. Keep human approval for medium-confidence containment.
2. Exclude the Wazuh manager, gateways, DNS, and trusted administration systems from automatic blocking.
3. Validate indicator type before enrichment or response.
4. Store the analyst, reason, alert ID, target agent, source address, and response result.
5. Add an expiry and controlled unblock process.
6. Search for successful logons following repeated failures.
7. Correlate authentication, process, network, account, and registry activity.
8. Test custom rules after Wazuh upgrades.
9. Add notification and escalation paths for failed response actions.
10. Preserve original event timestamps and timezone information.

## What I Achieved

- Custom Wazuh rules
- Authentication correlation
- After-hours dashboard views
- Windows and Linux investigation tables
- Webhook delivery from Wazuh to Tines
- IOC extraction and enrichment
- Slack SOC reporting
- Human approval before containment
- Agent-specific Wazuh Active Response
- Multi-source response evidence with endpoint firewall-rule validation
- Full 5Ws and How investigation reporting

## Limitations and Remaining Validation

- The private SSH source did not support meaningful public reputation scoring.
- The AI triage stage depended on evidence in the Wazuh webhook and did not replace analyst review.
- Changed ping behaviour supported the response assessment but did not identify the exact endpoint firewall rule.
- Linux and Windows require separate, tested response scripts.
- A complete recovery validation still needs a successful connectivity test after rule removal and repeated tests for false-positive measurement.

## Complete Screenshot Evidence

The following images preserve the original dashboard, rule, Tines, Slack, API, approval, response, and final validation evidence in sequence.

| Evidence 041 | Evidence 042 |
|---|---|
| ![Evidence 041](../evidence/04-dashboard-rules-soar/evidence-041.png) | ![Evidence 042](../evidence/04-dashboard-rules-soar/evidence-042.png) |

| Evidence 043 | Evidence 044 |
|---|---|
| ![Evidence 043](../evidence/04-dashboard-rules-soar/evidence-043.png) | ![Evidence 044](../evidence/04-dashboard-rules-soar/evidence-044.png) |

| Evidence 045 | Evidence 046 |
|---|---|
| ![Evidence 045](../evidence/04-dashboard-rules-soar/evidence-045.png) | ![Evidence 046](../evidence/04-dashboard-rules-soar/evidence-046.png) |

| Evidence 047 | Evidence 048 |
|---|---|
| ![Evidence 047](../evidence/04-dashboard-rules-soar/evidence-047.png) | ![Evidence 048](../evidence/04-dashboard-rules-soar/evidence-048.png) |

| Evidence 049 | Evidence 050 |
|---|---|
| ![Evidence 049](../evidence/04-dashboard-rules-soar/evidence-049.png) | ![Evidence 050](../evidence/04-dashboard-rules-soar/evidence-050.png) |

| Evidence 051 | Evidence 052 |
|---|---|
| ![Evidence 051](../evidence/04-dashboard-rules-soar/evidence-051.png) | ![Evidence 052](../evidence/04-dashboard-rules-soar/evidence-052.png) |

| Evidence 053 | Evidence 091 |
|---|---|
| ![Evidence 053](../evidence/04-dashboard-rules-soar/evidence-053.png) | ![Evidence 091](../evidence/04-dashboard-rules-soar/evidence-091.png) |

| Evidence 092 | Evidence 093 |
|---|---|
| ![Evidence 092](../evidence/04-dashboard-rules-soar/evidence-092.png) | ![Evidence 093](../evidence/04-dashboard-rules-soar/evidence-093.png) |

| Evidence 094 | Evidence 095 |
|---|---|
| ![Evidence 094](../evidence/04-dashboard-rules-soar/evidence-094.png) | ![Evidence 095](../evidence/04-dashboard-rules-soar/evidence-095.png) |

| Evidence 096 | Evidence 097 |
|---|---|
| ![Evidence 096](../evidence/04-dashboard-rules-soar/evidence-096.png) | ![Evidence 097](../evidence/04-dashboard-rules-soar/evidence-097.png) |

| Evidence 098 | Evidence 099 |
|---|---|
| ![Evidence 098](../evidence/04-dashboard-rules-soar/evidence-098.png) | ![Evidence 099](../evidence/04-dashboard-rules-soar/evidence-099.png) |

| Evidence 100 | Evidence 101 |
|---|---|
| ![Evidence 100](../evidence/04-dashboard-rules-soar/evidence-100.png) | ![Evidence 101](../evidence/04-dashboard-rules-soar/evidence-101.png) |

| Evidence 102 | Evidence 103 |
|---|---|
| ![Evidence 102](../evidence/04-dashboard-rules-soar/evidence-102.png) | ![Evidence 103](../evidence/04-dashboard-rules-soar/evidence-103.png) |

| Evidence 104 | Evidence 105 |
|---|---|
| ![Evidence 104](../evidence/04-dashboard-rules-soar/evidence-104.png) | ![Evidence 105](../evidence/04-dashboard-rules-soar/evidence-105.png) |

| Evidence 106 | Evidence 107 |
|---|---|
| ![Evidence 106](../evidence/04-dashboard-rules-soar/evidence-106.png) | ![Evidence 107](../evidence/04-dashboard-rules-soar/evidence-107.png) |

| Evidence 108 | Evidence 109 |
|---|---|
| ![Evidence 108](../evidence/04-dashboard-rules-soar/evidence-108.png) | ![Evidence 109](../evidence/04-dashboard-rules-soar/evidence-109.png) |

| Evidence 110 | Evidence 111 |
|---|---|
| ![Evidence 110](../evidence/04-dashboard-rules-soar/evidence-110.png) | ![Evidence 111](../evidence/04-dashboard-rules-soar/evidence-111.png) |

| Evidence 112 | Evidence 113 |
|---|---|
| ![Evidence 112](../evidence/04-dashboard-rules-soar/evidence-112.png) | ![Evidence 113](../evidence/04-dashboard-rules-soar/evidence-113.png) |

| Evidence 114 | Evidence 115 |
|---|---|
| ![Evidence 114](../evidence/04-dashboard-rules-soar/evidence-114.png) | ![Evidence 115](../evidence/04-dashboard-rules-soar/evidence-115.png) |

| Evidence 116 |  |
|---|---|
| ![Evidence 116](../evidence/04-dashboard-rules-soar/evidence-116.png) |  |



## Master Report

[Return to the Comprehensive Wazuh SIEM and Tines SOAR Report](../README.md)
