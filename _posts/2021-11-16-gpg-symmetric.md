---
layout: post
title:  "GnuPG - Encrypting Files with a Password"
date:   2021-11-16 00:00:00 -0400
categories: [Security, Encryption]
tags: [security, gpg, gnupg, pgp, encryption]
---
Encrypting files or data with a password is called symmetric encryption. The benefit to this method is the speed of the encryption and decryption process, so it is more suited to large files and data in bulk. However, the downside is that the password must be shared by some other means, whether by email, instant message, phone or in person. Also, anyone who gains access to the password will be able to access the data.

This is very similar to a [password protected zip file](https://en.wikipedia.org/wiki/ZIP_(file_format)#Encryption). The difference is while a protected zip file cannot have its contents viewed, you can replace a file within it without the recipient knowing. Also, zip uses encryption methods that are [known to be vulnerable](https://math.ucr.edu/~mike/zipattacks.pdf), whereas GnuPG uses the widely used AES256 cipher. It is better to zip your data and then encrypt it with the `gpg` tool to avoid this issue.

Here, we'll walk through encrypting and decrypting data using symmetric encryption in GnuPG.

### Encrypting a Short Message {#encryptmessage}
With GnuPG, you can encypt a message without ever writing the data to disk in plaintext. To do so, we'll use the `-o file` and `-c` options.

- `-o file` or `--output file` is used to send output to *file*.
- `-c` or `--symmetric` tells gpg to encrypt with a symmetric cipher using a passphrase.

If you do not provide an input file or any input from the pipleine (`|`), then gpg asks for the new passphrase and waits for you to type a message, after which you will press `Ctrl-D` to signify you are done. If you used `-o filename.gpg`, then the encrypted data will be stored in *filename.gpg*.

```bash
$ gpg -o message.gpg -c
# GnuPG will prompt for a password
This is a message I encrypted.    # Enter your message and end with a blank line
# Press <Ctrl-D> on macOS or Linux, <Ctrl-Z><Enter> on Windows

$ ls message.gpg 
-rw-r--r--  1 user  106 message.gpg
```

You can now send the output file via email or other means, but remember you'll need to provide the other person the password.

### Encrypting a File {#encryptfile}
GnuPG can only encrypt one file at a time, so if you have many files to send, you'll want to `zip` or `tar` them first. If you're sending a single file, then this is not necessary. Lets say you have a large batch of employee data that needs to be sent or stored, for example. Use the tool of your choosing to aggregate the data into a single file (eg. mybulkdata.zip). Next, use `gpg` with option `-c` to encrypt the data.

- `-c` or `--symmetric` tells gpg to encrypt with a symmetric cipher using a passphrase.

```bash
$ gpg -c mybulkdata.zip
# GnuPG will prompt for a password
$ ls mybulkdata.zip*
-rw-r--r--  1 user  385788673 mybulkdata.zip
-rw-r--r--  1 user  385788781 mybulkdata.zip.gpg
```

You can now store or send this confidential data and inform the recipient of the password so that they can decrypt it.

### Decrypting a File {#decryptfile}
Decrypting is a straightforward process. When you receive a file that is encrypted with a password, simply run `gpg` with the `-d` and `-o file` options. This will prompt for the password and store the location specified by *file*. If the data is a simple text file that you just wish to output to the terminal, you can leave off the `-o` option.

- `-d` or `--decrypt` will decrypt the file given on the command line (or STDIN if no file is specified) and write it to STDOUT (or the file specified with --output). If the decrypted file is signed, the signature is also  verified.
- `-o file` or `--output file` is used to send output to *file*.

```bash
# For binary files or data you wish to save to disk...

$ gpg -d -o mybulkdata.zip mybulkdata.zip.gpg
gpg: AES256.CFB encrypted data
# GnuPG will prompt for a password
gpg: encrypted with 1 passphrase
```
```bash
# For simple text you want to display to the terminal...

$ gpg -d message.gpg
gpg: AES256.CFB encrypted data
# GnuPG will prompt for a password
gpg: encrypted with 1 passphrase
This is a message I encrypted.
```
