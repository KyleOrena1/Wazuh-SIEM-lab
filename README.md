# Wazuh SIEM Lab

I built this lab to practice collecting Windows security events, configuring endpoint monitoring, writing detection logic, and investigating the activity behind alerts.

## Architecture

```mermaid
flowchart LR
    A["Windows 11 endpoint"] -->|"Security events and FIM"| B["Wazuh manager"]
    B --> C["Built-in and custom rules"]
    C --> D["Wazuh indexer"]
    D --> E["Dashboard and investigation"]
```

The Windows endpoint ran the Wazuh agent, while the manager, indexer, and dashboard were deployed together on an Ubuntu virtual machine hosted with Microsoft Hyper-V.

## What I completed

- Deployed a Wazuh all-in-one server on an Ubuntu virtual machine.
- Connected a Windows 11 endpoint with Wazuh agent 4.14.7.
- Generated controlled failed-login activity and traced it to Windows event `4625`.
- Investigated the matching built-in Wazuh alert and rule `60122`.
- Configured real-time file integrity monitoring for a dedicated Windows folder.
- Detected a test file being added, modified, and deleted.
- Created and tested a custom correlation rule for repeated failed logins.
- Confirmed that five failures against the same username within 60 seconds generated custom rule `100100` at level `10`.
- Wrote a practical response runbook for investigating repeated failed logins.

The planned deployment, detection, validation, and documentation work for this lab is complete.

## Investigations

- [Failed Windows login investigation](documentation/failed-login-investigation.md) — Traced a controlled login test to Windows event `4625` and Wazuh rule `60122`.
- [Windows file integrity monitoring](documentation/file-integrity-monitoring.md) — Configured a custom FIM directory and verified added, modified, and deleted events.
- [Custom failed-login correlation](documentation/custom-failed-login-correlation.md) — Correlated five failures against the same username into one level-10 alert.
- [Failed-login response runbook](documentation/failed-login-response-runbook.md) — Covers validation, escalation, containment, recovery, and case closure.
- [Reproducing the lab](documentation/reproducing-the-lab.md) — Lists the setup order, test commands, and validation checks.

## Detection and configuration files

- [Custom failed-login rule](rules/custom_failed_login_rules.xml) — Tested correlation rule `100100`.
- [Windows FIM configuration](configuration/windows-fim.xml) — Real-time monitoring entry for the dedicated lab folder.

## Evidence

- [Failed-login event details](screenshots/failed-login-event.png)
- [Built-in rule 60122 details](screenshots/failed-login-rule-details.png)
- [File integrity monitoring lifecycle](screenshots/fim-lifecycle.png)
- [Custom correlation alert](screenshots/custom-correlation-alert.png)

The screenshots were redacted to remove host identifiers and addressing details while keeping the relevant alert fields visible.

## Skills demonstrated

- Windows Security event analysis
- Wazuh agent and manager configuration
- File integrity monitoring
- Custom rule development and correlation
- Alert validation and incident documentation

## Lessons learned

- A useful investigation starts with the raw event fields, not only the alert title.
- Individual failed logins and a correlated burst of failures represent different detection cases.
- MITRE ATT&CK data attached to a rule provides context, but it does not prove that the mapped technique occurred.
- File integrity monitoring must establish a baseline before later changes can be interpreted correctly.
- Detection documentation should explain both what was observed and what the evidence does not establish.

## Lab environment

- Wazuh all-in-one deployment
- Ubuntu virtual machine hosted with Microsoft Hyper-V
- Windows 11 endpoint with Wazuh agent 4.14.7
- PowerShell for controlled event generation

## Repository structure

- `configuration/` — Reusable agent configuration examples
- `documentation/` — Investigation notes and response guidance
- `rules/` — Custom Wazuh detection rules
- `screenshots/` — Redacted evidence from completed tests

## Security notice

Tests are limited to systems I own or am authorized to use. Passwords and unnecessary personal or device information are removed before evidence is published. Credentials shown in test commands are dummy values, not real account credentials.
