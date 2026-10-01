# Timeline — ADMIN$ Share Investigation

| Date / Time | Evidence | Observation | Assessment |
|---|---|---|---|
| 28-09-2026 07:39:52 | Windows OS | Last boot time reported by the endpoint | Host baseline |
| 01-10-2026 07:47 | PowerShell | `C:\ADMINShareLab\Evidence` created | Investigation workspace established |
| 01-10-2026 | Win32_ComputerSystem | `DESKTOP-9MMM37V`, Dell Latitude 5420, WORKGROUP | Host identity established |
| 01-10-2026 | Win32_OperatingSystem | Windows 11 Pro, build 26200 | OS baseline established |
| 01-10-2026 | Get-SmbShare | `ADMIN$` mapped to `C:\WINDOWS` | Administrative share confirmed |
| 01-10-2026 | Get-SmbShare | `ADMIN$` state reported as Online | Share available |
| 01-10-2026 | Get-SmbShareAccess | Administrators, Backup Operators, and Interactive permissions observed | Share permissions documented |
| 01-10-2026 | Get-SmbConnection | No active connection observed during baseline check | No persistent SMB connection visible |
| 01-10-2026 | Get-SmbSession | No active SMB session observed | No persistent SMB session visible |
| 01-10-2026 | Get-SmbOpenFile | No open SMB file observed | No persistent SMB file state visible |
| 01-10-2026 08:05:26 | PowerShell | `\\localhost\ADMIN$\System32` successfully accessed | Controlled ADMIN$ access confirmed |
| 01-10-2026 08:05:26 | PowerShell | System32 directory contents returned | Read access confirmed |
| 01-10-2026 08:05:26 | Get-Date | Test timestamp recorded | Correlation reference established |
| 01-10-2026 | Get-SmbConnection | No active connection observed after test | No persistent connection visible |
| 01-10-2026 | Get-SmbSession | No active session observed after test | No persistent session visible |
| 01-10-2026 | Get-SmbOpenFile | No open SMB file observed after test | No persistent file state visible |
| 01-10-2026 | Security Event 5140 | No matching events found | Share-access telemetry unavailable |
| 01-10-2026 | Security Event 5145 | No matching events found | Detailed share-access telemetry unavailable |
| 28-09-2026 07:21:13–07:21:30 | Security Event 4624 | Multiple successful logon events available | Events predated the ADMIN$ test and were not correlated |
| 01-10-2026 | Investigation review | No supporting evidence of malicious follow-on activity established | Suspicious activity not established |

---

## Key Timeline Relationship

```text
ADMIN$ Share Exists
        ↓
Share Configuration Reviewed
        ↓
Share Permissions Reviewed
        ↓
Controlled Local ADMIN$ Access
        ↓
System32 Contents Returned
        ↓
No Persistent SMB Session Observed
        ↓
No Persistent SMB Open File Observed
        ↓
5140 Not Available
        ↓
5145 Not Available
        ↓
4624 Records Available From Earlier Date
        ↓
No Reliable Authentication Correlation
        ↓
No Malicious Activity Established
```

---

## Timeline Assessment

The timeline confirms that the `ADMIN$` share was present and that a controlled local read operation successfully accessed it on 01 October 2026 at approximately 08:05:26. The expected Windows Security share-access events were not available, while the available 4624 events occurred on 28 September and could not be reliably associated with the test. The investigation therefore establishes the share access itself but does not establish suspicious remote administration, lateral movement, or remote execution.
