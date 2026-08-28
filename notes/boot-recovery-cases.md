# Windows Boot Recovery Case Studies

This document summarizes four controlled Windows 11 boot-recovery incidents completed in a VMware laboratory using Windows PE and command-line recovery tools.

The recovery constraint was the same in every scenario:

- do not format the disk;
- do not reinstall Windows;
- do not create a replacement virtual machine;
- preserve the existing user data;
- diagnose and recover each failure independently;
- validate the result after remediation.

The emphasis is on evidence-driven troubleshooting: identify the failure, confirm the relevant system state, apply the smallest justified repair, and verify that Windows can boot again without data loss.

---

## Case 1 — Corrupted Boot Configuration Data

### Symptom

Windows failed to start normally and entered the recovery environment instead of loading the operating system.

The scenario was associated with corrupted or missing **Boot Configuration Data (BCD)**.

### Evidence and Diagnosis

Windows PE was used to inspect the disk because drive letters in the recovery environment may differ from those used during a normal boot.

The main Windows and EFI partitions were identified with DiskPart and temporarily assigned explicit drive letters.

```cmd
diskpart
list volume
select volume 1
assign letter=W
select volume 2
assign letter=S
exit
```

The Windows installation and EFI partition were then checked before any repair was attempted.

```cmd
dir W:\Windows
dir W:\Users
dir S:\EFI
```

These checks established that:

- the Windows installation was present on `W:`;
- the user profile data was still present;
- `S:` was the EFI System Partition.

### Remediation

The boot files were rebuilt with:

```cmd
bcdboot W:\Windows /s S: /f UEFI
```

The command returned:

```text
Boot files successfully created.
```

### Validation

The VM was restarted without booting from Windows PE.

Windows started successfully and the existing local user profile remained available.

### What This Proves

The recovery confirms that rebuilding the UEFI boot files restored the startup path for this scenario without reinstalling Windows.

### What This Does Not Prove

It does not prove that every Windows startup failure is caused by BCD corruption. Similar symptoms can result from missing system files, storage-driver problems, partition damage, firmware configuration, or other boot-chain failures.

---

## Case 2 — Missing `winload.efi`

### Symptom

Windows could not complete startup and Automatic Repair was unable to resolve the failure.

The scenario was associated with a missing `winload.efi`, an essential Windows boot loader component in a UEFI system.

### Evidence and Diagnosis

After identifying the offline Windows partition, the file was checked directly:

```cmd
dir W:\Windows\System32\winload.efi
```

The result was:

```text
File Not Found
```

This provided direct evidence that the expected boot file was absent.

### Remediation

System File Checker was executed against the offline Windows installation:

```cmd
sfc /scannow /offbootdir=W:\ /offwindir=W:\Windows
```

SFC reported that corrupted files had been found and successfully repaired.

### Validation

The file was checked again:

```cmd
dir W:\Windows\System32\winload.efi
```

The second check showed that `winload.efi` had been restored.

The VM was then rebooted from disk and Windows started successfully. The existing user data remained accessible.

### What This Proves

The before-and-after file check demonstrates that the missing boot component was restored and that the repaired system was able to boot.

### What This Does Not Prove

A successful SFC result alone does not prove that all boot problems are resolved. The final reboot is required because file integrity and successful operating-system startup are separate validation points.

---

## Case 3 — `INACCESSIBLE_BOOT_DEVICE` Caused by Disabled NVMe Driver

### Symptom

Windows displayed:

```text
INACCESSIBLE_BOOT_DEVICE (0x7B)
```

The system could not access the boot device during startup.

### Evidence and Diagnosis

Because the VM used an NVMe disk, the offline Windows registry was inspected to determine whether the `stornvme` storage driver was enabled.

The offline SYSTEM hive was loaded:

```cmd
reg load HKLM\OFFLINE W:\Windows\System32\Config\SYSTEM
```

The active ControlSet was identified:

```cmd
reg query HKLM\OFFLINE\Select
```

The `Current` value indicated that `ControlSet001` was active.

The storage-driver startup value was then queried:

