# Investigation Notes

## 1. Investigation Setup

The investigation workspace was created at:

```text
C:\ADMINShareLab\Evidence
```

Workspace creation was recorded on:

```text
01 October 2026 07:47
```

The investigation was performed against:

```text
DESKTOP-9MMM37V
```

---

## 2. Host Baseline

The host was identified as:

```text
Hostname     : DESKTOP-9MMM37V
Manufacturer : Dell Inc.
Model        : Latitude 5420
Domain       : WORKGROUP
Domain Role  : 0
```

Operating system:

```text
Windows 11 Pro
Version: 10.0.26200
Build: 26200
```

Last boot time reported by the operating system:

```text
28 September 2026 07:39:52
```

The currently logged-in user was:

```text
desktop-9mmm37v\dell
```

---

## 3. ADMIN$ Share Verification

The host reported the following SMB shares:

```text
ADMIN$    C:\WINDOWS    Remote Admin
C$        C:\           Default share
D$        D:\           Default share
IPC$                     Remote IPC
```

The `ADMIN$` share was specifically inspected.

Observed configuration:

```text
Name                 : ADMIN$
Path                 : C:\WINDOWS
Description          : Remote Admin
ShareState            : Online
ShareType             : FileSystemDirectory
CurrentUsers          : 0
FolderEnumerationMode : Unrestricted
CachingMode           : Manual
LeasingMode           : Full
EncryptData           : False
```

The presence of `ADMIN$` is expected Windows functionality and was not treated as suspicious by itself.

---

## 4. Share Permissions

The command:

```powershell
Get-SmbShareAccess -Name ADMIN$
```

returned access entries including:

```text
BUILTIN\Administrators
BUILTIN\Backup Operators
NT AUTHORITY\INTERACTIVE
```

The observed entries had:

```text
AccessControlType : Allow
AccessRight       : Full
```

These entries describe the share configuration. They do not establish that the accounts were actively using the share.

---

## 5. Network Baseline

IPv4 addresses observed during the investigation included:

```text
Wi-Fi                         192.168.1.6
VMware Network Adapter VMnet8 192.168.203.1
VMware Network Adapter VMnet1 192.168.174.1
```

The presence of VMware virtual network adapters reflects the local virtualization environment.

---

## 6. Existing SMB Activity

The following commands were executed before and after the controlled test:

```powershell
Get-SmbConnection
Get-SmbSession
Get-SmbOpenFile
```

No active SMB connections, sessions, or open files were returned at the points where these commands were checked.

This is not interpreted as proof that no SMB operation occurred. These commands show current/persistent SMB state rather than providing a complete historical audit trail.

---

## 7. Controlled ADMIN$ Access

A controlled local read operation was performed:

```powershell
Get-ChildItem "\\localhost\ADMIN$\System32" |
    Select-Object -First 10 Name, Length, LastWriteTime
```

The command successfully returned files and directories from `System32`.

Examples included:

```text
0409
AccountHealthAssets
Advanced Installers
af-ZA
am-ET
AppLocker
appmgmt
appraiser
AppV
ar-SA
```

This confirmed that the local account was able to access the `ADMIN$` share.

No file modification, deletion, or execution was performed.

---

## 8. Test Timestamp

The controlled access was followed by:

```powershell
Get-Date
```

Observed timestamp:

```text
01 October 2026 08:05:26
```

This timestamp was used as the primary reference point when considering event correlation.

---

## 9. Security Event ID 5140

The Security log was queried for Event ID `5140`.

Result:

```text
No events were found that match the specified selection criteria.
```

The query was repeated using a focused time range and again produced no matching events.

Therefore:

```text
5140 telemetry: Not observed
```

This should be documented as a telemetry/visibility result rather than interpreted as proof that the share access did not happen.

---

## 10. Security Event ID 5145

Security Event ID `5145` was also queried.

Result:

```text
No events were found that match the specified selection criteria.
```

Therefore:

```text
5145 telemetry: Not observed
```

No detailed share-access event was available to provide additional information about the controlled operation.

---

## 11. Security Event ID 4624

Event ID `4624` records were available.

The most recent records shown by the query were concentrated around:

```text
28 September 2026 07:21:13
to
28 September 2026 07:21:30
```

The controlled `ADMIN$` test occurred on:

```text
01 October 2026 08:05:26
```

The available 4624 records therefore predated the test by approximately three days.

They were not used as evidence of the controlled `ADMIN$` operation.

---

## 12. SMB State After the Test

The following commands were executed again:

```powershell
Get-SmbConnection
Get-SmbSession
Get-SmbOpenFile
```

No active SMB session or open SMB file was returned.

This means that no persistent/current SMB state was visible at the time of the post-test checks.

---

