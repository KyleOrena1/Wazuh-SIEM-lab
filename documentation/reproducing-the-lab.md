# Reproducing the lab

This guide covers the order I used to reproduce the main results in this repository. It assumes an isolated lab environment and systems you own or are authorized to test.

## Prerequisites

- Ubuntu virtual machine
- Windows 11 test endpoint
- Wazuh all-in-one deployment
- Wazuh Windows agent
- Administrator access on the Windows endpoint
- Access to the Wazuh dashboard and manager

Replace any example values with addresses and names from your own lab.

## 1. Deploy Wazuh

Install the Wazuh all-in-one components on the Ubuntu virtual machine. Confirm that the manager, indexer, and dashboard services are running before connecting an endpoint.

Open the dashboard at:

```text
https://<wazuh-manager-address>
```

## 2. Connect the Windows endpoint

In the Wazuh dashboard, open **Agents management**, create a Windows agent, and use the generated deployment command on the test endpoint.

Confirm that the agent appears as active before continuing. If the agent does not connect, check the configured manager address, the Windows service, host networking, and any firewall rules between the endpoint and manager.

## 3. Generate a failed-login event

Run the following in Administrator PowerShell:

```powershell
1..5 | ForEach-Object {
    net use \\127.0.0.1\IPC$ /user:FakeWazuhUser LabPassword123!
}
```

This uses a made-up account against the local endpoint. The attempts should fail with Windows system error 1326.

In **Threat Hunting > Events**, search for:

```text
"FakeWazuhUser"
```

Open a matching result and confirm:

- Windows event ID `4625`;
- Wazuh rule ID `60122`;
- alert level `5`; and
- the expected test username.

## 4. Configure file integrity monitoring

Create a dedicated test directory:

```powershell
New-Item -Path 'C:\Wazuh-FIM-Lab' -ItemType Directory
```

Add the configuration from [`configuration/windows-fim.xml`](../configuration/windows-fim.xml) inside the Windows agent's `<syscheck>` section, then restart the agent:

```powershell
Restart-Service WazuhSvc
Get-Service WazuhSvc
```

The service should return a status of `Running`.

## 5. Test the file lifecycle

Create, modify, and delete one file:

```powershell
Set-Content -Path 'C:\Wazuh-FIM-Lab\test.txt' -Value 'Original lab file'
Add-Content -Path 'C:\Wazuh-FIM-Lab\test.txt' -Value 'Modified during FIM test'
Remove-Item 'C:\Wazuh-FIM-Lab\test.txt'
```

In **File Integrity Monitoring > Events**, search for:

```text
"test.txt"
```

Verify the following results:

| Action | Rule ID | Level |
|---|---:|---:|
| Added | `554` | `5` |
| Modified | `550` | `7` |
| Deleted | `553` | `7` |

## 6. Add the custom correlation rule

Add [`rules/custom_failed_login_rules.xml`](../rules/custom_failed_login_rules.xml) as a custom Wazuh rule file. Reload or restart the Wazuh manager after saving the rule.

The rule looks for five matches of built-in rule `60122` against the same username within 60 seconds.

Run the failed-login test again, then search Threat Hunting for:

```text
Multiple failed Windows logins
```

Confirm that Wazuh creates one alert with:

- custom rule ID `100100`;
- alert level `10`; and
- the expected correlation description.

## Validation checklist

The lab is working as expected when:

- the Windows agent is active;
- event `4625` appears after the login test;
- rule `60122` identifies the individual failures;
- FIM reports the file being added, modified, and deleted; and
- rule `100100` creates one correlated alert after five failures.

Results can vary slightly by Wazuh version and endpoint configuration. Validate the observed fields instead of relying only on the number of search results.
