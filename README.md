# ADMIN$ Share Investigation

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

## Concept

`ADMIN$` is a built-in Windows administrative share that normally maps to the Windows installation directory, usually `C:\Windows`.

It is intended for remote administration and can be used by legitimate administrative tools and services. The same mechanism can also appear during attacker activity involving remote administration, lateral movement, remote execution, or file transfer.

Therefore, an `ADMIN$` access event should be treated as an **investigation lead rather than proof of malicious activity**.

The investigation focuses on:

- Share configuration
- Share permissions
- SMB sessions and connections
- Successful network authentication
- Windows Security Event ID 5140
- Windows Security Event ID 5145
- Related process and network telemetry
- Wazuh endpoint correlation
- Timeline reconstruction
- Telemetry limitations

---

## Objectives

- Verify the presence and configuration of the `ADMIN$` share.
- Identify the directory associated with the share.
- Examine the configured share permissions.
- Establish the investigating user and host network baseline.
- Perform controlled access to `ADMIN$`.
- Determine whether the controlled access creates SMB session evidence.
- Search for Windows Security Event ID 5140.
- Search for Windows Security Event ID 5145.
- Review Event ID 4624 for potentially related network authentication.
- Examine supporting Sysmon and Wazuh telemetry where available.
- Correlate timestamps, accounts, source addresses, and activity.
- Document telemetry gaps instead of treating missing events as proof that activity did not occur.
- Assess the observed activity using the available evidence.

---

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

## Final Assessment

The investigation confirmed that `ADMIN$` is enabled and accessible on `DESKTOP-9MMM37V`, and a controlled local read operation successfully accessed `ADMIN$\System32`. However, the available evidence did not provide Security Event ID 5140 or 5145 records corresponding to the test, and the available 4624 events were from an earlier date. No persistent SMB session or open SMB file was visible after the test. Therefore, the lab demonstrates the mechanics and investigative limitations of `ADMIN$` activity, but the collected telemetry does not establish suspicious remote administrative activity or lateral movement.

---

## Evidence Files

```text
C:\ADMINShareLab\Evidence\
├── Host-Baseline.txt
├── OS-Baseline.txt
├── ADMINShare-Permissions.txt
├── ADMINShare-Configuration.txt
├── SMB-Connections.txt
├── SMB-Sessions.txt
├── Network-Baseline.txt
├── Test-User.txt
├── ADMINShare-Test-Time.txt
├── Security-5140.txt
└── Security-5145.txt
```

---

## DFIR Lesson

`ADMIN$` is an example of why a single artifact should not be interpreted in isolation.

A stronger investigation would ideally correlate:

```text
Authentication
      ↓
Source Address
      ↓
ADMIN$ Access
      ↓
File / Share Activity
      ↓
Process Creation
      ↓
Network Activity
      ↓
Remote Execution / Follow-on Activity
```

The absence of one telemetry source should be documented as a visibility limitation rather than converted into an assumption about what did or did not happen.
