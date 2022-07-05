---
layout: post
title:  "Offline Root Certificate Authority - Windows Server"
date:   2022-07-03 00:00:00 -0400
categories: [Services, Certificate Authorities]
tags: [windows, certificate, security]
---
Every organization that has an Active Directory structure, or any other service that uses SSL/TLS services (ie. HTTPS, RDP, etc.), should have a certificate authority. This certificate authority is used to verify that you are indeed talking directly with the server you think you are and that the connection is secure. Having a certificate authority is also required to enable secure LDAP (LDAPS), though this can also be done with an [external certificate authority](https://docs.microsoft.com/en-us/troubleshoot/windows-server/identity/enable-ldap-over-ssl-3rd-certification-authority){:target="_blank"}.

Choosing to host your offline root CA on a Windows-based system, rather than Linux, will feel more comfortable to those who are not familiar to Linux. However, since this server will be shutdown 99% of the time and should never be on the network once configured, it may seem like a waste of a server license.

## Why have an **offline** Root CA?
> If a root CA is in some way compromised (broken into, hacked, stolen, or accessed by an unauthorized or malicious person), then all of the certificates that were issued by that CA are also compromised. Since certificates are used for data protection, identification, and authorization, the compromise of a CA could compromise the security of an entire organizational network. For that reason, many organizations that run internal PKIs install their root CA offline. That is, the CA is never connected to the company network, which makes the root CA an offline root CA. Make sure that you keep all CAs in secure areas with limited access.[^fn-rootCA] <br><br>
*-- Microsoft TechNet*

## Requirements
* Latest Windows Server available installed on a single-purpose physical server
    * You will need to be able to transfer files to and from this device via USB drive.
    * We'll be using Windows Server 2022 for this example.
    * This system will be used as an offline root CA, therefore it will be powered off after we're done.
* USB drive formatted with ExFAT or FAT32 file system

## Ensure the Server is Updated
1. Update the server with Windows Update

---

> At this point, there is no need for the server to be connected to the network. You should remove the network cable from the ethernet port. If this is a device with a wireless card, ensure it is not connected to the wireless or even remove the WLAN card.
{: .prompt-warning }
---

## Create CAPolicy.inf File
1. Open Powershell as **Administrator**
2. Create a new file with `notepad.exe` called `CAPolicy.inf` in **C:\Windows**
```powershell
cd C:\Windows
notepad.exe CAPolicy.inf
```
3. Answer `Yes` to create the new file
4. Copy this text into the file
    ```ini
[Version]
Signature="$Windows NT$"
[Certsrv_Server]
RenewalKeyLength=4096
RenewalValidityPeriod=Years
RenewalValidityPeriodUnits=20
AlternateSignatureAlgorithm=1
    ```
    {: file='C:\Windows\CAPolicy.inf'}

5. Save and close the CAPolicy.inf file

## Install the Active Directory Certificate Services Role
1. Open **Server Manager**, if not already open
2. Click **Manage** in the top right corner
3. Click **Add Roles and Features**
4. On *Before You Begin*, click **Next**
5. On *Installation Type*, ensure **Role-based or feature-based installation** is selected and click **Next**
6. On *Server Selection*, ensure this server is selected and click **Next**
7. On *Server Roles*, check **Active Directory Certificate Services**, on the pop-up click **Add Features**, then click **Next**
8. On *Features*, click **Next**
9. On *AD CS*, click **Next**
10. On *Role Services*, ensure **Cerificate Authority** is checked and click **Next**
11. On *Confirmation*, click **Install**

## Configure the Certificate Authority Service
1. If the *Installation Wizard* is still open, click **Configure Active Directory Certificate Services on the destination server**. Otherwise, in *Server Manager* click the **flag icon** in the upper right which should have a warning symbol (⚠️), then **Post-deployment Configuration**
2. On *Credentials*, click **Next**
3. On *Role Services*, check **Certification Authority**, then click **Next**
4. On *Setup Type*, click **Next** (Standalone CA is the only available option)
5. On *CA Type*, select **Root CA** and click **Next**
6. On *Private Key*, ensure **Create a new private key** is selected, then click **Next**
7. On *Cryptography*, set the *Key length* to **4096**
8. On *CA Name*, set the *Common name* to a descriptive value, such as "Contoso Root CA", then click **Next**
9. On *Validity Period*, set the *validity period* to **20 Years**.
10. On *Certificate Database*, click **Next**
11. On *Confirmation*, click **Configure**

## Perform Post Installation Configuration
1. Open Powershell ad **Administrator**
2. Define the *Active Directory Configuration Partition Distinguished Name* (Substitute your DC values)
```powershell
certutil -setreg CA\DSConfigDN "CN=Configuration,DC=ad,DC=example,DC=edu"
```
3. Define *CRL Period Units* and *CRL Period*
```powershell
certutil -setreg CA\CRLPeriodUnits 52
certutil -setreg CA\CRLPeriod "Weeks"
certutil -setreg CA\CRLDeltaPeriodUnits 0
```
4. Define *CRL Overlap Period Units* and *CRL Overlap Period*
```powershell
certutil -setreg CA\CRLOverlapPeriodUnits 12
certutil -setreg CA\CRLOverlapPeriod "Hours"
```
5. Define *Validity Period Units* for all issued certificates by this CA
```powershell
certutil -setreg CA\ValidityPeriodUnits 10
certutil -setreg CA\ValidityPeriod "Years"
```

## Restart the Certificate Authority Service and Publich the CRL
1. Restart the CA service
```powershell
Restart-Service certsvc
```
2. Publish the Certificate Revocation List (CRL)
```powershell
certutil -crl
```

## Copy the Root CA Certificate and CRL to USB Drive
You should now copy `*.crl` and `*.crt` to a USB drive. The two files can be found at `C:\Windows\system32\CertSrv\CertEnroll`{: .filepath}.

> You will need the **root CA certificate** and **CRL file** when you configure the Windows Enterprise Subordinate CA
{: .prompt-tip}

## Shutdown and Store the Machine
You may now shutdown the server and store it. The only times the server will be needed are when you stand up a new subordinate (intermediate) certificate authority, or when a sub-CA needs its certificate renewed. Sub-CA's should renew their certificates every ten years or so.

## Footnotes
[^fn-rootCA]: <https://social.technet.microsoft.com/wiki/contents/articles/2900.offline-root-certification-authority-ca.aspx>