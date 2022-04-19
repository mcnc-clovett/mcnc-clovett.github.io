---
layout: post
title:  "PowerShell Script Signing"
date:   2021-03-07 00:00:00 -0400
categories: Powershell General
tags: powershell security certificates codesigning
---
Windows PowerShell has what’s called an **Execution Policy**. This policy defines how scripts are handled, and by default it is set to **Restricted**, which means no script is allowed to run. This is to prevent anyone from accidentally running a rogue script on the computer. What we typically see are Execution Policies set to **Bypass**, which allows any script to run. Anyone with administrative rights can set this, which makes Execution Policy more of a *safety* feature than a *security* one.

There are two policies that require scripts to be signed, RemoteSigned and AllSigned. RemoteSigned requires that any script downloaded from the internet be signed. This way you can be sure that the script wasn’t altered in transit. AllSigned means that every script that runs must be signed and valid, which includes local scripts that you create yourself. If you run a signed script under either of these policies that isn’t signed by a Trusted Publisher, you’ll be prompted with a warning. Also, if the script was altered since it was last signed by you or anyone else, you’ll receive an error. This is a way that you can trust the scripts you’ve written haven’t been altered with some malicious code trying to trick you into infecting your entire network.

## Configure the Certificate Templates {#templates}
1. Open the **Certificate Authority Console** for your CA, right click **Certificate Templates** and click **Manage**
2. Right click the **Code Signing** template then **Duplicate Certificate**
3. Under **General** tab:
    * Name the new template (ie. *“PS Code Signing”*)
    * Enter a Validity Period (ie. 2 years)
4. Under **Compatibility** tab:
    * Select the OS version of your CA server
    * Select the oldest version of Windows in the district
5. Under **Request Handling** tab:
    * Check **Allow private key to be exported**
    * Select **Prompt the user during enrollment and require user input when the private key is used**
6. Under **Cryptography** tab:
    * For **Provider Category**, select **Key Storage Provider**
    * For **Algorithm name** ensure **ECDSA_P256** is selected and **Minimum key size** is **256**
    * Select **Requests must use one of the following providers** and check **Microsoft Software Ley Storage Provider**
    * Set **Request hash** to **SHA256**
7. Under **Security** tab:
    * Add the group or users that you wish to be able to sign scripts for the district, and that they have the **Read** and **Enroll** permissions
8. Click **OK** and close the **Certificate Templates Console**
9. Open certsrv.msc, right click Certificate Templates > **New > Certificate Template to Issue**
10. Select the code signing certificate you created (ie. *"PS Code Signing"*) then click **OK**

## Enroll for a Signing Certificate {#enrollment}
1. Open certmgr.msc and under **Personal\Certificates**:
2. Right click in the white space > **All Tasks > Request New Certificate…**
3. Click **Next**, **Next**
4. Select the code signing certificate you created (ie. *“PS Code Signing”*) and click **Enroll**
5. Export the private key:
    1. Right click the new certificate > **All Tasks > Export…**
    2. Click **Next**, **Next**, **Next**
    3. Enter a save location and filename (ie. *publicSigningKey.cer*) and click **Next**
    4. Click **Finish**
6. Export the private key:
    1. Right click the new certificate > **All Tasks > Export…**
    2. Click **Next**
    3. Select **Yes**, export the private key, click **Next**
        * ****Optional**** If you’re going to use your code signing key on a Yubikey or similar device, check **Delete the private key if the export is successful**
    4. Click **Next**
    5. Check the box for Password protection and enter a secure password, set **Encryption** to **AES256-SHA256** and click **Next**
    6. Enter a save location and filename (ie. *privateSigningKey.pfx*) and click **Next**
    7. Click **Finish**

## Configure the Group Policy Objects {#gpos}
1. Under **Computer Configuration>Administrative Templates>Windows Components>Windows PowerShell**:
    1. Double-click **Turn on Script Execution**
        1. Check **Enabled**
        2. From the dropdown, choose **Allow only signed scripts**
2. Under **Computer Configuration>Policies>Windows Settings>Security Settings>Public Key Policies>Trusted Publishers**:
    1. Right click in the white space > **Import…**
    2. Click **Next**
    3. Browse for the public certificate that you exported previously (ie. *publicSigningKey.cer*)
    4. Click **Next**, enter the password used when the certificate was exported, click **Next**, **Finish**
3. Open PowerShell and run `gpupdate /force`

## Sign a Script and Test {#signing}
1. Open PowerShell as the user who was issued the certificate
2. Create a test script with 
```powershell
echo “Write-Host ‘Hello World!’” > test.ps1
```
3. Store the certificate in a variable with 
```powershell
$cert = Get-ChildItem Cert:\CurrentUser\My -CodeSigningCert
```
4. Sign the script 
```powershell
Set-AuthenticodeSignature -Certificate $cert -TimestampServer http://timestamp.digicert.com .\test.ps1
```
    * The time stamp server is important because without a verified time of signing, the script will no longer be vaid if your certificate expires. Using a time stamp certification in the signature will verify that the certificate was valid at the time it was signed.
5. Verify the script was signed 
```powershell
Get-AuthenticodeSignature .\test.ps1  | select *
```
6. Run the script `./test.ps1`

If you open your script in Notepad or type `cat test.ps1`, you'll see a long block of random letters and numbers. This is the new signature you've just applied. If you change any part of the script, this signature will no longer be valid and display an error when ran. To validate any changes you make, you simply re-sign the certificate as before. Since you added your signing certificate to **Trusted Publishers** in a GPO, all machines in your domain will trust that the script is legitimate.

## Secure Your Private Key
Your private signing key that we exported earlier is very important and shouldn't be shared with anyone. If you wish to use this key on another PC, you'll need to import it. It is highly recommended that you place this file in a secure location, and also add your key to a device that supports Personal Identity Verification (PIV), such as a Yubikey. Using a PIV device allows you to carry your signing key and use it wherever needed, without having to import the key itself.