# Wazuh SIEM Lab

I built this lab to practice collecting Windows security events, configuring endpoint monitoring, writing detection logic, and investigating the activity behind alerts.

## What I completed

- Deployed a Wazuh all-in-one server on an Ubuntu virtual machine.
- Connected a Windows endpoint with Wazuh agent 4.14.7.
- Generated controlled failed-login activity and traced it to Windows event `4625`.
- Investigated the matching built-in Wazuh alert and rule `60122`.
- Configured real-time file integrity monitoring for a dedicated Windows folder.
- Detected a test file being added, modified, and deleted.
- Created and tested a custom correlation rule for repeated failed logins.
- Confirmed that five failures against the same username within 60 seconds generated custom rule `100100` at level `10`.

The core lab and detection tests are complete. Remaining work is presentation-focused: adding sanitized screenshots and a concise incident-response runbook.

## Investigations

- [Failed Windows login investigation](documentation/failed-login-investigation.md) — Traced a controlled login test to Windows event `4625` and Wazuh rule `60122`.
- [Windows file integrity monitoring](documentation/file-integrity-monitoring.md) — Configured a custom FIM directory and verified added, modified, and deleted events.
- [Custom failed-login correlation](documentation/custom-failed-login-correlation.md) — Correlated five failures against the same username into one level-10 alert.

## Detection and configuration files

- [Custom failed-login rule](rules/custom_failed_login_rules.xml) — Tested correlation rule `100100`.
- [Windows FIM configuration](configuration/windows-fim.xml) — Real-time monitoring entry for the dedicated lab folder.

## Lab environment

- Wazuh all-in-one deployment
- Ubuntu virtual machine hosted with Microsoft Hyper-V
- Windows 11 endpoint with Wazuh agent 4.14.7
- PowerShell for controlled event generation

## Repository structure

- `configuration/` — Reusable agent configuration examples
- `documentation/` — Investigation notes and findings
- `rules/` — Custom Wazuh detection rules
- `screenshots/` — Reserved for sanitized lab evidence

## Security notice

Tests are limited to systems I own or am authorized to use. Passwords and unnecessary personal or device information are removed before evidence is published. Credentials shown in test commands are dummy values, not real account credentials.
