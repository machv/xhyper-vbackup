# Hyper-V Backup module
This repository contains a fork of the https://www.powershellgallery.com/packages/xHyper-VBackup module, originally created by Taylor Brown and includes compatibility fixes that are not yet included in original module which is available in the PowerShell gallery.

## Create a full backup of VM
```powershell
$vmName = "<VM name>" # Name of the VM to backup
$fullBackupPath = "D:\Backups\$vmName\Full" # Path to store full backup

# Create and remember backup snapshot of the VM
$checkpoint = New-VmBackupCheckpoint -VmName $vmName -ConsistencyLevel CrashConsistent

# Exports that snapshot to dedicated folder
Export-VMBackupCheckpoint -VmName $vmName -DestinationPath $fullBackupPath -BackupCheckpoint $checkpoint

# Removes backup snapshot and converts it as reference point for future incremental backups
Convert-VmBackupCheckpoint -BackupCheckpoint $checkpoint
```

## Incremental backup
```powershell
$vmName = "<VM name>" # Name of the VM to backup
$diffBackupPath = "D:\Backups\$vmName\Diff_$(Get-Date -UFormat "%Y-%m-%d_%H%M%S")" # Path to store diff backup

# Create and remember backup snapshot of the VM
$checkpoint = New-VmBackupCheckpoint -VmName $vmName -ConsistencyLevel CrashConsistent

# Select which reference point will be used for differential backup
$referencePoint = Get-VmReferencePoints -VmName $vmName 

# Exports differential backup of the machine 
Export-VMBackupCheckpoint -VmName $vmName -DestinationPath $diffBackupPath -BackupCheckpoint $checkpoint -ReferencePoint $referencePoint

# Removes backup snapshot and converts it as reference point for future incremental backups
Convert-VmBackupCheckpoint -BackupCheckpoint $checkpoint
```

## Restore full backup
```powershell
$vmName = "<VM name>" # Name of the VM to restore
$fullBackupPath = "D:\Backups\$vmName\Full" # Path with to the full backup
$restorePath = "D:\Restored-Full\$vmName"

# Find a virtual machine configuration file
$vmConfigFile = Get-ChildItem -Path $fullBackupPath -Recurse -Include "*.vmcx"

# Import a VM from backup
$imported = Import-VM -Path $vmConfigFile.FullName -GenerateNewId -Copy -VirtualMachinePath $restorePath -VhdDestinationPath "$restorePath\Virtual Hard Disks"

# (optional) Rename the VM
$imported | Rename-VM -NewName "$($imported.Name)-restored-full"
```

## Restore differential backup
```powershell
$vmName = "<VM name>" # Name of the VM to restore
$fullBackupPath = "D:\Backups\$vmName\Full" # Path with to the full backup
$diffBackupPath = "D:\Backups\$vmName\Diff_$(Get-Date -UFormat "%Y-%m-%d_%H%M%S")" # Path to store diff backup
$restorePath = "D:\Restored-Diff\$vmName"

# Copy virtual disks from the full backup in advance to make sure diff could merge properly during import
$disksPathSource = Join-Path $fullBackupPath "Virtual Hard Disks"
$diskPathDestination = Join-Path $restorePath "Virtual Hard Disks"
Copy-Item $disksPathSource $diskPathDestination -Recurse -Force -ErrorAction SilentlyContinue

# Find a virtual machine configuration file
$vmConfigFile = Get-ChildItem -Path $diffBackupPath -Recurse -Include "*.vmcx"

# Import a VM from backup
$imported = Import-VM -Path $vmConfigFile.FullName -GenerateNewId -Copy -VirtualMachinePath $restorePath -VhdDestinationPath "$restorePath\Virtual Hard Disks"

# (optional) Rename the VM
$imported | Rename-VM -NewName "$($imported.Name)-restored-diff"
```

## Notes
If you would encounter any issues with the import, you can use `Compare-VM` commandlet with the same parameters as a `Import-VM` commandlet and check the incompatibilities.
