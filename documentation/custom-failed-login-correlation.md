# Custom failed-login correlation rule

## Goal

The built-in Windows rule identified individual failed logins, but I wanted one higher-priority alert when several failures targeted the same username in a short period. I created and tested a custom Wazuh correlation rule for that behavior.

## Detection logic

The custom rule is stored in [`rules/custom_failed_login_rules.xml`](../rules/custom_failed_login_rules.xml).

It uses built-in Wazuh rule `60122` as its starting point. That rule had already detected the Windows event ID `4625` activity from my controlled failed-login test.

The custom rule triggers when:

- rule `60122` matches five times;
- the matches occur within 60 seconds; and
- the target username is the same across those matches.

The resulting alert uses custom rule ID `100100`, level `10`, and MITRE ATT&CK technique `T1110` (Brute Force).

## Rule

```xml
<group name="windows,windows_security,authentication_failed,">
  <rule id="100100" level="10" frequency="5" timeframe="60">
    <if_matched_sid>60122</if_matched_sid>
    <same_field>win.eventdata.targetUserName</same_field>
    <description>Multiple failed Windows logins for the same user within 60 seconds</description>
    <mitre>
      <id>T1110</id>
    </mitre>
  </rule>
</group>
```

## Test procedure

I generated five failed local authentication attempts in Administrator PowerShell:

```powershell
1..5 | ForEach-Object { net use \\127.0.0.1\IPC$ /user:FakeWazuhUser WrongPassword123! }
```

The command used a made-up account and test password on my own endpoint. Each attempt returned Windows system error 1326 because the username or password was incorrect.

## Verified result

After reloading the Wazuh manager, I searched Threat Hunting for the custom rule description. Wazuh returned one correlated alert with the following verified values:

| Field | Result |
|---|---|
| Agent | `Kyle-Windows-PC` |
| Custom rule ID | `100100` |
| Alert level | `10` |
| Description | Multiple failed Windows logins for the same user within 60 seconds |
| Correlation threshold | Five matches in 60 seconds |
| Parent rule | `60122` |
| Test username | `FakeWazuhUser` |
| MITRE ATT&CK mapping | `T1110` — Brute Force |

## Findings

The custom rule converted several related low-level authentication failures into one higher-severity alert. Requiring the same target username makes the detection more specific than simply counting unrelated login failures across the endpoint.

This was a controlled validation, not evidence of an actual brute-force attack. In a real investigation, I would review the source system, target account, timing, surrounding successful logins, and other endpoint activity before deciding whether to contain the host or disable an account.

## Outcome

The test verified that I could build a correlation rule on top of an existing Wazuh rule, reload the ruleset, generate matching activity, and confirm the new alert by its custom rule ID and severity.
