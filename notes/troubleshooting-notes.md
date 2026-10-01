# Troubleshooting Notes — ADMIN$ Share Investigation

## 1. Event ID 5140 Returned No Results

### Query

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id      = 5140
}
```

### Result

```text
Get-WinEvent: No events were found that match the specified selection criteria.
```

### Interpretation

This does not prove that `ADMIN$` was not accessed.

The controlled test successfully accessed:

```text
\\localhost\ADMIN$\System32
```

The absence of Event ID 5140 indicates that the expected share-access telemetry was not available in the queried Security log.

Possible explanations include:

- Object Access auditing configuration
- Advanced Audit Policy configuration
- Event collection limitations
- Event retention
- The specific access path not generating the expected event under the current configuration

The correct DFIR approach is to document the limitation.

---

## 2. Event ID 5145 Returned No Results

### Query

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id      = 5145
}
```

### Result

```text
Get-WinEvent: No events were found that match the specified selection criteria.
```

### Interpretation

The detailed network-share access event was not available.

Therefore, fields such as:

```text
Share Name
Relative Target Name
Source Address
Access Mask
Access Check Results
```

could not be used to reconstruct the controlled access.

---

## 3. Event ID 4624 Was Available but Not Correlated

Event ID `4624` records were present, but the observed records were from:

```text
28 September 2026 07:21:13
to
28 September 2026 07:21:30
```

The controlled `ADMIN$` access occurred on:

```text
01 October 2026 08:05:26
```

Therefore, the available 4624 records were not treated as evidence of the test.

### Lesson

Do not correlate events simply because they have the same event ID.

Use:

```text
Timestamp
Account
Logon Type
Source Address
Logon ID
Related Activity
```

to establish a defensible relationship.

---

## 4. SMB Session Commands Returned Nothing

The following commands returned no active results:

```powershell
Get-SmbConnection
Get-SmbSession
Get-SmbOpenFile
```

This occurred even though the following command successfully accessed the administrative share:

```powershell
Get-ChildItem "\\localhost\ADMIN$\System32"
```

### Interpretation

The commands report current or observable SMB state.

They should not be treated as a complete historical record of every SMB operation.

A short-lived local access can complete without leaving a persistent session visible when the follow-up commands are executed.

---

## 5. ADMIN$ Access Worked

The controlled command:

```powershell
Get-ChildItem "\\localhost\ADMIN$\System32"
```

successfully returned directory contents.

This confirms:

```text
ADMIN$ exists
+
ADMIN$ is accessible
+
The test account could read the share
```

It does not establish:

```text
Remote access
Malicious access
Lateral movement
Remote execution
Compromise
```

Additional evidence would be required for those conclusions.

---

## 6. ADMIN$ Permissions Look Broad

The share permissions included:

```text
BUILTIN\Administrators
BUILTIN\Backup Operators
NT AUTHORITY\INTERACTIVE
```

with `Allow / Full` access observed.

### Interpretation

These are configured share permissions and should be interpreted in the context of Windows administrative-share functionality.

The permission configuration alone is not evidence that these identities actually accessed the share.

---

## 7. Network Baseline Included VMware Adapters

The host had:

```text
192.168.1.6
192.168.203.1
192.168.174.1
```

The latter two addresses belong to VMware virtual network adapters.

### Investigation Lesson

When investigating source or destination addresses, distinguish:

```text
Physical network interface
Virtualization interfaces
Loopback
Link-local addresses
Other local interfaces
```

before assigning meaning to an IP address.

---

## 8. Localhost Access Versus Remote Access

The controlled operation used:

```text
\\localhost\ADMIN$
```

This is different from investigating access such as:

```text
\\REMOTE-HOST\ADMIN$
```

or an inbound SMB connection from another machine.

Therefore, this lab demonstrates the administrative-share mechanism and telemetry investigation, but it does not by itself establish remote lateral movement.

---

## 9. Telemetry Absence

The following telemetry was not obtained:

```text
Security 5140
Security 5145
Correlated 4624 for the test
Persistent SMB session
Persistent SMB open-file state
```

These gaps should remain explicitly documented.

### Incorrect conclusion

```text
No 5140 = ADMIN$ was never accessed
```

### Evidence-based conclusion

```text
ADMIN$ access was successfully performed,
but the expected 5140/5145 share-access telemetry
was not available in the collected Security log.
```

---

## 10. Recommended Future Improvement

For a future version of the lab, Windows Advanced Audit Policy can be configured before generating the controlled activity so that network-share auditing is intentionally enabled.

The investigation could then compare:

```text
ADMIN$ access
        ↓
5140
        ↓
5145
        ↓
4624
        ↓
Source Address
        ↓
Account
        ↓
Process / Network Activity
```

This would provide stronger end-to-end telemetry correlation.

---

## 11. Core Troubleshooting Lesson

A DFIR investigation should distinguish between:

```text
Activity did not happen
```

and:

```text
Activity happened but the available telemetry did not capture it
```

In this lab, the second interpretation is better supported because the controlled `ADMIN$` access was directly observed through PowerShell even though the corresponding Security share-access events were unavailable.
