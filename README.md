# Hyper-V Backup module
This repository contains fork of the https://www.powershellgallery.com/packages/xHyper-VBackup module created by Taylor Brown and includes compatibility fixes that are not yet included in original module available in PowerShell gallery.

## Create full backup of VM
```powershell
$vmName = "<VM to backup>"
# Create and remember backup snapshot of the VM
$checkpoint = New-VmBackupCheckpoint -VmName $vmName -ConsistencyLevel CrashConsistent

# Exports that snapshot to dedicated folder
Export-VMBackupCheckpoint -VmName $vmName -DestinationPath "E:\Backups\Full" -BackupCheckpoint $checkpoint

# Removes backup snapshot and converts it as reference point for future incremental backups
Convert-VmBackupCheckpoint -BackupCheckpoint $checkpoint
```

## Incremental backup
```powershell
$vmName = "<VM to backup>"

# Create and remember backup snapshot of the VM
$checkpoint = New-VmBackupCheckpoint -VmName $vmName -ConsistencyLevel CrashConsistent

# Select which reference point will be used for differential backup
$referencePoint = Get-VmReferencePoints -VmName $vmName 

# Exports differential backup of the machine 
Export-VMBackupCheckpoint -VmName $vmName -DestinationPath "E:\Backups\Diff" -BackupCheckpoint $checkpoint -ReferencePoint $referencePoint

# Removes backup snapshot and converts it as reference point for future incremental backups
Convert-VmBackupCheckpoint -BackupCheckpoint $checkpoint
```
