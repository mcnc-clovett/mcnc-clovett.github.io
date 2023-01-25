---
layout: page
title:  "Free Tools"
icon: fas fa-tools
order: 3
---
We all love FREE. 

# Tool List (in not particular order)
1. [MTR](#mtr)
2. [WinMTR](#winmtr-redux)
3. [Tftpd64](#tftpd64)
4. [Wireshark](#wireshark)
5. [Nmap](#nmap)
6. [WinMerge](#winmerge)
7. [WinSCP](#winscp)
8. [Chocolatey](#chocolatey)
9. [Homebrew](#homebrew)
10. [IIS Crypto]()
11. [7-zip](#7-zip)
12. [Notepad++](#notepad)
13. [OpenSSH](#openssh)

## MTR
mtr combines the functionality of the 'traceroute' and 'ping' programs in a single network diagnostic tool.

As mtr starts, it investigates the network connection between the host mtr runs on and a user-specified destination host. After it determines the address of each network hop between the machines, it sends a sequence ICMP ECHO requests to each one to determine the quality of the link to each machine. As it does this, it prints running statistics about each machine.
* Supported Operating Systems
    * Linux
    * macOS
* Homepage
    * <https://www.bitwizard.nl/mtr/>
* Installation
    * Linux
        * `sudo apt install mtr-tiny`
    * macOS ([Homebrew](#homebrew))
        * `brew install mtr`

## WinMTR-Redux
WinMTR is a free MS Windows visual application that combines the functionality of the traceroute and ping in a single network diagnostic tool. This is a graphical Windows version of the popular diagnostic [MTR](#mtr) tool.
* Supported Operating System
    * Windows
* Homepage
    * <https://github.com/White-Tiger/WinMTR>
* Download Page
    * <https://github.com/White-Tiger/WinMTR/releases>
* Install Commands
    * Windows ([Chocolatey](#chocolatey))
        * `choco install winmerge-redux`

## Tftpd64
Tftpd64 is a free, lightweight, opensource IPv6 ready application which includes DHCP, TFTP, DNS, SNTP and Syslog servers as well as a TFTP client.
The TFTP client and server are fully compatible with TFTP option support (tsize, blocksize and timeout), which allow the maximum performance when transferring the data. Some extended features such as directory facility, security tuning, interface filtering; progress bars and early acknowledgments enhance usefulness and throughput of the TFTP protocol for both client and server. The included DHCP server provides unlimited automatic or static IP address assignment.
* Supported Operating System
    * Windows
* Homepage 
    * <https://pjo2.github.io/tftpd64/>
* Download Page
    * <https://bitbucket.org/phjounin/tftpd64/downloads/>
* Install Commands
    * Windows ([Chocolatey](#chocolatey))
        * `choco install tftpd32`

## Wireshark
Wireshark is the world’s foremost and widely-used network protocol analyzer. It lets you see what’s happening on your network at a microscopic level and is the de facto (and often de jure) standard across many commercial and non-profit enterprises, government agencies, and educational institutions. Wireshark development thrives thanks to the volunteer contributions of networking experts around the globe and is the continuation of a project started by Gerald Combs in 1998.
* Supported Operating Systems
    * Windows
    * macOS
    * Linux
* Homepage
    * <https://www.wireshark.org/>
* Download Page
    * <https://www.wireshark.org/#download>
* Install Commands
    * Windows ([Chocolatey](#chocolatey))
        * `choco install wireshark`
    * macOS ([Homebrew](#homebrew))
        * `brew install wireshark`
    * Linux
        * `apt install wireshark`

## Nmap
Nmap ("Network Mapper") is a free and open source utility for network discovery and security auditing. Many systems and network administrators also find it useful for tasks such as network inventory, managing service upgrade schedules, and monitoring host or service uptime.
* Supported Operating Systems
    * Windows
    * macOS
    * Linux
* Homepage
    * <https://nmap.org/>
* Download Page
    * <https://nmap.org/download>
* Install Commands
    * Windows ([Chocolatey](#chocolatey))
        * `choco install nmap`
    * macOS ([Homebrew](#homebrew))
        * `brew install nmap`
    * Linux
        * `apt install nmap`

## WinMerge
WinMerge is an Open Source differencing and merging tool for Windows. WinMerge can compare both folders and files, presenting differences in a visual text format that is easy to understand and handle.
* Supported Operating System
    * Windows
* Homepage
    * <https://winmerge.org/>
* Download Page
    * <https://winmerge.org/downloads/>
* Install Commands
    * Windows ([Chocolatey](#chocolatey))
        * `choco install winmerge`

## WinSCP
WinSCP is a popular SFTP client and FTP client for Microsoft Windows! Copy files between a local computer and remote servers using FTP, FTPS, SCP, SFTP, WebDAV or S3 file transfer protocols.
* Supported Operating System
    * Windows
* Homepage
    * <https://winscp.net/>
* Download Page
    *  <https://winscp.net/eng/download.php>
* Install Commands
    * Windows ([Chocolatey](#chocolatey))
        * `choco install winscp`

## Chocolatey
Chocolatey has the largest online registry of Windows packages. Chocolatey packages encapsulate everything required to manage a particular piece of software into one deployment artifact by wrapping installers, executables, zips, and/or scripts into a compiled package file.

Package submissions go through a rigorous moderation review process, including automatic virus scanning. The community repository has a strict policy on malicious and pirated software.
* Supported Operating System
    * Windows
* Homepage
    * <https://chocolatey.org/>
* Download Page
    * <https://chocolatey.org/install>
* Install Commands
    * Windows (Powershell)
        * ```powershell
        Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
        ```

## Homebrew
Homebrew installs the stuff you need that Apple (or your Linux system) didn’t.
* Supported Operating Systems
    * macOS
    * Linux
* Homepage
    * <https://brew.sh/>
* Install Commands
    * macOS & Linux
        * ```terminal
        /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
        ```

## IIS Crypto
IIS Crypto is a free tool that gives administrators the ability to enable or disable protocols, ciphers, hashes and key exchange algorithms on Windows Server 2008, 2012, 2016, 2019 and 2022. It also lets you reorder SSL/TLS cipher suites offered by IIS, change advanced settings, implement Best Practices with a single click, create custom templates and test your website.
* Supported Operating System
    * Windows
* Homepage
    * <https://www.nartac.com/Products/IISCrypto/>
* Download Page
    * https://www.nartac.com/Products/IISCrypto/Download
* Install Commands
    * Windows ([Chocolatey](#chocolatey))
        * `choco install iiscrypto`

## 7-zip
7-Zip is a file archiver with a high compression ratio.
* Supported Operating Systems
    * Windows
    * macOS
    * Linux
* Homepage
    * <https://www.7-zip.org/>
* Download Page
    * <https://www.7-zip.org/download.html>
* Install Commands
    * Windows ([Chocolatey](#chocolatey))
        * `choco install 7zip`
    * macOS ([Homebrew](#homebrew))
        * `brew install sevenzip`
    * Linux
        * `apt install 7zip`

## Notepad++
Notepad++ is a free (as in “free speech” and also as in “free beer”) source code editor and Notepad replacement that supports several languages.
* Supported Operating Systems
    * Windows
* Homepage
    * <https://notepad-plus-plus.org/>
* Download Page
    * <https://notepad-plus-plus.org/downloads/>
* Install Commands
    * Windows ([Chocolatey](#chocolatey))
        * `choco install notepadplusplus`

## OpenSSH
OpenSSH is the premier connectivity tool for remote login with the SSH protocol. It encrypts all traffic to eliminate eavesdropping, connection hijacking, and other attacks. In addition, OpenSSH provides a large suite of secure tunneling capabilities, several authentication methods, and sophisticated configuration options.
* Supported Operating Systems
    * Windows (built-in since Win10 1809)
    * macOS (built-in)
    * Linux (built-in)
* Homepage
    * <https://www.openssh.com/>
* Download Page
    * Its built into your OS...
* Install Commands
    * Seriously, its built into your OS. Stop using PuTTY.