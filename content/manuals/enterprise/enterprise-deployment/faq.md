---
title: Enterprise deployment FAQs
linkTitle: FAQs
description: Frequently asked questions for deploying Docker Desktop at scale
keywords: msi, deploy, docker desktop, faqs, pkg, mdm, jamf, intune, windows, mac, enterprise, admin
tags: [FAQ, admin]
weight: 70
aliases:
 - /desktop/install/msi/faq/
 - /desktop/setup/install/msi/faq/
 - /desktop/setup/install/enterprise-deployment/faq/
---

## MSI

Common questions about installing Docker Desktop using the MSI installer.

### What happens to user data if they have an older Docker Desktop installation (i.e. `.exe`)?

Users must [uninstall](/manuals/desktop/uninstall.md) older `.exe` installations before using the new MSI version. This deletes all Docker containers, images, volumes, and other Docker-related data local to the machine, and removes the files generated by Docker Desktop. 

To preserve existing data before uninstalling, users should [backup](/manuals/desktop/settings-and-maintenance/backup-and-restore.md) their containers and volumes.

For Docker Desktop 4.30 and later, the `.exe` installer includes a `-keep-data` flag that removes Docker Desktop while preserving underlying resources such as the container VMs:

```powershell
& 'C:\Program Files\Docker\Docker\Docker Desktop Installer.exe' uninstall -keep-data
```

### What happens if the user's machine has an older `.exe` installation?

The MSI installer detects older `.exe` installations and blocks the installation until the previous version is uninstalled. It prompts the user to uninstall their current/old version first, before retrying to install the MSI version.

### My installation failed, how do I find out what happened?

MSI installations may fail silently, offering little diagnostic feedback.

To debug a failed installation, run the install again with verbose logging enabled:

```powershell
msiexec /i "DockerDesktop.msi" /L*V ".\msi.log"
```

After the installation has failed, open the log file and search for occurrences of `value 3`. This is the exit code Windows Installer outputs when it has failed. Just above the line, you will find the reason for the failure.

### Why does the installer prompt for a reboot at the end of every fresh installation?

The installer prompts for a reboot because it assumes that changes have been made to the system that require a reboot to finish their configuration.

For example, if you select the WSL engine, the installer adds the required Windows features. After these features are installed, the system reboots to complete configurations so the WSL engine is functional.

You can suppress reboots by using the `/norestart` option when launching the installer from the command line:

```powershell
msiexec /i "DockerDesktop.msi" /L*V ".\msi.log" /norestart
```

### Why isn't the `docker-users` group populated when the MSI is installed with Intune or another MDM solution?

It's common for MDM solutions to install applications in the context of the system account. This means that the `docker-users` group isn't populated with the user's account, as the system account doesn't have access to the user's context.

As an example, you can reproduce this by running the installer with `psexec` in an elevated command prompt:

```powershell
psexec -i -s msiexec /i "DockerDesktop.msi"
```
The installation should complete successfully, but the `docker-users` group won't be populated.

As a workaround, you can create a script that runs in the context of the user account. 

The script would be responsible for creating the `docker-users` group and populating it with the correct user.

Here's an example script that creates the `docker-users` group and adds the current user to it (requirements may vary depending on environment):

```powershell
$Group = "docker-users"
$CurrentUser = [System.Security.Principal.WindowsIdentity]::GetCurrent().Name

# Create the group
New-LocalGroup -Name $Group

# Add the user to the group
Add-LocalGroupMember -Group $Group -Member $CurrentUser
```

> [!NOTE]
>
> After adding a new user to the `docker-users` group, the user must sign out and then sign back in for the changes to take effect.

## MDM

Common questions about deploying Docker Desktop using mobile device management
(MDM) tools such as Jamf, Intune, or Workspace ONE.

### Why doesn't my MDM tool apply all Docker Desktop configuration settings at once?

Some MDM tools, such as Workspace ONE, may not support applying multiple
configuration settings in a single XML file. In these cases, you may need to
deploy each setting in a separate XML file.

Refer to your MDM provider's documentation for specific deployment
requirements or limitations.