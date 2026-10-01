# Lab Scenario

A Windows endpoint is being investigated to understand how activity involving the built-in `ADMIN$` administrative share can be identified and analyzed during a DFIR investigation. The share is a normal Windows feature used for administrative operations, but similar mechanisms can also appear during remote administration, lateral movement, or remote execution.

The investigation is performed on a controlled Windows endpoint using PowerShell and Windows SMB functionality. The analyst first establishes the host and network baseline, verifies the `ADMIN$` configuration, reviews its permissions, and checks for existing SMB activity before generating a controlled test event.

The investigation will focus on:

- Verifying that `ADMIN$` is available and identifying its underlying Windows directory.
- Reviewing the accounts and groups permitted to access the share.
- Establishing the user and network context of the investigation.
- Performing a controlled read-only access to `ADMIN$`.
- Recording the exact time of the test for subsequent event correlation.
- Checking Windows Security telemetry for share-access and authentication events.
- Reviewing SMB session and connection state before and after the test.
- Using available Sysmon and Wazuh telemetry to look for supporting evidence.

The analyst must distinguish between **administrative-share functionality and suspicious activity**. The presence of `ADMIN$`, configured administrative permissions, or a successful controlled access does not by itself indicate compromise. Stronger conclusions require correlation between the account, source address, authentication activity, share access, process execution, and any subsequent network or file activity.

The final assessment will document what was directly observed, which telemetry was available or missing, whether the controlled activity could be correlated with Windows Security events, and whether the collected evidence supports any indication of suspicious administrative-share usage.
