# Wazuh SIEM Lab

I built this lab to practice collecting Windows security events, configuring endpoint monitoring, finding alerts in Wazuh, and investigating the activity behind them.

## Progress

- Installed the Wazuh all-in-one server on an Ubuntu VM.
- Connected a Windows endpoint and verified its agent was active.
- Generated failed login attempts with a test username and traced them to Windows event `4625`.
- Configured real-time file integrity monitoring for a custom Windows folder.
- Detected a test file being added, modified, and deleted.
- Reviewed the alert fields and documented the findings from both tests.

The lab is still in progress. A custom detection rule and additional investigation scenario have not been completed yet.

## Investigations

- [Failed Windows login investigation](documentation/failed-login-investigation.md) — Traced a controlled login test to Windows event `4625` and Wazuh rule `60122`.
- [Windows file integrity monitoring](documentation/file-integrity-monitoring.md) — Configured a custom FIM directory and verified added, modified, and deleted file events.

## Configuration

- [Windows FIM configuration](configuration/windows-fim.xml) — Real-time monitoring entry used for the dedicated lab folder.

## Lab environment

- Wazuh all-in-one deployment
- Ubuntu virtual machine
- Oracle VirtualBox
- Windows host system with Wazuh agent 4.14.7

## Repository structure

- `configuration/` — Reusable configuration examples from the lab
- `documentation/` — Investigation notes and findings
- `rules/` — Reserved for custom rules after they are developed and tested
- `screenshots/` — Reserved for sanitized lab evidence

## Security notice

Tests are limited to systems I own or am authorized to use. Passwords and unnecessary personal or device information are removed before evidence is published. Credentials shown in test commands are dummy values, not real account credentials.
