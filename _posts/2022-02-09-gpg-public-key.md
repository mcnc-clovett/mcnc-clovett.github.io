---
layout: post
title:  "GnuPG - Encrypting Files with a Public Key"
date:   2022-02-09 00:00:00 -0400
categories: Security Encryption
tags: security gpg gnupg pgp encryption
---
Encrypting files or data with a public key is called asymmetric encryption. The benefit to this method is the extra security of encrypting data for a specific recipient. This is done by using the person's **public key** to encrypt the data, which they then decrypt with their **private key** that only they have access to. Using this method, you do not have to exchange a password which could compromise the confidential data.

Public keys are kept in a **keyring** on your computer so that you can easily use them again and again. You can also create your own **keypair** to share your public key with others and receive encypted data, as well as use your private key to sign data so people know the data came from you.

Here, we'll walk through encrypting and decrypting data using public key encryption in GnuPG.

### Obtaining Public Keys {#publickeys}
You can obtain someone's public key by any of the typical means including, email, external drive or even physical paper. Sending this key in plaintext is perfectly safe, which is why it is called the public key. You import a key using a couple different methods.

If someone shares the key file with you, you can use the `--import` option in `gpg`.

```bash
~ $ gpg --import myfriend.asc
gpg: key D5A4CF5ABE861374: public key "Christopher Lovett <clovett@mailbox.mcnc.org>" imported
gpg: Total number processed: 1
gpg:               imported: 1

~ $ gpg -k D5A4CF5ABE861374
pub   ed25519/D5A4CF5ABE861374 2021-01-17 [C] [expires: 2023-11-13]
      Key fingerprint = 764F 8DCD BAF1 8D49 7717  88AD D5A4 CF5A BE86 1374
uid                 [ultimate] Christopher Lovett <clovett@mailbox.mcnc.org>
uid                 [ultimate] Christopher Lovett <clovett@mcnc.org>
sub   rsa4096/5BBCA0B2FFC0B65D 2021-01-21 [S] [expires: 2022-05-12]
sub   rsa4096/DE1D6F0A38429A0B 2021-01-21 [E] [expires: 2022-05-12]
sub   rsa4096/5F950E2EA7D452E4 2021-01-21 [A] [expires: 2022-05-12]
```

