---
layout: post
title:  "Creating an Enterprise Subordinate CA from an Offline Root CA"
date:   2022-07-04 00:00:00 -0400
categories: [Services, Certificate Authorities]
tags: [windows, certificate, security, active directory]
---
Every organization that has an Active Directory structure, or any other service that uses SSL/TLS services (ie. HTTPS, RDP, etc.), should have a certificate authority. This certificate authority is used to verify that you are indeed talking directly with the server you think you are and that the connection is secure. Having a certificate authority is also required to enable secure LDAP (LDAPS), though this can also be done with an [external certificate authority](https://docs.microsoft.com/en-us/troubleshoot/windows-server/identity/enable-ldap-over-ssl-3rd-certification-authority){:target="_blank"}.

This guide covers the second part of a **two-tiered** certificate authority (CA) setup, which is the Enterprise **Subordinate** or **Intermiediate** CA. The Enterprise CA is a Active Directory domain-joined server. This CA will be online and will issue certificates on behalf of the offline CA, which is protected off-network.

## Requirements
* Windows Server joined to your Active Directory domain
    * The server should be fully updated before starting
    * This server should have a static IP address
    * We will be using Windows Server 2022 for this example
* An offline Root CA
    * This can either be [Linux](/posts/offline-linux-ca/) or [Windows](/posts/offline-windows-ca/)-based and is covered elsewhere on this site
* (Recommended) [Private Enterprise Number](https://en.wikipedia.org/wiki/Private_Enterprise_Number) or [OID](https://en.wikipedia.org/wiki/Object_identifier) to represent your organization
    * One of these numbers can be obtained at no cost from IANA, here: [IANA PEN Application Form](https://pen.iana.org/pen/PenApplication.page)
    * You can check the registry to see if your organization has already obtained one, here: [IANA PEN Registry](https://www.iana.org/assignments/enterprise-numbers)

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
[PolicyStatementExtension]
Policies=InternalPolicy
[InternalPolicy]
OID=1.2.3.4.1455.67.89.5
[Certsrv_Server]
RenewalKeyLength=4096
RenewalValidityPeriod=Years
RenewalValidityPeriodUnits=10
CRLPeriod=weeks
CRLPeriodUnits=1
LoadDefaultTemplates=1
AlternateSignatureAlgorithm=1
[CRLDistributionPoint]
[AuthorityInformationAccess]
    ```
    {: file='C:\Windows\CAPolicy.inf'}

    * The OID listed above is an example by Microsoft. You should use one from IANA as mentioned in the [Requirements](#requirements) section above.
    * Notice that `LoadDefaultTemplates` is set to `0`, this will prevent unwanted or unneeded certificates from being issued after the install.

5. Save and close the CAPolicy.inf file

## Publish the Root CA Certificate and CRL
These two files were generated when you create either you [Linux](/posts/offline-linux-ca/) or [Windows](/posts/offline-windows-ca/) offline CA. You should have stored these files on a USB drive so that you can now import them into Active Directory.
* The Root CA Certificate should have a `.crt` file extension
* The Root CA Certificate revocation list (CRL) should have a `.crl` file extension

1. Open Powershell as **Administrator**
2. Assuming the CA cert and CRL files are on the root of `E:`, publish Root CA Certificate and CRL in Active Directory
```powershell
certutil -f -dspublish "E:\Root-CA_Example Root CA.crt" RootCA
certutil -f -dspublish "E:\Example Root CA.crl" RootCA
```
3. Add the Root CA Certificate and CRL in this server's local store
```powershell
certutil -addstore -f root "E:\Root-CA_Example Root CA.crt"
certutil -addstore -f root "E:\Example Root CA.crl"
```

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
10. On *Role Services*, check **Cerificate Authority** and **Certificate Authority Web Enrollment**, on the pop-up click **Add Features**, then click **Next**
11. On *Web Server Role (IIS)*, click **Next**
12. On *Role Services*, click **Next**
13. On *Confirmation*, click **Install**

## Configure the Certificate Authority Service
1. If the *Installation Wizard* is still open, click **Configure Active Directory Certificate Services on the destination server**. Otherwise, in *Server Manager* click the **flag icon** in the upper right which should have a warning symbol (⚠️), then **Post-deployment Configuration**
2. On *Credentials*, click **Next**
3. On *Role Services*, check **Certification Authority** and **Certificate Authority Web Enrollment**, then click **Next**
4. On *Setup Type*, ensure **Enterprise CA** is selected then click **Next**
5. On *CA Type*, select **Subordinate CA** and click **Next**
6. On *Private Key*, ensure **Create a new private key** is selected, then click **Next**
7. On *Cryptography*, set the *Key length* to **4096**
8. On *CA Name*, set the *Common name* to a descriptive value, such as "Contoso Subordinate CA". You may either accept the pre-filled *Distinguished name* or provide your own, such as `O=Example School,L=Raleigh,ST=North Carolina,C=US` then click **Next**
9. On *Certificate Request*, note the path in the *File name* field. This is where the certificate request file will be saved. Click **Next**
10. On *Certificate Database*, click **Next**
11. On *Confirmation*, click **Configure**
12. Once the installation is finished, close the wizard.

    > The warning message is expected. This is to inform you that you must manually use the `.req` file to request a CA certificate from your offline Root CA. You should now copy this file to a USB drive to load onto your offline Root CA.
    {: .prompt-warning}

## Request a CA Certificate from the Offline Root CA
Now that you have a certificate request, you must use your offline Root CA to obtain the Subordinate CA certificate.

### Linux-based Offline CA
1. Insert your USB drive containing the `.req` file into the offline Root CA server
2. Find the path to your device (all devices in Linux are represented by files)
```shell
sudo sfdisk -l
```
    - `-l` List all devices
3. My device was `/dev/sdb1`
```
    Disk /dev/sdb: 7.52 GiB, 8058306560 bytes, 15738880 sectors
    Disk model: USB Disk        
    Units: sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disklabel type: gpt
    Disk identifier: 730CBDED-473E-4991-9AF8-A6910A72F498

    Device     Start      End  Sectors  Size Type
    /dev/sdb1   2048 15736831 15734784  7.5G Microsoft basic data
```

4. Mount the USB Drive to the `/media` directory
```shell
sudo mount /dev/sdb1 /media
```

5. Generate the Sub-CA Certificate using OpenSSL
```shell
sudo openssl ca -config /etc/ssl/root-ca/openssl.cnf -extensions v3_intermediate_ca -in /media/CA01.ad.example.edu_ad-CAS-CA.req -out /media/CA01.ad.example.edu_ad-CAS-CA.crt -days 3650 -notext
```
* `ca` This is a Certificate Authority operation
* `-config` The location of the OpenSSL configuration file to use
* `-extentions` The certificate extensions to use, as defined in the config file
* `-in` The location of the `.req` file from the Subordinate CA
* `-out` Location and name of the output `.crt` certificate for the Sub-CA
* `-days` How long the certificate will be valid, 10 years in this case
* `-notext` Do not include a text version of the certificate in the output

6. You will be asked for the password of the Root CA's private key, which you should have documented
```
Using configuration from /etc/ssl/root-ca/openssl.cnf
Enter pass phrase for /etc/ssl/root-ca/private/ca.key:
```

7. You will be asked to verify the request and signature. If this looks correct, answer `y` to both questions. Below is my example:
```
Enter pass phrase for /etc/ssl/root-ca/private/ca.key:
Check that the request matches the signature
Signature ok
Certificate Details:
        Serial Number:
            60:1c:cb:4e:95:f9:4b:2f:42:30:e6:fe:0a:bb:6a:2f
        Validity
            Not Before: Jul  5 18:00:58 2022 GMT
            Not After : Jul  2 18:00:58 2032 GMT
        Subject:
            countryName               = US
            stateOrProvinceName       = North Carolina
            localityName              = RTP
            organizationName          = MCNC
            organizationalUnitName    = CNE
            commonName                = MCNC Subordinate CA
        X509v3 extensions:
            X509v3 Subject Key Identifier: 
                2A:69:D8:FB:5E:9E:2C:B4:B3:06:29:52:01:0B:5A:6E:E3:18:2C:55
            X509v3 Authority Key Identifier: 
                46:E7:C6:7A:74:1E:10:56:94:CC:39:22:FD:87:3F:AD:E0:EF:8E:D5
            X509v3 Basic Constraints: critical
                CA:TRUE, pathlen:0
            X509v3 Key Usage: critical
                Digital Signature, Certificate Sign, CRL Sign
Certificate is to be certified until Jul  2 18:00:58 2032 GMT (3650 days)
Sign the certificate? [y/n]:y
1 out of 1 certificate requests certified, commit? [y/n]y
Write out database with 1 new entries
Data Base Updated
```

6. Unmount the USB drive **before** removing it
```shell
sudo umount /media
```
7. Shutdown the offline Root CA

### Windows-based Offline CA
1. Insert your USB drive into the offline Windows Root CA server
2. On the offline Windows Root CA server, open the **Certificate Authority** MMC by going to **Tools > Certificate Authority** in Server Manager or by running `certsrv.msc`
3. In the **Certificate Authority** MMC, right-click your Root CA, then select **All Tasks > Submit New request...**
4. Browse to your `.req` file and click **Open**
5. Double click on **Pending Requests** you should see your new request listed
6. Right-click on the request and select **All Tasks > Issue**
    * You should see the new certificate under **Issued Certificates** and note the **Request ID** for the next step
7. Open Powershell as **Administrator** and export the new Sub-CA certificate to your USB drive
```powershell
certreq -retrieve 2 "E:\CA01.ad.example.edu_ad-CAS-CA.crt"
```
    * `-retrieve` This is the number of the **Request ID** from the **Issued Certificates** container
8. Click **OK** on the *Certificate Authority List* pop-up
9. Safely eject and remove the USB drive from the server
10. Shutdown the offline Root CA

## Install the new Sub-CA Certificate on the Enterprise Subordinate CA
1. Insert your USB drive into the offline Windows Root CA server
2. On the Windows Enterprise Subordinate CA server, open the **Certificate Authority** MMC by going to **Tools > Certificate Authority** in Server Manager or by running `certsrv.msc`
3. In the **Certificate Authority** MMC, right-click your Subordinate CA, then select **All Tasks > Install CA Certificate...**
4. Change the File type dropdown to **X.509 Certificate (\*.cer,\*.crt)**
5. Browse to your USB drive or location of the `.crt` file from the Root CA and select it
6. Click **Open**
7. Right-click right-click your Subordinate CA and select **All Tasks > Start Service**
    * Certificate Services should have started and a green check should be on you Subordinate CA

## Perform Post Installation Configuration Tasks on the Subordinate Issuing CA
1. Open Powershell as **Administrator**
2. Configure the CRL and Delta CRL settings
```powershell
certutil -setreg CA\CRLPeriodUnits 1
certutil -setreg CA\CRLPeriod "Weeks"
certutil -setreg CA\CRLDeltaPeriodUnits 1
certutil -setreg CA\CRLDeltaPeriod "Days"
```
3. Define CRL overlap settings
```powershell
certutil -setreg CA\CRLOverlapPeriodUnits 12
certutil -setreg CA\CRLOverlapPeriod "Hours"
```
4. Set the Validity Period to half the life of the Subordinate CA certificate. In this case, 5 years. *The default is 2 years*
```powershell
certutil -setreg CA\ValidityPeriodUnits 5
certutil -setreg CA\ValidityPeriod "Years"
```
5. Retart the Certificate Authority service
```powershell
Restart-Service certsvc
```
6. Generate the Certificate Revocation List 
```powershell
certutil -crl
```

## (Recommended) Publish the Domain Controller Certificate Template
1. In the **Certificate Authority** MMC of the Subordinate CA, right click on **Certificate Templates** and select **New > Certificate Template to Issue**
2. On the *Enable Certificate Templates* pop-up, select **Domain Controller** and click **OK**
* After some time (gpupdate), the Domain Controllers will enroll themselves with a certificate which will enable LDAPS (port 636)