# admin-share-investigation

ADMIN$ is a built-in Windows administrative share that points to the Windows installation directory, normally C:\Windows. It is designed for remote administration and can be used by legitimate administrative tools and services.

From a DFIR/SOC perspective, ADMIN$ is interesting because attackers can also abuse administrative shares for remote execution, lateral movement, software deployment, and transferring tools.

However:

Access to ADMIN$ by itself does not prove malicious activity.

The investigation should determine:

Which account accessed the share?
What was the source system or IP address?
When did the access occur?
Was the access authenticated successfully?
What permissions or operations were requested?
Was there a related process or network connection?
Does the activity correspond to expected administration?
Are there additional indicators suggesting lateral movement?

A Windows DFIR/SOC investigation focused on the **ADMIN$ administrative share**, SMB activity, Windows Security telemetry, and endpoint correlation.

The investigation examines how an analyst can determine whether administrative share activity represents expected Windows administration or requires further investigation. The lab was performed on a Windows 11 endpoint using PowerShell, Windows SMB tooling, and Windows event telemetry.

> **Investigation principle:** Access to `ADMIN$` is not inherently malicious. The surrounding authentication, source, account, timing, process, and network evidence must be considered before drawing conclusions.

---

## Investigation Overview

| Category | Details |
|---|---|
| Investigation Type | Windows DFIR / SOC Investigation |
| Primary Artifact | `ADMIN$` administrative share |
| Protocol | SMB |
| Primary Telemetry | Windows Security Events |
| Supporting Telemetry | SMB PowerShell cmdlets, Sysmon, Wazuh |
| Host | `DESKTOP-9MMM37V` |
| Operating System | Windows 11 Pro |
| Build | `10.0.26200` |
| Domain | `WORKGROUP` |
| User | `desktop-9mmm37v\dell` |
| Workspace | `C:\ADMINShareLab\Evidence` |
| Test Date | 01 October 2026 |

---

## Lab Objectives

- Establish a controlled Windows environment for investigating administrative share activity.
- Identify the purpose and normal configuration of the `ADMIN$` administrative share.
- Verify whether `ADMIN$` is enabled and determine the directory it exposes.
- Examine the configured permissions associated with the administrative share.
- Establish the investigating user, hostname, operating system, and network baseline.
- Review existing SMB connections, sessions, and open files before generating test activity.
- Perform a controlled read-only access to `ADMIN$` and record the exact test timestamp.
- Determine whether the controlled access produces observable Windows Security telemetry.
- Investigate Security Event ID `5140` for network share access.
- Investigate Security Event ID `5145` for detailed network share access information.
- Review Security Event ID `4624` and determine whether any authentication events can be reliably correlated with the test.
- Examine SMB state after the controlled access and document whether any persistent session remains visible.
- Correlate available account, timestamp, source address, share, and process information.
- Use Sysmon and Wazuh as supporting telemetry sources where applicable.
- Distinguish successful `ADMIN$` access from evidence of malicious remote administration or lateral movement.
- Document telemetry gaps and avoid interpreting missing events as proof that an activity did not occur.
- Develop an evidence-based assessment of the observed administrative share activity.
  
---

## Lab Scenario

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

## Environment

### Host

```text
Hostname: DESKTOP-9MMM37V
Manufacturer: Dell Inc.
Model: Latitude 5420
Operating System: Windows 11 Pro
Version: 10.0.26200
Build: 26200
Domain: WORKGROUP
Domain Role: 0
User: desktop-9mmm37v\dell
```

### Network Baseline

The host had the following IPv4 interfaces during the investigation:

```text
Wi-Fi                         192.168.1.6/24
VMware Network Adapter VMnet8 192.168.203.1/24
VMware Network Adapter VMnet1 192.168.174.1/24
```

The Wi-Fi address was the primary physical network address observed during the baseline.

---

## Investigation Workflow

