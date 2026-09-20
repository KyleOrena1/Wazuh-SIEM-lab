# File integrity monitoring on Windows

## Goal

I configured Wazuh to monitor a dedicated Windows folder and verified that it detected when a test file was added, modified, and deleted.

## Lab setup

- Wazuh all-in-one server on an Ubuntu VM
- Windows endpoint running Wazuh agent 4.14.7
- Windows test endpoint connected to Wazuh
- Monitored folder: `C:\Wazuh-FIM-Lab`

The test was limited to a folder created specifically for this lab.

## Configuration

I created the test directory in Administrator PowerShell:

```powershell
New-Item -Path 'C:\Wazuh-FIM-Lab' -ItemType Directory
```

I then added the following entry inside the `<syscheck>` section of the Windows agent's `ossec.conf` file:

```xml
<directories check_all="yes" realtime="yes">C:\Wazuh-FIM-Lab</directories>
```

After saving the configuration, I restarted the agent and confirmed that the service was running:

```powershell
Restart-Service WazuhSvc
Get-Service WazuhSvc
```

The agent log reported that it was monitoring `c:\wazuh-fim-lab` with real-time monitoring enabled.

A reusable copy of the configuration entry is available in [`configuration/windows-fim.xml`](../configuration/windows-fim.xml).

## Test procedure

I created the file with:

```powershell
Set-Content -Path 'C:\Wazuh-FIM-Lab\test.txt' -Value 'Original lab file'
```

Once Wazuh established its baseline, I changed the file twice:

```powershell
Add-Content -Path 'C:\Wazuh-FIM-Lab\test.txt' -Value 'Modified during FIM test'
Add-Content -Path 'C:\Wazuh-FIM-Lab\test.txt' -Value 'Second change after baseline'
```

Finally, I deleted only the lab file:

```powershell
Remove-Item 'C:\Wazuh-FIM-Lab\test.txt'
```

## Investigation

In **File Integrity Monitoring > Events**, I searched for `"test.txt"` with the Windows test agent selected. Wazuh returned three events for the monitored path.

| File action | Wazuh rule ID | Alert level |
|---|---:|---:|
| Added | `554` | `5` |
| Modified | `550` | `7` |
| Deleted | `553` | `7` |

All three alerts referenced:

```text
c:\wazuh-fim-lab\test.txt
```

## Findings

The results show that the Windows agent loaded the custom directory setting and reported the test file's lifecycle to Wazuh. Real-time monitoring detected the modification and deletion without waiting for the normal scheduled scan interval.

The first observed event was classified as `added`, which established the file in Wazuh's monitored inventory. A later content change produced the `modified` event, and removing the file produced the `deleted` event.

## Outcome

This test verified that I could configure a custom FIM scope, validate the agent configuration through its log, generate controlled file activity, and investigate the resulting Wazuh alerts.

In a real investigation, unexpected changes to important files would require checking the user or process responsible, comparing hashes or file contents, reviewing nearby endpoint activity, and deciding whether containment was necessary.

## Evidence

The screenshot below shows the three-event sequence and the verified rule IDs for the modification and deletion. Host identifiers and the dashboard address were redacted before publication.

![Windows file integrity monitoring events](../screenshots/fim-lifecycle.png)
