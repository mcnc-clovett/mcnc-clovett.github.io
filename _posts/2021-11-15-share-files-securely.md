---
layout: post
title:  "Best Method to Share Files Securely: GPG"
date:   2021-11-15 00:00:00 -0400
categories: [Security, Encryption]
tags: [security, gpg, gnupg, pgp, encryption]
published: false
---
I'm often asked what is the most secure method for sending files over the internet. Is Google Drive a safe option? How about 
OneDrive or Dropbox? The short answer is no. At any time, your account could be compromised and your valuable data accessed by 
a third party. This may be information requested by your HR or finance department, or perhaps the login credentials to your 
servers or network infrastructure. At no time should this vauable information sit in plain text, anywhere. 

You may be thinking that storing this data in a zip file that you password protect is safe, **not true**. A password protected zip 
file simply protects the data in individual files from being read. Those individual files, however, can be replaced/overwritten 
or deleted from the archive altogether. For instance, a third party could intercept your direct deposit request form, replace it 
with *their* bank info, while you and payroll are none the wiser.

This is why I want to introduce you to `GnuPG` or `gpg` for short. GnuPG, which stands for GNU Privacy Guard, is a tool for 
Windows, macOS and Linux that allows you to create keys that you can use to sign and encrypt files and documents. You can then 
send those files and documents, in any fashion, with full confidence that they arrive as sent and are only accessible by the intended 
party.

---

## How GPG Works

When starting out with GPG, you'll create a **key pair** which consists of a public and private key. The **private** key is used by you 
to sign and encrypt files. The **public** key is used by another party to verify and, along with *their* private key, decrypt the file you 
sent. The public key can, and should, be shared publicly. You can send the public key via email, post it on a web site or even 
upload it to a key server, such as [keys.openpgp.org](https://keys.openpgp.org){:target="_blank"}.

After the initial setup of your keys, signing and encrypting files will be much easier. You can store as many public keys on your 
computer as you like, whether they be from your co-workers or a vendor you deal with. The following links will go over step-by-step 
how to set up GPG on your computer:

| [**Windows (Gpg4win)**](/security/windows-gpg4win.html) | [**macOS (GPGTools)**](/security/macos-gpgtools.html) | [**Linux (gnupg)**](/security/linux-gnupg.html) |