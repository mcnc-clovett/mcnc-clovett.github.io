---
layout: post
title:  "Windows-based SFTP Server"
date:   2021-09-08 00:00:00 -0400
categories: [Services, SSH / SFTP]
tags: [networking, ssh, windows, sftp, security]
---
This guide demonstrates the precedure for creating a Windows-based SFTP server using the built-in OpenSSH instance. The end result is an SFTP server only available to the “sftpusers” group via sftp only. There will be **no** ssh shell access to the server.

## System Requirements
* Windows 10 1809 or later, Windows Server 2019 or later
* A careful eye on security
* Recommendations, in case of compromise
    * Do **not** add the server to the domain
    * If the server is exposed to the internet, keep it in a **DMZ**
    * Do **not** use the server for other purposes

## Create SFTP Users
1. Add local user account for service (eg. ‘Powerschool’)
2. Add local group named ‘sftpusers’
    1. Add service user to ‘sftpusers’ group

## Create SFTP Chroot folder
1. Under C:\, create a folder named ‘sftp’
2. Edit the folder security settings and disable inheritance
3. Add ‘sftpusers’ group with Modify rights

## Install OpenSSH Server
1. Open PowerShell as administrator
2. Enter the following command
```powershell
Get-WindowsCapability -Online -Name OpenSSH.Server* | Add-WindowsCapability -Online
```

## Start OpenSSH Service
1. Open PowerShell as administrator
2. Enter the following commands
```powershell
Set-Service sshd -StartupType Automatic
Start-Service sshd
```

## Configure OpenSSH Service
1. Open PowerShell as administrator
2. Backup default config
```powershell
cd C:\ProgramData\ssh
cp .\sshd_config .\sshd_config.bak
```
3. Open configuration file with
```powershell
notepad.exe sshd_config
```
4. Replace entire configuration with
```
ForceCommand internal-sftp
Subsystem  sftp   internal-sftp -d "C:\sftp\"
ChrootDirectory C:\sftp
PermitTunnel no
AllowAgentForwarding no
AllowTcpForwarding no
X11Forwarding no
AllowGroups sftpusers
```
5. Save and close the file

## Restart OpenSSH Service
1. Open PowerShell as administrator
2. Enter the following command:
```powershell
Restart-Service sshd
```

## Test SSH Connection (should fail)
1. Verify you are not able to connect to the server from another machine via ssh with any user account
```shell
ssh <useraccount>@<newserverip>
```

## Test SFTP Connection (should work)
1. Use sftp to connect to the server (you can also use WinSCP of Filezilla)
```shell
sftp <useraccount>@<newserverip>
```
2. Once connected, verify you cannot change to a higher directory
    * `cd ..`, for example, should return “permission denied”

## Ensure a Safe Environment
1. Edit the **local firewall** to only allow certain hosts/networks to **port 22**
2. If allowing access from the **internet**, edit the **internet firewall** ACLs to only allow traffic in from trusted IP addresses on **port 22**
3. **Ensure** automatic updates are running and do not require intervention.
4. **Ensure** endpoint security is installed and running