1. Create the investigation workspace.
2. Collect host and operating-system information.
3. Verify the `ADMIN$` share.
4. Inspect the share configuration.
5. Review `ADMIN$` permissions.
6. Identify the test account.
7. Record network configuration.
8. Check existing SMB connections and sessions.
9. Access `ADMIN$\System32` in a controlled manner.
10. Record the test timestamp.
11. Re-check SMB sessions and open files.
12. Search for Security Event ID 5140.
13. Search for Security Event ID 5145.
14. Review available Event ID 4624 records.
15. Review supporting endpoint telemetry.
16. Document the resulting evidence and limitations.

---

## ADMIN$ Configuration

The endpoint reported the following administrative share:

```text
Name        : ADMIN$
Path        : C:\WINDOWS
Description : Remote Admin
ShareState  : Online
ShareType   : FileSystemDirectory
CurrentUsers: 0
```

The share was therefore present and online at the time of investigation.

---

## ADMIN$ Permissions

`Get-SmbShareAccess -Name ADMIN$` returned administrative share permissions including:

```text
BUILTIN\Administrators
BUILTIN\Backup Operators
NT AUTHORITY\INTERACTIVE
```

The observed access right was `Full` with an `Allow` access control type.

These permissions describe the configured share access and are not, by themselves, evidence of compromise.

---

## Controlled Access

The following operation was performed:

```powershell
Get-ChildItem "\\localhost\ADMIN$\System32" |
    Select-Object -First 10 Name, Length, LastWriteTime
```

The command successfully returned directory contents from the Windows `System32` directory.

The test was therefore able to access the `ADMIN$` share locally.

No file modification, deletion, or execution through the share was performed as part of the controlled test.

---

## SMB Session Results

The following commands were used:

```powershell
Get-SmbConnection
Get-SmbSession
Get-SmbOpenFile
```

No active SMB connection, SMB session, or open SMB file was returned after the controlled local access.

This does not mean that no SMB-related operation occurred internally. It means that no persistent/current session or open-file state was visible through these commands at the time they were executed.

---

## Windows Security Event Results

### Event ID 5140

A search for Security Event ID `5140` returned:

```text
No events were found that match the specified selection criteria.
```

This result was obtained both when searching the recent time window and when checking the Security log for Event ID 5140 more generally.

### Event ID 5145

A search for Security Event ID `5145` also returned:

```text
No events were found that match the specified selection criteria.
```

Therefore, the investigation did not obtain direct Windows Security telemetry confirming the controlled `ADMIN$` access through Events 5140 or 5145.

---

## Event ID 4624

Event ID `4624` records were present on the endpoint.

The available records were concentrated around:

```text
28 September 2026 07:21:13
through
28 September 2026 07:21:30
```

The controlled `ADMIN$` test occurred on:

```text
01 October 2026 08:05:26
```

Because the available 4624 records predated the controlled test by several days, they were not treated as evidence of the `ADMIN$` access performed during this investigation.

This is an important correlation principle:

> An authentication event should not be associated with a share-access event solely because both are Event ID 4624 records. The timestamps and other identifying fields must support the relationship.

---

## Evidence Assessment

| Evidence | Result | Interpretation |
|---|---|---|
| `ADMIN$` exists | Confirmed | Normal Windows administrative share |
| `ADMIN$` path | `C:\WINDOWS` | Expected configuration |
| Share state | Online | Share was available |
| Share permissions | Administrators / Backup Operators / Interactive | Configured access permissions |
| Local `ADMIN$` access | Confirmed | Controlled test succeeded |
| SMB connection after test | None observed | No persistent connection visible |
| SMB session after test | None observed | No active session visible |
| SMB open file | None observed | No open SMB file visible |
| Security 5140 | Not observed | Direct share-access telemetry unavailable |
| Security 5145 | Not observed | Detailed share-access telemetry unavailable |
| Security 4624 | Present, but older | Not correlated with this test |
| Malicious follow-on activity | Not established | No supporting evidence collected |

---

