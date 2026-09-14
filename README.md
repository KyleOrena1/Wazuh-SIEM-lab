# Wazuh SIEM Lab

I built this lab to practice collecting Windows security events, finding alerts in Wazuh, and investigating the activity behind them.

## Progress

- Installed the Wazuh all-in-one server on an Ubuntu VM.
- Connected a Windows endpoint and verified its agent was active.
- Generated failed login attempts with a test username.
- Found the matching alerts and reviewed the Windows event and Wazuh rule details.

The lab is still in progress. Custom detection rules and additional investigations have not been completed yet.

## Investigations

- [Failed Windows login investigation](documentation/failed-login-investigation.md) — Traced a local login test to Windows event `4625` and Wazuh rule `60122`.

## Lab Environment

- Wazuh all-in-one deployment
- Ubuntu virtual machine
- Oracle VirtualBox
- Windows host system with Wazuh agent 4.14.7

## Repository Structure

- `documentation/` — Lab notes and investigation reports
- `rules/` — Reserved for custom rules as they are developed and tested
- `screenshots/` — Reserved for sanitized lab evidence

## Security Notice

Tests are limited to systems I own or am authorized to use. Passwords and unnecessary personal or device information are removed before evidence is published. Credentials shown in test commands are dummy values, not real account credentials.