If the person has stored their key on a [keyserver](https://keys.openpgp.org/), like I have at [https://keys.openpgp.org/](https://keys.openpgp.org/), you may import the key directly within the `gpg` tool.

```bash
~ $ gpg --keyserver hkps://keys.openpgp.org --search-key clovett@mcnc.org
gpg: data source: https://keys.openpgp.org:443
(1)     Christopher Lovett <clovett@mailbox.mcnc.org>
        Christopher Lovett <clovett@mcnc.org>
          256 bit EDDSA key D5A4CF5ABE861374, created: 2021-01-17
Keys 1-1 of 1 for "clovett@mcnc.org".  Enter number(s), N)ext, or Q)uit > 1 # Enter the number of the key you wish to import
gpg: key D5A4CF5ABE861374: public key "Christopher Lovett <clovett@mailbox.mcnc.org>" imported
gpg: Total number processed: 1
gpg:               imported: 1
```

### Encrypting a File {#encryptfile}
GnuPG can only encrypt one file at a time, so if you have many files to send, you'll want to `zip` or `tar` them first. If you're sending a single file, then this is not necessary. Lets say you have a batch of employee data that needs to be sent or stored, for example. Use the tool of your choosing to aggregate the data into a single file (eg. mybulkdata.zip). Next, use `gpg` with option `-e` to encrypt the data. We'll use a few more options as well.

- `-e` or `--encrypt` tells gpg to encrypt with the public key given with *name*.
- `-r name` or `--recipient name` will encrypt for user id *name*. If this option is not specified, GnuPG asks for the user-id. This is the id associated with the public key of the recipient.

```bash
~ $ gpg -e -r clovett@mcnc.org mybulkdata.zip
# You may receive this warning. This is because you haven't signed the recipient's
# public key. If you are sure this is the correct key, you can answer 'y'.
gpg: DE1D6F0A38429A0B: There is no assurance this key belongs to the named user

sub  rsa4096/DE1D6F0A38429A0B 2021-01-21 Christopher Lovett <clovett@mailbox.mcnc.org>
 Primary key fingerprint: 764F 8DCD BAF1 8D49 7717  88AD D5A4 CF5A BE86 1374
      Subkey fingerprint: 4D81 E207 C4B1 6C69 968B  4108 DE1D 6F0A 3842 9A0B

It is NOT certain that the key belongs to the person named
in the user ID.  If you *really* know what you are doing,
you may answer the next question with yes.

Use this key anyway? (y/N) y    # Answer 'y' here if you're sure this is the correct key

~ $ ls mybulkdata.zip*
-rw-r--r--  1 user  385788673 mybulkdata.zip
-rw-r--r--  1 user  385788781 mybulkdata.zip.gpg
```

You can now store or send this confidential data and the recipient can decrypt the file with their private key.

### Encrypting a Short Message {#encryptmessage}
With GnuPG, you can encypt a message without ever writing the data to disk in plaintext. To do so, we'll use the `-o file`, `-e`, `-a` and `-r name` options.

- `-o file` or `--output file` is used to send output to *file*.
- `-e` or `--encrypt` tells gpg to encrypt with the public key given with *name*.
- `-a` or `--armor` to create ASCII armored output. The default is to create the binary OpenPGP format. This is optional, but the output can be easily copy-pasted to email or instant message.
- `-r name` or `--recipient name` will encrypt for user id *name*. If this option is not specified, GnuPG asks for the user-id. This is the id associated with the public key of the recipient.

If you do not provide an input file or any input from the pipleine (`|`), then gpg waits for you to type a message, after which you will press `Ctrl-D` or `Ctrl-Z` to signify you are done. If you used `-o filename.gpg`, then the encrypted data will be stored in *filename.gpg*.

```bash
~ $ gpg -o message.asc -e -a -r clovett@mcnc.org
# You may receive this error. This is because you haven't signed the recipient's
# public key. If you are sure this is the correct key, you can answer 'y'.
gpg: DE1D6F0A38429A0B: There is no assurance this key belongs to the named user

sub  rsa4096/DE1D6F0A38429A0B 2021-01-21 Christopher Lovett <clovett@mailbox.mcnc.org>
 Primary key fingerprint: 764F 8DCD BAF1 8D49 7717  88AD D5A4 CF5A BE86 1374
      Subkey fingerprint: 4D81 E207 C4B1 6C69 968B  4108 DE1D 6F0A 3842 9A0B

It is NOT certain that the key belongs to the person named
in the user ID.  If you *really* know what you are doing,
you may answer the next question with yes.

Use this key anyway? (y/N) y    # Answer 'y' here if you're sure this is the correct key
This is a message I encrypted.    # Enter your message and end with a blank line
# Press <Ctrl-D> on macOS or Linux, <Ctrl-Z><Enter> on Windows

~ $ ls message.gpg 
-rw-r--r--  1 user  106 message.asc
```

You can now send the output file via email or other means. The recipient can decrypt the file with their private key.

### Creating Your Own Keypair {#privatekeys}
Sending encrypted files and verifying signatures are only half of the things you can do with `gpg`. You may need to receive data or sign it yourself. To do this, you'll need your own **keypair**.

You'll see that keys can have expiration dates. These can be updated at any time, even after expiration, provided you still have access to the private key. Expiration dates are used to ensure others update their copy of your key and prevent someone from using a stolen subkey after expiration, among other things. It is good practice to set an expiration *at least* for your subkeys.

>The private key is your identity to others, which they must be able to trust. You should keep the private key safe at all costs. For example, my private master key is stored on physical paper and on USB drive in a fire-proof lock box.
{: .prompt-warning}

#### Quick Key Creation {#quickmethod}
```bash
# Create a new master key that never expires
~ $ gpg --quick-gen-key 'Big Cootie <bigcootie@example.com>' ed25519 cert 0
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.
# GnuPG will prompt for a password to secure the new private key
gpg: key 163F9BB1EB0CC234 marked as ultimately trusted
gpg: revocation certificate stored as '/Users/user/.gnupg/openpgp-revocs.d/CEB8AD11C13AF31E86FB3DF7163F9BB1EB0CC234.rev'
public and secret key created and signed.

pub   ed25519 2021-11-15 [C]
      CEB8AD11C13AF31E86FB3DF7163F9BB1EB0CC234
uid                            Big Cootie <bigcootie@example.com>
```
```bash
# Create signing subkey using the 'fingerprint' of the new master key
# that expires in 6 months
~ $ gpg --quick-add-key CEB8AD11C13AF31E86FB3DF7163F9BB1EB0CC234 ed25519 sign 6m
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.
# GnuPG will prompt for the private key's password from earlier
```
```bash
# Create encryption subkey using the 'fingerprint' of the new master key
# that expires in 6 mounths
~ $ gpg --quick-add-key CEB8AD11C13AF31E86FB3DF7163F9BB1EB0CC234 cv25519 encrypt 6m
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.
# GnuPG will prompt for the private key's password from earlier
```

#### Detailed Key Creation {#detailedmethod}
```bash
~ $ gpg --full-gen-key --expert
gpg (GnuPG) 2.3.3; Copyright (C) 2021 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Please select what kind of key you want:
   (1) RSA and RSA
   (2) DSA and Elgamal
   (3) DSA (sign only)
   (4) RSA (sign only)
   (7) DSA (set your own capabilities)
   (8) RSA (set your own capabilities)
   (9) ECC (sign and encrypt) *default*
  (10) ECC (sign only)
  (11) ECC (set your own capabilities)
  (13) Existing key
  (14) Existing key from card
Your selection? 11    # Enter '11' to select an ECC key
```
```bash
Possible actions for this ECC key: Sign Certify Authenticate 
Current allowed actions: Sign Certify 

   (S) Toggle the sign capability
   (A) Toggle the authenticate capability
   (Q) Finished

Your selection? s    # Enter 's' to turn off signing
```
```bash
Possible actions for this ECC key: Sign Certify Authenticate 
Current allowed actions: Certify 

   (S) Toggle the sign capability
   (A) Toggle the authenticate capability
   (Q) Finished

Your selection? q    # Enter 'q' to finish
```
```bash
Please select which elliptic curve you want:
   (1) Curve 25519 *default*
   (2) Curve 448
   (3) NIST P-256
   (4) NIST P-384
   (5) NIST P-521
   (6) Brainpool P-256
   (7) Brainpool P-384
   (8) Brainpool P-512
   (9) secp256k1
Your selection? 1    # Enter '1' to use a Curve 25519 key
```
```bash
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0) 0    # Enter '0' or your desired expiration period
```
```bash
Key does not expire at all
Is this correct? (y/N) y    # Enter 'y' to finish the key setup
```
```bash
GnuPG needs to construct a user ID to identify your key.

Real name: Big Cootie    # Enter your name
Email address: bigcootie@example.com    # Enter your email address
Comment:     # A comment is optional
```
```bash
You selected this USER-ID:
    "Big Cootie <bigcootie@example.com>"

Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit? o    # Enter 'o' to complete the uid
```
```bash
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.
# GnuPG will prompt for a password to secure the new private key
gpg: key AD551B435E0EA0BC marked as ultimately trusted
gpg: revocation certificate stored as '/Users/user/.gnupg/openpgp-revocs.d/DE6C0EA75D05EA6300BE78D1AD551B435E0EA0BC.rev'
public and secret key created and signed.

pub   ed25519 2021-11-15 [C]
      DE6C0EA75D05EA6300BE78D1AD551B435E0EA0BC
uid                            Big Cootie <bigcootie@example.com>
```

Now create your subkeys, one for signing and one for encryption.
```bash
~ $ gpg --edit-key --expert bigcootie@example.com
gpg (GnuPG) 2.3.3; Copyright (C) 2021 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Secret key is available.

sec  ed25519/AD551B435E0EA0BC
     created: 2021-11-15  expires: never  usage: C   
     trust: ultimate      validity: ultimate
[ultimate] (1). Big Cootie <bigcootie@example.com>

gpg> addkey    # Enter 'addkey' to add a new subkey
```
```bash
Please select what kind of key you want:
   (3) DSA (sign only)
   (4) RSA (sign only)
   (5) Elgamal (encrypt only)
   (6) RSA (encrypt only)
   (7) DSA (set your own capabilities)
   (8) RSA (set your own capabilities)
  (10) ECC (sign only)
  (11) ECC (set your own capabilities)
  (12) ECC (encrypt only)
  (13) Existing key
  (14) Existing key from card
Your selection? 10    # Enter '10' for an ECC signing key
```
```bash
Please select which elliptic curve you want:
   (1) Curve 25519 *default*
   (2) Curve 448
   (3) NIST P-256
   (4) NIST P-384
   (5) NIST P-521
   (6) Brainpool P-256
   (7) Brainpool P-384
   (8) Brainpool P-512
   (9) secp256k1
Your selection? 1    # Enter '1' for Curve 25519
```
```bash
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0) 6m    # Enter '6m' or your desired expiration period
Key expires at Sat May 14 18:46:58 2022 EDT
Is this correct? (y/N) y    # Enter 'y' to confirm
Really create? (y/N) y    # Enter 'y' to be extra sure
```
```bash
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.
# GnuPG will prompt for the private key's password from earlier

sec  ed25519/AD551B435E0EA0BC
     created: 2021-11-15  expires: never  usage: C   
     trust: ultimate      validity: ultimate
ssb  ed25519/E4E10D8C00A4AEC8
     created: 2021-11-15  expires: 2022-05-14  usage: S   
[ultimate] (1). Big Cootie <bigcootie@example.com>

gpg> addkey    # Enter 'addkey' to add a new subkey
```
```bash
Please select what kind of key you want:
   (3) DSA (sign only)
   (4) RSA (sign only)
   (5) Elgamal (encrypt only)
   (6) RSA (encrypt only)
   (7) DSA (set your own capabilities)
   (8) RSA (set your own capabilities)
  (10) ECC (sign only)
  (11) ECC (set your own capabilities)
  (12) ECC (encrypt only)
  (13) Existing key
  (14) Existing key from card
Your selection? 12    # Enter '12' for an ECC encryption key
```
```bash
Please select which elliptic curve you want:
   (1) Curve 25519 *default*
   (2) Curve 448
   (3) NIST P-256
   (4) NIST P-384
   (5) NIST P-521
   (6) Brainpool P-256
   (7) Brainpool P-384
   (8) Brainpool P-512
   (9) secp256k1
Your selection? 1    # Enter '1' for Curve 25519
```
```bash
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0) 6m    # Enter '6m' or your desired expiration period
Key expires at Sat May 14 19:04:06 2022 EDT
Is this correct? (y/N) y    # Enter 'y' to confirm
Really create? (y/N) y    # Enter 'y' to be extra sure
```
```bash
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.
# GnuPG will prompt for the private key's password from earlier

sec  ed25519/AD551B435E0EA0BC
     created: 2021-11-15  expires: never  usage: C   
     trust: ultimate      validity: ultimate
ssb  ed25519/E4E10D8C00A4AEC8
     created: 2021-11-15  expires: 2022-05-14  usage: S   
ssb  cv25519/9CA997BF45733016
     created: 2021-11-15  expires: 2022-05-14  usage: E   
[ultimate] (1). Big Cootie <bigcootie@example.com>

gpg> save    # Enter 'save' to save the changes and exit gpg
```

#### Exporting Your Public Key {#publicexport}
You'll need to share your public key with others for them to send you encrypted data. I recommend using the `-a` and `-o file` along with `--export` to export your public key to a text file.

- `-o file` or `--output file` is used to send output to *file*.
- `-a` or `--armor` to create ASCII armored output.  The default is to create the binary OpenPGP format.
- `--export` to either export all keys from all keyrings (default keyring and those registered via option --keyring), or if at least one name is given, those of the given name.

```bash
~ $ gpg -o bigcootie.pub -a --export bigcootie@example.com
```
You can now share the output file, `bigcootie.pub` in this case, with whoever you need to converse with.

You can also upload your key to a keyserver. Upload the public key file here:

- [https://keys.openpgp.org/upload](https://keys.openpgp.org/upload)

#### Guard Your Keys {#guardthekeys}
You should also create a revocation certificate that you can upload to keyservers if your key is compromised. This certificate and your private key should be backed up offline and secured.

The following will export your private key and create a revocation certificate:

- `-o file` or `--output file` is used to send output to *file*.
- `-a` or `--armor` to create ASCII armored output.  The default is to create the binary OpenPGP format.
- `--export-secret-key` Same as --export, but exports the secret keys instead. The exported keys are written to STDOUT or to the file given with option --output. This command is often used along with the option --armor to allow for easy printing of the key for paper backup; however the external tool paperkey does a better job of creating backups on paper. Note that exporting a secret key can be a security risk if the exported keys are sent over an insecure channel.
- `--gen-revoke name` or `--generate-revocation name` will generate a revocation certificate for the complete key.

```bash
# Export your secret key
~ $ gpg -o bigcootie.sec --export-secret-key -a bigcootie@example.com
# GnuPG will ask for the password to your key so it can be exported

# Create a revocation certificate
~ $ gpg -o bigcootie.rev --gen-revoke bigcootie@example.com

sec  ed25519/163F9BB1EB0CC234 2021-11-15 Big Cootie <bigcootie@example.com>

Create a revocation certificate for this key? (y/N) y    # Enter 'y' to continue
Please select the reason for the revocation:
  0 = No reason specified
  1 = Key has been compromised
  2 = Key is superseded
  3 = Key is no longer used
  Q = Cancel
(Probably you want to select 1 here)
Your decision? 0    # Enter '0' or '1' reason for the revocation
Enter an optional description; end it with an empty line:
>     # A description is optional
Reason for revocation: No reason specified
(No description given)
Is this okay? (y/N) y    # Enter 'y' to continue
ASCII armored output forced.
Revocation certificate created.

Please move it to a medium which you can hide away; if Mallory gets
access to this certificate he can use it to make your key unusable.
It is smart to print this certificate and store it away, just in case
your media become unreadable.  But have some caution:  The print system of
your machine might store the data and make it available to others!
```

### Decrypting a File {#decryptfile}
Decrypting is a straightforward process. When you receive a file that is encrypted with a password, simply run `gpg` with the `-d` and `-o file` options. This will prompt for the password and store the location specified by *file*. If the data is a simple text file that you just wish to output to the terminal, you can leave off the `-o` option.

- `-d` or `--decrypt` will decrypt the file given on the command line (or STDIN if no file is specified) and write it to STDOUT (or the file specified with --output). If the decrypted file is signed, the signature is also  verified.
- `-o file` or `--output file` is used to send output to *file*.

```bash
# For binary files or data you wish to save to disk...

~ $ gpg -d -o mybulkdata.zip mybulkdata.zip.gpg
gpg: encrypted with cv25519 key, ID 24FDDB38B2CF1DE8, created 2021-11-15
      "Big Cootie <bigcootie@example.com>"
# GnuPG will prompt for a password
```
```bash
# For simple text you want to display to the terminal...

~ $ gpg -d message.gpg
gpg: encrypted with cv25519 key, ID 24FDDB38B2CF1DE8, created 2021-11-15
      "Big Cootie <bigcootie@example.com>"
# GnuPG will prompt for a password
This is a message I encrypted.
```
