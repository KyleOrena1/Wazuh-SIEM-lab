# Failed-login response runbook

## Purpose

This runbook outlines how I would investigate and respond to repeated failed Windows logins detected by Wazuh. It is written for the custom lab rule `100100`, but the same process can be used for unexpected Windows event `4625` alerts.

## Alert criteria

Start this process when either of the following occurs:

- custom rule `100100` detects five failed logins against the same username within 60 seconds;
- built-in rule `60122` appears repeatedly for the same account, endpoint, or source; or
- failed logins occur alongside a successful login or other suspicious endpoint activity.

## 1. Validate the alert

Confirm that the event is current and that the affected endpoint is still reporting to Wazuh. Record the following details:

- alert timestamp and time range;
- agent name and ID;
- target username;
- source address and workstation;
- Windows event ID;
- Wazuh rule ID and level;
- authentication package and logon type;
- status and substatus codes.

Check whether the activity came from an approved test, a known service, or a user who recently changed a password. Do not close the alert based only on the username.

## 2. Review surrounding activity

Search the same endpoint, username, and source address before and after the alert. Look for:

- additional event `4625` failures;
- event `4624` successful logins;
- account lockouts or password changes;
- new processes, services, or scheduled tasks;
- unexpected file or registry changes;
- similar activity on other endpoints.

A successful login shortly after repeated failures raises the priority because it may indicate that an account was guessed or compromised.

## 3. Determine severity

Treat the event as lower risk when it is tied to a documented test or a confirmed user mistake and no related suspicious activity is present.

Escalate the event when:

- the source is unknown;
- a privileged or sensitive account is targeted;
- failures affect several accounts or endpoints;
- a successful login follows the failures;
- the activity continues after the user denies making the attempts; or
- other endpoint alerts appear in the same period.

## 4. Contain the activity

Containment depends on the evidence and the organization's approval process. Possible actions include:

- disabling or resetting the affected account;
- revoking active sessions;
- isolating the endpoint;
- blocking the source address;
- preserving relevant logs and system evidence; and
- notifying the user, help desk, or incident-response lead.

For this lab test, containment was not required because the activity was generated locally with a made-up account.

## 5. Recover and monitor

If the activity is confirmed as malicious, remove any persistence or malware, patch the affected system, and restore account access only after validation. Continue monitoring the username, source, and endpoint for repeated failures or successful access.

## 6. Document and close

The case record should include:

- what triggered the alert;
- the timeline and affected systems;
- the evidence reviewed;
- the reason for the final classification;
- any containment or recovery actions;
- whether the rule needs tuning; and
- follow-up work assigned to another team or user.

Close the alert only when the activity is explained, the risk is addressed, and the supporting evidence is recorded.
