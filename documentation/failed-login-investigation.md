# Investigating failed Windows logins with Wazuh

## Goal

I wanted to verify that my Windows endpoint could send failed-login events to Wazuh and that I could trace an alert back to the activity that caused it.

## Lab setup

- Wazuh all-in-one server on an Ubuntu VM
- Windows endpoint running Wazuh agent 4.14.7
- Agent name: `Kyle-Windows-PC` (ID `001`)

This was a controlled test on my own computer, not an attack against another system.

## Test

I ran the following command in Administrator PowerShell:

```powershell
1..5 | ForEach-Object { net use \\127.0.0.1\IPC$ /user:FakeWazuhUser WrongPassword123! }
```

The command attempted five connections to the local computer using a made-up username and test password. Each attempt returned system error 1326: the username or password was incorrect. The credentials above are test values, not real account credentials.

## Investigation

In Wazuh Threat Hunting, I opened the Events tab for agent `001`, selected the last 24 hours, and searched for:

```text
"FakeWazuhUser"
```

The search returned 10 matching records across two clusters of five. The captured PowerShell run shows five attempts, so I did not treat all 10 records as evidence of a single execution. I opened one matching alert and checked the account, source address, Windows event, and Wazuh rule.

| Field | Observed value |
|---|---|
| Alert timestamp | September 14, 2026, 13:02:27.144, as displayed in the dashboard |
| Agent | `Kyle-Windows-PC` / `001` |
| Target username | `FakeWazuhUser` |
| Source address | `127.0.0.1` |
| Authentication package | `NTLM` |
| Logon type | `3` |
| Windows channel | `Security` |
| Windows event ID | `4625` |
| Event message | An account failed to log on. |
| Status / substatus | `0xc000006d` / `0xc0000064` |
| Wazuh rule ID | `60122` |
| Wazuh alert level | `5` |
| Rule description | Logon Failure - Unknown user or bad password |
| Rule groups | `windows`, `windows_security`, `authentication_failed` |

## Findings

The alert identifies the same test username and loopback address used in the command. Together with the failed-logon message, this connects the observed alert to the local authentication test.

Wazuh collected the Windows Security event and applied its built-in rule `60122`. I did not create a custom rule for this test, and this result does not demonstrate a separate brute-force correlation alert.

The rule also included MITRE ATT&CK metadata for `T1531`, Account Access Removal. I recorded that as rule metadata rather than evidence that an account was removed or its access changed. The activity verified here was an unsuccessful login attempt.

## Outcome

I verified the path from a controlled Windows login attempt to an alert in Wazuh, then used the alert fields to explain what happened. No successful access was demonstrated, and no containment action was needed for this planned test.

For an unexpected alert like this, my next checks would be the source host, surrounding login activity, whether the target account exists, and any successful logins near the same time.

## Evidence handling

The findings were checked against the PowerShell output and Wazuh event-detail screenshots. Screenshots are not embedded in this write-up yet. Before publishing evidence, I will remove credentials and unnecessary device identifiers.