```cmd
reg query HKLM\OFFLINE\ControlSet001\Services\stornvme /v Start
```

The result showed:

```text
Start    REG_DWORD    0x4
```

In this scenario, `0x4` meant the driver was disabled.

### Remediation

The driver startup value was changed to `0x0`:

```cmd
reg add HKLM\OFFLINE\ControlSet001\Services\stornvme /v Start /t REG_DWORD /d 0 /f
```

The command completed successfully.

The value was queried again:

```cmd
reg query HKLM\OFFLINE\ControlSet001\Services\stornvme /v Start
```

The new result showed:

```text
Start    REG_DWORD    0x0
```

Finally, the offline registry hive was cleanly unloaded:

```cmd
reg unload HKLM\OFFLINE
```

### Validation

After rebooting from disk, Windows started normally.

An existing user file was opened to confirm that the system recovery had not removed the user data.

### What This Proves

The troubleshooting sequence establishes a direct chain:

`INACCESSIBLE_BOOT_DEVICE`  
→ NVMe boot driver inspected  
→ `stornvme` found disabled  
→ startup value corrected  
→ corrected value verified  
→ Windows boot restored.

This is stronger than applying a generic repair command because the remediation was tied to a confirmed configuration fault.

### What This Does Not Prove

`INACCESSIBLE_BOOT_DEVICE` is not specific to `stornvme`. The same stop code can have other causes, including storage-controller changes, incompatible drivers, disk problems, or boot-storage configuration changes. The registry evidence was what made this remediation appropriate in this particular incident.

---

## Case 4 — Missing `ntoskrnl.exe`

### Symptom

Windows displayed a recovery error indicating that the operating system could not be loaded because the kernel file was missing or contained errors.

The affected file was:

```text
\WINDOWS\system32\ntoskrnl.exe
```

The reported error code was:

```text
0xc000000f
```

### Evidence and Diagnosis

The suspected file was checked directly from Windows PE:

```cmd
dir W:\Windows\System32\ntoskrnl.exe
```

The result was:

```text
File Not Found
```

Other important startup files were also checked:

```cmd
dir W:\Windows\System32\hal.dll
dir W:\Windows\System32\winload.efi
```

Both were present, narrowing the observed missing-file condition to `ntoskrnl.exe`.

### Remediation

Offline System File Checker was used:

```cmd
sfc /scannow /offbootdir=W:\ /offwindir=W:\Windows
```

SFC reported that corrupted files had been found and successfully repaired.

### Validation

The kernel file was checked again:

```cmd
dir W:\Windows\System32\ntoskrnl.exe
```

This time the file was present.

The VM was rebooted from disk, Windows started successfully, and an existing user file remained accessible.

### What This Proves

The sequence demonstrates a specific missing-system-file diagnosis followed by offline repair and direct verification that the file was restored before reboot testing.

### What This Does Not Prove

The presence of `ntoskrnl.exe` alone does not guarantee a healthy Windows installation. Successful boot and access to existing data were therefore treated as separate validation steps.

---

## Recovery Workflow

Across the four incidents, the same troubleshooting discipline was applied:

1. Observe the startup symptom.
2. Boot into Windows PE.
3. Identify the offline Windows and EFI partitions.
4. Confirm that the existing Windows installation and user data are present.
5. Test the component directly related to the observed failure.
6. Apply a targeted remediation.
7. Re-run the relevant test after the change.
8. Reboot from the repaired disk.
9. Confirm successful Windows startup.
10. Confirm that existing user data remains accessible.

## Tools Used

- Windows PE
- Command Prompt
- DiskPart
- `bcdboot`
- System File Checker (`sfc`)
- Registry command-line tools (`reg`)
- VMware Workstation
- Windows 11

## Key Engineering Principle

A repair command was not treated as evidence of recovery by itself.

The lab separated:

**symptom → evidence → diagnosis → remediation → technical validation → functional validation**

This approach reduces unnecessary destructive actions and is particularly relevant to endpoint support scenarios where preserving the user's existing data is a primary requirement.
