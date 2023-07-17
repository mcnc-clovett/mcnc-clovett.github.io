---
layout: post
title:  "Connecting to Old SSH Servers"
date:   2023-07-15 00:00:00 -0400
categories: [Services, SSH / SFTP]
tags: [security, ssh, cli, command line, openssh]
---
`Unable to negotiate with 192.0.2.1 port 22: no matching key exchange method found. Their offer: diffie-hellman-group-exchange-sha1,diffie-hellman-group14-sha1,diffie-hellman-group1-sha1`

Have you come across an error like this when trying to SSH into a switch or server? It happens when the SSH server and your SSH client disagree on what algorithms they support. By default, newer versions of OpenSSH don't support older, less secure algorithms. This becomes a more prevalent problem as SSH clients update to newer versions and your switches and servers remain on older ones.

Don't worry, though. You can enable support for these algorithms in your SSH client configuration for good!

## Configuration Locations
First, lets locate where your SSH client configuration is. This is relatively the same between all operating systems.

- **Windows:** `C:\Users\<username>\.ssh\config`
- **macOS:** `/Users/<username>/.ssh/config`
- **Linux:** `/home/<username>/.ssh/config`

If this folder and/or file doesn't exist, you can create them.

### Windows (cmd.exe)
```console
C:\Users\cne> mkdir %HOMEPATH\.ssh
C:\Users\cne> cd %HOMEPATH%\.ssh
C:\Users\cne\.ssh> copy NUL config
C:\Users\cne\.ssh> notepad config
```

### Windows (Powershell)
```console
PS C:\Users\cne> mkdir ~\.ssh
PS C:\Users\cne> cd ~\.ssh
PS C:\Users\cne> New-Item -Path config -ItemType File
PS C:\Users\cne> notepad config
```

### macOS
```terminal?prompt=%
cne@mcnc ~ % mkdir -m 700 ~/.ssh
cne@mcnc ~ % cd ~/.ssh
cne@mcnc .ssh % touch config
cne@mcnc .ssh % open config
```

### Linux
```terminal
cne@mcnc:~$ mkdir -m 700 ~/.ssh
cne@mcnc:~$ cd ~/.ssh
cne@mcnc:~/.ssh$ touch config
cne@mcnc:~/.ssh$ nano config
```

## SSH Settings
There are several errors you may run across while connecting to older SSH servers. The most common settings related to these are `KexAlgorithms` (key exchange), `HostKeyAlgorithms`, `Ciphers` and `MACs` (message authentication codes).

### KexAlgorithms
Key echange algorithms are used to exchange the secret key that the SSH server and client will use to encrypt the traffic between them. The error you'll see is much like this:

`Unable to negotiate with 192.0.2.1 port 22: no matching key exchange method found. Their offer: diffie-hellman-group-exchange-sha1,diffie-hellman-group14-sha1,diffie-hellman-group1-sha1`

The key words here are `no matching key exchange method found`. The server supports the algorithms listed, which you must enable at least one of to communicate with it. You can do this by copying the offers and placing them in your `config` file with the configuration name `KexAlgorithms` and using the `+` to add them. Do *NOT* forget the `+`, or you'll disable the defaults.

```ssh
KexAlgorithms +diffie-hellman-group-exchange-sha1,diffie-hellman-group14-sha1,diffie-hellman-group1-sha1
```

You can list all the key exchange algorithms your client supports with `ssh -Q kex`:
```console
C:\Users\cne> ssh -Q kex
diffie-hellman-group1-sha1
diffie-hellman-group14-sha1
diffie-hellman-group14-sha256
diffie-hellman-group16-sha512
diffie-hellman-group18-sha512
diffie-hellman-group...
```
### HostKeyAlgorithms
Host key algorithms specify which host key types that the server supports. The typical error looks something like this:

`Unable to negotiate with 192.0.2.1 port 22: no matching host key type found. Their offer: ssh-rsa`

The key words here are `no matching host key type found`. The server supports the algorithms listed, which you must enable at least one of to communicate with it. You can do this by copying the offers and placing them in your `config` file with the configuration name `HostKeyAlgorithms` and using the `+` to add them. Do *NOT* forget the `+`, or you'll disable the defaults.

```ssh
HostKeyAlgorithms +ssh-rsa
```

You can list all the host key algorithms your client supports with `ssh -Q key`:
```console
C:\Users\cne> ssh -Q key
ssh-ed25519
ssh-ed25519-cert-v01@openssh.com
ssh-rsa
ssh-dss
ecdsa-sha2-nistp256
ecdsa-sha2-nistp384
ecdsa-sha2-nistp521
ssh-rsa-cert...
```

### Ciphers
Ciphers are used to actually do the encrypting of the traffic between client and server. An error you would see for ciphers is like this:

`Unable to negotiate with 192.0.2.1 port 22: no matching cipher found. Their offer: aes256-cbc,aes192-cbc,aes128-cbc`

The key words here are `no matching cipher found`. The server supports the ciphers listed, which you must enable at least one of to communicate with it. You can do this by copying the offers and placing them in your `config` file with the configuration name `Ciphers` and using the `+` to add them. Do NOT forget the `+`, or you'll disable the defaults.

```ssh
Ciphers +aes256-cbc,aes192-cbc,aes128-cbc
```

You can list all the ciphers that your client supports with `ssh -Q cipher`:
```console
C:\Users\cne> ssh -Q cipher
3des-cbc
aes128-cbc
aes192-cbc
aes256-cbc
aes128-ctr
aes192-ctr
aes256-ctr
aes128-gcm...
```

### MACs
MACs, or message authentication codes, are used to ensure that data hasn't been tampered with before they reach their intended recipient. A MAC error looks like this:

`Unable to negotiate with 192.0.2.1 port 22: no matching MAC found. Their offer: hmac-sha1`

The key words here are `no matching MAC found`. The server supports the MACs listed, which you must enable at least one of to communicate with it. You can do this by copying the offers and placing them in your `config` file with the configuration name `MACs` and using the `+` to add them. Do NOT forget the `+`, or you'll disable the defaults.

```ssh
MACs +hmac-sha1
```

You can list all the MACs that your client supports with `ssh -Q mac`:
```console
C:\Users\cne> ssh -Q mac
hmac-sha1
hmac-sha1-96
hmac-sha2-256
hmac-sha2-512
hmac-md5
hmac-md5-96
umac-64@openssh.com
umac-128@openssh.com
hmac-sha1-etm...
```

## Conclusion
Depending on how old the server is, you may not have to add each of the settings listed here, but they are the most common. The configuration I have to communicate with the server used in this example looks like this:

```ssh
KexAlgorithms +diffie-hellman-group-exchange-sha1
HostKeyAlgorithms +ssh-rsa
Ciphers +aes256-cbc
```

If you'd like to use these less secure configurations only on certain servers, you can use `Host` to list the hosts and/or networks like this:
```ssh
Host 192.0.2.1 192.168.* 172.16.0.*
    KexAlgorithms +diffie-hellman-group-exchange-sha1
    HostKeyAlgorithms +ssh-rsa
    Ciphers +aes256-cbc
```
