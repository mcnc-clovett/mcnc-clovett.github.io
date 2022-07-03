---
layout: post
title:  "Linux-based SFTP Server"
date:   2022-07-01 00:00:00 -0400
categories: [Services, SSH / SFTP]
tags: [networking, ssh, linux, sftp, security]
---
This guide demonstrates the precedure for creating a Linux-based SFTP server using OpenSSH. The end result is an SFTP server only available to the “sftpusers” group via sftp only. SSH shell access will only be available to users outside of that group. Also, internet-based devices will require an SSH key to login to a shell.

## System Requirements
* Any Linux distribution will suffice,
but we'll be using Ubuntu 22.04 LTS Server
* A careful eye on security
* Recommendations, in case of compromise
    * Do **not** tie the server to the Active Directory for authentication
    * If the server is exposed to the internet, keep it in a **DMZ**
    * Do **not** use the server for other purposes

## Ensure OpenSSH Server is Installed
1. Enter the following commands
```shell
sudo apt update && sudo apt dist-upgrade -y
sudo apt install openssh-server -y
```

## Create SFTP Users
1. Add local group named ‘sftpusers’
```shell
sudo addgroup sftpusers
```
2. Add local user account for service (eg. ‘Powerschool’) and add user to ‘sftpusers’ group
```shell
sudo adduser --home /powerschool --no-create-home --ingroup sftpusers --shell /usr/sbin/nologin powerschool
```
- `--home` Location of the user's home directory. This is relative to the path of the root directory for SFTP users that we will set later.
- `--no-create-home` Do not create the home directory. This is because it will actually exist elsewhere. (You'll see)
- `--ingroup` This is the group the user will be added to.
- `--shell` The shell to be used by the user. The SFTP users will not have access to a shell, therefore they are given `nologin`.
- `powershell` This is the name of the user you wish to create.

## Create SFTP Directory Structure
1. Create the "chroot" folder for the SFTP users
```shell
sudo mkdir -m 751 /sftp
```
2. Create the folder to be used by the user we created and set the ownership
```shell
sudo mkdir -m 700 /sftp/powerschool
sudo chown powerschool:sftpusers /sftp/powerschool
```

## Configure OpenSSH Service
1. Make a backup of the SSH config file
```shell
cd /etc/ssh
sudo mv sshd_config sshd_config.bak
```
2. Open a new config file using `nano` or other text editor
```shell
sudo nano sshd_config
```
3. Paste the following text into the new file

```
HostKey /etc/ssh/ssh_host_rsa_key
HostKey /etc/ssh/ssh_host_ed25519_key

# Set this to "yes" to allow logins to shell from the internet without an SSH key
PasswordAuthentication no
PubkeyAuthentication yes
PermitRootLogin prohibit-password

AllowAgentForwarding no
AllowTcpForwarding no
GatewayPorts no
X11Forwarding no
PermitTunnel no
DisableForwarding yes

Subsystem       sftp    internal-sftp

Match Group sftpusers
        # Set this to "no" to require SSH key for SFTP users
        PasswordAuthentication yes
        ForceCommand internal-sftp
        ChrootDirectory /sftp
        DisableForwarding yes
        PermitTunnel no

Match Address 10.0.0.0/8,172.16.0.0/12,192.168.0.0/16
        # Set this to "no" to require SSH key to login from local nets
        PasswordAuthentication yes
```

- Close the config file with `Ctrl-X`, then press `Y` to confirm the changes and press `Enter` to close it.

## Restart OpenSSH Service
1. With the new config file in place, restart the `sshd` service
```shell
sudo systemctl restart sshd
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
    * `cd ..`, for example, should return “Permission denied”

## Ensure a Safe Environment
1. Edit the **local firewall** to only allow certain hosts/networks to **port 22**
2. If allowing access from the **internet**, edit the **internet firewall** ACLs to only allow traffic in from trusted IP addresses on **port 22**
