# Evidence Index

## Purpose

This index identifies the strongest evidence supporting the Wazuh SIEM and Tines SOAR project. The detailed reports contain the complete set of 116 original screenshots. The selected evidence below supports the main portfolio claims without requiring a reviewer to inspect every image.

| Evidence ID | Source | Evidence | What it supports |
|---|---|---|---|
| E01 | Architecture diagram | [Lab architecture](images/00-lab-architecture.svg) | Four-system design, roles, addresses, and data flow |
| E02 | Wazuh Dashboard | [Initial dashboard access](images/01-wazuh-dashboard-overview.png) | Dashboard access before agent enrolment. The image shows no registered agents. |
| E03 | Wazuh agent inventory | [Connected agents](images/02-connected-agents.png) | Windows and Ubuntu enrolment |
| E04 | Wazuh Windows Security telemetry | [Account creation, Event ID 4720](images/30-wazuh-event-4720-account-created.png) | Local account `student1` created by `Grace Ogebe` on the Windows endpoint |
| E05 | Windows Security telemetry | [Account deletion, Event ID 4726](images/04-account-deletion-event-4726.png) | Account lifecycle visibility |
| E06 | Wazuh Windows Security telemetry | [Successful logon, Event ID 4624](images/31-wazuh-event-4624-successful-logon.png) | Successful service logon with Logon Type `5` |
| E07 | Wazuh source event | [SSH authentication source event](images/08-ssh-authentication-source-event.png) | Source IP, target user, agent, and SSH authentication fields. The image shows rule `5710`, not custom rule `100101`. |
| E08 | Wazuh rules | [Custom rule configuration](images/11-custom-wazuh-rules.png) | Detection engineering |
| E09 | Ubuntu configuration | [FIM configuration](images/09-fim-ossec-configuration.png) | Monitored `/opt/CompanyData` path. The screenshot also records the malformed closing tag later corrected. |
| E10 | Wazuh FIM | [FIM events dashboard](images/10-fim-events-dashboard.png) | File modification and deletion events for `/opt/CompanyData/test.txt` |
| E11 | Windows command shell | [Run-key and account commands](images/13-registry-run-key-command.png) | Account creation, group change, failed syntax, and successful `CalcPersist` command |
| E12 | Metasploit | [Windows session](images/23-msfconsole-windows-session.png) | Controlled Windows reverse connection |
| E13 | Metasploit | [Linux session](images/27-msfconsole-linux-session.png) | Controlled Linux reverse connection |
| E14 | Tines | [Event Transform configuration](images/14-tines-event-transform.png) | Field-extraction workflow configuration |
| E15 | Tines | [Block condition](images/15-tines-block-condition.png) | Conditional branch based on the extracted recommendation value |
| E16 | Tines | [Wazuh webhook event](images/16-tines-wazuh-webhook-event.png) | Alert timestamp and Ubuntu agent context |
| E17 | Tines | [Human approval](images/21-human-approval-page.png) | Human control before containment |
| E18 | Endpoint test | [Connectivity before response](images/17-connectivity-before-response.png) | Pre-response connectivity baseline |
| E19 | Endpoint test | [Connectivity transition](images/18-connectivity-response-transition.png) | Replies followed by timeouts during response testing |
| E20 | Wazuh telemetry | [Active Response log](images/19-wazuh-active-response-log.png) | `active-response/bin/firewall-drop` and `srcip` evidence |
| E21 | Ubuntu firewall | [iptables block and removal](images/12-iptables-block-and-unblock.png) | DROP rules for `192.168.66.131` in INPUT and FORWARD, followed by controlled rule removal |
| E22 | Slack | [SOC triage report](images/20-slack-triage-report.png) | Analyst-facing report for rule `100101`. The private address was labelled malicious by lab instruction, not public intelligence. |
| E23 | Wazuh Windows Security telemetry | [Local-group membership, Event ID 4732](images/32-wazuh-event-4732-group-membership.png) | `student1` added to the local Administrators group |
| E24 | Wazuh Ubuntu telemetry | [Successful SSH authentication](images/33-wazuh-successful-ssh-authentication.png) | Accepted password for `graceogebe` from `192.168.66.131` to agent `002` |
| E25 | Windows terminal | [Successful SSH session](images/34-windows-successful-ssh-session.png) | Interactive access to Ubuntu as `graceogebe` after earlier failed tests |
| E26 | Wazuh Ubuntu telemetry | [Linux process discovery](images/35-wazuh-linux-whoami-process.png) | Sysmon for Linux event containing `whoami` activity for `graceogebe` |
| E27 | Wazuh Ubuntu telemetry | [Linux session event](images/36-wazuh-linux-session-telemetry.png) | `systemd` session started for user `graceogebe` |
| E28 | Wazuh Dashboard | [SOC activity overview](images/37-basic-soc-activity-dashboard.png) | Windows failed-logon and account-change panels |
| E29 | Wazuh Dashboard | [Outside-hours Linux activity](images/38-linux-outside-hours-dashboard.png) | SSH, PAM, and syslog activity displayed outside working hours |
| E30 | Wazuh index pattern | [Outside-hours scripted field](images/39-outside-working-hours-scripted-field.png) | Script logic marking activity before `07:00` or at/after `18:00` |
| E31 | Wazuh FIM | [Windows FIM event](images/40-windows-fim-event.png) | Modification of `C:\\CompanyData\\payroll.txt` on agent `001` |
| E32 | Ubuntu configuration | [FIM configuration error](images/41-ubuntu-fim-config-error.png) | Malformed closing tag that caused the agent configuration issue before correction |
| E33 | Wazuh custom alert | [Guest account alert](images/42-wazuh-guest-account-alert.png) | Triggered custom message for the Windows Guest account being enabled |
| E34 | Wazuh rule editor | [Guest-account rule configuration](images/43-wazuh-custom-rule-editor.png) | Event `4722` target-account match and rule `100200` configuration |
| E35 | Wazuh manager configuration | [Active Response configuration](images/44-wazuh-active-response-configuration.png) | Enabled `firewall-drop` Active Response command with local execution |
| E36 | Wazuh custom alert | [Rule 100101 alert](images/45-wazuh-custom-rule-100101-alert.png) | Level 10 SSH correlation, frequency `3`, timeframe `120`, and MITRE `T1110` |
| E37 | Metasploit | [Local exploit suggester, part 1](images/46-windows-local-exploit-suggester-1.png) | Potential Windows local privilege and persistence checks. Suggestions are not treated as successful exploitation. |
| E38 | Metasploit | [Local exploit suggester, part 2](images/47-windows-local-exploit-suggester-2.png) | Remaining candidate modules and check results |
| E39 | Metasploit | [Handler and UAC test](images/48-windows-handler-and-uac-test.png) | Windows callback, UAC state, Administrators membership, and controlled `fodhelper` test |
| E40 | Wazuh Sysmon | [Windows whoami event](images/49-wazuh-windows-whoami-event.png) | Process image, host, agent, and timestamp for identity discovery |
| E41 | Windows command shell | [Account and Run-key commands](images/50-windows-account-and-run-key-commands.png) | Account changes and successful `CalcPersist` command |
| E42 | Ubuntu endpoint | [Payload download and execution](images/51-ubuntu-payload-download-and-execution.png) | `wget`, file creation, permission change, and execution attempt |
| E43 | Metasploit | [Ubuntu Meterpreter details](images/52-ubuntu-meterpreter-session-details.png) | Session `3`, callback addresses, `getuid`, and shell creation |
| E44 | Ubuntu shell | [OS and user discovery](images/53-ubuntu-os-and-user-discovery.png) | Ubuntu version, PTY creation, and `whoami` |
| E45 | Ubuntu shell | [Account and group discovery](images/54-ubuntu-account-and-group-discovery.png) | UID, group membership, and `/etc/passwd` review |
| E46 | Wazuh server | [Custom Tines integration files](images/55-custom-tines-integration-files.png) | Creation of `custom-tines` integration files |
| E47 | Wazuh server | [Integration ownership](images/56-custom-tines-file-ownership.png) | `root:wazuh` ownership applied to custom integration files |
| E48 | Wazuh configuration | [Tines integration configuration](images/57-wazuh-tines-integration-config.png) | JSON forwarding through `custom-tines` for rule `100101` |
| E49 | Tines | [Webhook payload](images/58-tines-wazuh-webhook-payload.png) | Received Wazuh SSH alert and supporting fields |
| E50 | Tines | [IOC extraction fields](images/59-tines-ioc-extraction-fields.png) | Extraction of recommendation, IOC type, and IOC value |
| E51 | Tines | [Recommendation condition](images/60-tines-block-recommendation-condition.png) | Block and do-not-block workflow routing |
| E52 | Tines | [Approval settings](images/61-tines-approval-page-settings.png) | Team access and submission-privacy settings |
| E53 | Slack | [Bot scopes](images/62-slack-bot-token-scopes.png) | `chat:write`, `chat:write.public`, and `users:read` permissions |
| E54 | Slack | [Bot-scope confirmation](images/63-slack-bot-token-scopes-confirmation.png) | Duplicate retained view confirming selected scopes |
| E55 | Slack | [Application authorisation](images/64-slack-app-authorization.png) | Workspace permission review for `Wazuh-SOC` |
| E56 | Slack | [SOC triage report](images/65-slack-soc-triage-report.png) | Delivered analyst-facing investigation output |
| E57 | Windows endpoint | [Connectivity block transition](images/66-connectivity-block-transition.png) | Successful replies changing to timeouts during response testing |
| E58 | Wazuh telemetry | [Firewall-drop event fields](images/67-wazuh-firewall-drop-event-fields.png) | Active Response program and `srcip` value `192.168.66.131` |
| E59 | Slack | [Alert findings](images/68-slack-alert-findings.png) | Rule, source, target, port, frequency, MITRE, and enrichment summary |
| E60 | Slack | [Recommendations and approval link](images/69-slack-recommendations-and-approval-link.png) | Analyst recommendations and decision path |
| E61 | Windows endpoint | [Connectivity before block](images/70-connectivity-before-block.png) | Continuous successful ICMP replies before containment |

## Validation Standard

Each major finding should connect to at least one screenshot, event field, rule, query, or response log.

Screenshots should be interpreted with the surrounding report text. A screenshot of an API success status does not prove endpoint containment. Response claims require workflow evidence, Wazuh execution evidence, and endpoint-level verification.

## Full Evidence Folders

| Folder | Contents | Count |
|---|---|---|
| `evidence/01-server-deployment/` | Server installation, networking, services, and initial dashboard | 10 |
| `evidence/02-agents-sysmon/` | Agent enrolment, Sysmon, decoder, services, and endpoint configuration | 26 |
| `evidence/03-telemetry-c2/` | Account, authentication, process, registry, payload, Meterpreter, and C2 evidence | 41 |
| `evidence/04-dashboard-rules-soar/` | Dashboards, rules, Tines, Slack, API, approval, and response evidence | 39 |

## Evidence Handling Tasks

- Retain original screenshots separately from annotated copies.
- Redact tokens, webhook secrets, credentials, and reusable public tunnel addresses.
- Preserve timestamps and timezones shown by each source.
- Add a successful connectivity test after rule removal.
- Record hashes for controlled payload files when available.
