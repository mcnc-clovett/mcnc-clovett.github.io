---
layout: post
title:  "Securing Admins with Smart Cards (Yubikeys)"
date:   2021-03-26 00:00:00 -0400
categories: [Security, Authentication]
tags: [security, gpg, gnupg, pgp, encryption, yubikey, smartcard, certificates, windows, powershell]
---
We all know that giving local admin rights to normal users is a **big no-no**, but what happens if an administrator’s account is compromised? What if a domain admin’s account credentials are obtained? To prevent a potentially catastrophic event, you should use some form of multi-factor authentication to protect those accounts. We all understand that some normal users find difficultly with using multi-factor logins, but as administrators, we don’t really have a choice. *The fate of the network is in our hands.*

One way of providing security is **only using admin credentials when necessary**. Another step on top of that is to use multi-factor authentication. One method that is fairly easy to set up is a smart card implementation. For this example we are going to use the [Yubikey](https://www.yubico.com){:target="_blank"} product, which provides Personal Identity Verification (PIV) functionality. This allows us to store an authentication certificate on a portable key to use to log in to an admin account.

### Example AD Setup
In our example, the Active Directory structure is as follows:
* ***ad.example.com*** (Domain Root)
    * **Technology** (OU)
        * johndoe (User - Supervisor)
        * admin.johndoe (User - Admin Account)
        * *Domain Admins* (Group)
            * *admin.johndoe*
        * *Enrollment Agents* (Group)
            * *johndoe*
        * **No Logon Local** (OU)
            * admin.billybob (User - Admin Account)
            * *Local Admins* (Group)
                * *admin.billybob*

### Install Yubikey Drivers
Push out, by your preferred method, the driver for your smart cards system-wide. This can be through SCCM, GPO or any other method. An example install script for the [Yubikey Smart Card Minidriver](https://www.yubico.com/support/download/smart-card-drivers-tools/){:target="_blank"} is below. You can also get more information from [Yubico's website](https://support.yubico.com/hc/en-us/articles/360015654560-Deploying-the-YubiKey-Minidriver-to-Workstations-and-Servers#Manual-Install){:target="_blank"}.

```powershell
$installPath = "\\ad.example.com\SYSVOL\ad.example.com\scripts\yubikey\YubiKey-Minidriver-4.1.1.210-x64.msi"
if ( !(Get-WmiObject WIN32_PnpSignedDriver | Where-Object { $_.DeviceName -eq "YubiKey Smart Card Minidriver" -and $_.DriverVersion -ge "4.1.1.210"}) -and (Test-Path $installPath) ) {
    msiexec.exe -i $installPath INSTALL_LEGACY_NODE=1 /quiet
} 
```

### Certificate Setup
#### Create CA Templates
1. Add **Enrollment Agents** *group* to **Read** access on the **Enrollment Agent** *template*
2. Duplicate the **Smartcard Logon** template
    1. Under **General** tab:
        * Enter a name for the template (ie. *“Yubikey”*)
        * Enter a validity period (ie. *5 years*)
        * Check **Publish certificate in Active Directory**
        * Check **Do not automatically reenoll if a duplicate certificate exists in Active Directory**
    2. Under **Compatibility** tab:
        * Select the OS where the Certificate Authority resides
        * Select the oldest version of Windows in use on the domain
    3. Under **Request Handling** tab:
        * For **Purpose**, select **Signature and encryption**
        * Check **Include symmetric algorithms allowed by the subject** 
        * Uncheck **Renew with the same key**
        * Check **For automatic renewal of smart card certificates, use the existing key if a new key cannot be created** 
        * Check **Prompt the user during enrollment**
    4. Under **Cryptography** tab:
        * For **Provider category**, click the arrow and select **Key Storage Provider** from the dropdown
        * For **Algorithm name**, select either **RSA**
        * For **Minimum key size**, enter **2048**
        * Check **Requests must use one of the following providers:**
        * Under **Providers**, select **Microsoft Smart Card Key Storage Provider**
        * Click the arrow for **Request hash** and select **SHA256** from the list displayed
    5. Under **Security** tab:
        * For **Group or user names:** Confirm **Authenticated Users** is listed. If is not, click **Add**, enter the name of the group, and then click OK
        * For **Permissions for Authenticated Users**, be sure the option for **Read** is checked
        * Add the **Enrollement Agents** group and check the option for **Read** and **Enroll**
    6. Under **Issuance Requirements** tab:
        * Check **This number of authorized signatures**, and enter **1**
        * For **Policy type required in signature**, select **Application policy**
        * For **Application policy**, select **Certificate Request Agent**
3. Publish both the Enrollment Agent and new Smartcard Logon (Yubikey) templates to the Certificate Authority

#### Create Enrollment Agents
* Open `certmgr.msc`, and under **Personal\Certificates**:
    1. Right click > **All Tasks > Request New Certificate…**
    2. Click through and select the **Enrollment Agent** template, then click **Enroll**

#### Enroll a User Account with a Smart Card
* Also in `certmgr.msc` under **Personal\Certificates**:
    1. Right click > **All Tasks > Advanced Operations**, then select **Enroll on Behalf of**
    2. Click through and select the new smart card template (Yubikey)
    3. Type in the user account you want to enroll (*admin.johndoe*) and click **Enroll**
    4. Enter the **PIN** for the smart card
    5. Open **PowerShell** and enter `certutil -scroots update` to add the Enterprise Root Certificate to the smart card

You should now be able to login with the admin account (*admin.johndoe*) by attempting to open a program like PowerShell as administrator. With the Yubikey inserted into the computer, load an application as administrator to bring up the elevation window as shown below. You may need to click on **More choices** to bring up the smart card entry. As you can see, once the smart card is selected, the assigned user is already entered. You now you be able to type in the PIN for the Yubikey and press enter. You should now be running the application as the admin user assigned to the card.

![smart-card-logon](/images/card-login.png)

### Set the GPOs

#### Set Removal Policy
Should the user need to step away from a machine or remote desktop session, the session should lock. These settings will ensure the session locks when the smart card is removed.

1. Open Group Policy Manager and create a GPO for smart card login at the root of the domain (ie. *Smart Card Policy*), or use the Default Domain Policy
    1. Under **Computer Configuration>Policies>Windows Settings>Security Settings>System Services**:
        * Set **Smart Card Removal Policy** to **Automatic**
            * *(Keep in mind, the service will not auto-start until the device is rebooted. You can also start the service manually if you are unable to reboot a device at the time.)*
    2. Under **Computer Configuration>Policies>Windows Settings>Security Settings>Local Policies>Security Options**:
        * Set **Interactive logon: Smart card removal behavior** to **Disconnect if a remote Remote Desktop Services session**

#### Set Local Administrators
Here we will set what users are allowed to be local administrators domain-wide. Be very selective here and add any other accounts that are absolutely needed. For example, SCCM or other software management groups may need to be added.

1. Under **Computer Configuration>Policies>Windows Settings>Security Settings>Restricted Groups**:
    1. Right click > **Add Group**
    2. Enter **Administrators** and click **OK**
    3. Under **Administrators Properties**, click **Add…**, then add the following:
        * AD\Domain Admins
        * AD\Local Admins
        * Administrator

### Prevent Privileged Users from Logging in Directly
You may wish to prevent certain user accounts from logging in to workstations directly, ie. the Windows Logon screen. A good example would be admin accounts. ***Everyone*** should be using their normal account to login, and only elevate when permissions are needed for a certain task. Any admin or account that you do wish to login to a workstation or server should **NOT** have this GPO applied.

1. Create a new GPO under **Technology\No Logon Local** OU (ie. *No Logon Local Policy*)
    * \* ***WARNING*** \* **Make sure that you *DO NOT* create this GPO in the root of the domain. This must only apply to very specific users that should not be able to log in locally. Whoever this GPO applies to will be immediately logged out.**
2. Under **User Configuration > Policies > Administrative Templates > System**
    1. Set **Custom User Interface** to **Enabled**, under **Interface file name** enter **logoff.exe**, click **OK**

### Force Privileged Users to Use Smart Cards
Now that you have assigned smart cards to your users, you need to force logins to require the card/key. Once this option is selected for each user, they must use the Yubikey to access the assigned admin account.

1. In **Active Directory Users and Computers**, find the users that should only use smart card authentication (*admin.johnsmith* & *admin.billybob*)
2. For each user, under their **Account** tab, check **Smart card required for interactive logon**
