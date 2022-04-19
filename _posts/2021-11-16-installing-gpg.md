---
layout: post
title:  "GnuPG - Intro & Installation"
date:   2021-11-16 00:00:00 -0400
categories: [Security, Encryption]
tags: [security, gpg, gnupg, pgp, encryption]
---
You can skip right to the installation for [Windows](#windows) or [macOS](#macos). GnuPG is included in all major Linux distributions.

## Introduction {#intro}
**GnuPG** or [**"GNU Privacy Guard"**](https://gnupg.org/) is a free and open source implementation of the [OpenPGP standard](https://www.ietf.org/rfc/rfc4880.txt). GnuPG was first released in 1997, and has been actively developed and updated ever since. It is primarily used to secure communications between two or more people, however, you can encrypt any file for personal storage as well. Encryption can be done with the latest secure methods either by [public key cryptography](https://en.wikipedia.org/wiki/Public-key_cryptography) via [RSA](https://en.wikipedia.org/wiki/RSA_(cryptosystem)) or [ECC](https://en.wikipedia.org/wiki/Elliptic-curve_cryptography), or [symmetric cryptography](https://en.wikipedia.org/wiki/Symmetric-key_algorithm) via [AES](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard) or other ciphers. 

### Public Key Cryptography
Public key cryptography, also known as asymmetric cryptography, works by using someone else's public key to encrypt information that they can decrypt with their private key. The idea is that *only* that person has possession and control of their private key, therefore you can be sure that they are the *only* person that can access the data. However, you should be sure that the public key you are using actually belongs to the intended recipient.

As a side note, you can use your private key to sign data so that the recipient knows that you sent it and that it hasn't changed since you signed it.

![Public Key Encryption](/images/security/Public_key_encryption.png)

### Symmetric Key Cryptography
Symmetric key cryptography works by setting a password, and using that password in some cipher to encrypt data. The recipient would then use that password to decrypt the data. The issue here is that the recipient must first know the password. The other concern is that the recipient may not be the only one who knows the password, also the password could be brute-forced. As of this writing, GnuPG uses the AES256 cipher, so you know your data is secured, as long as the password isn’t compromised.

## Installation on Windows {#windows}
GnuPG can be installed on Windows through the [official installer](https://gnupg.org/ftp/gcrypt/binary/) or via [Gpg4Win](https://gpg4win.org/download.html). We'll covering the official installer, as that is all that is needed for basic functionality. If you would like a graphical interface, you can use Gpg4Win if you choose.

1. Visit [https://gnupg.org/ftp/gcrypt/binary/](https://gnupg.org/ftp/gcrypt/binary/) in your browser and select the most recent version.
    - As of this writing the lastest installer is `gnupg-w32-2.3.3_20211012.exe`
2. Run the installer. You may need to provide administrator access.
3. Open a terminal or powershell window.
4. Type in `gpg --version`
    - You should get something similar to:

```console
PS C:\Users\User> gpg --version
gpg (GnuPG) 2.3.3
libgcrypt 1.9.4
Copyright (C) 2021 g10 Code GmbH
License GNU GPL-3.0-or-later <https://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Home: C:\Users\User\AppData\Roaming\gnupg
Supported algorithms:
Pubkey: RSA, ELG, DSA, ECDH, ECDSA, EDDSA
Cipher: IDEA, 3DES, CAST5, BLOWFISH, AES, AES192, AES256, TWOFISH,
        CAMELLIA128, CAMELLIA192, CAMELLIA256
AEAD: EAX, OCB
Hash: SHA1, RIPEMD160, SHA256, SHA384, SHA512, SHA224
Compression: Uncompressed, ZIP, ZLIB, BZIP2
```

## Installation on macOS {#macos}
GnuPG can be installed on macOS through [Homebrew](https://brew.sh/), via [GPGSuite](https://gpgtools.org/) or [gpgOSX](https://sourceforge.net/p/gpgosx/docu/Download/). I'll be covering the Homebrew method, as that is all that is needed for basic functionality. If you would like a graphical interface, you can use GPGSuite if you choose.

1. If Homebrew is not installed, follow the instructions at [https://brew.sh](https://brew.sh/)
2. Once Homebrew is installed run `brew install gnupg` in a Terminal window.
3. Now that GnuPG is installed, run `gpg --version` in the terminal.
    - You should get something similar to:

```console
~/ > gpg --version
gpg (GnuPG) 2.3.3
libgcrypt 1.9.4
Copyright (C) 2021 Free Software Foundation, Inc.
License GNU GPL-3.0-or-later <https://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Home: /Users/User/.gnupg
Supported algorithms:
Pubkey: RSA, ELG, DSA, ECDH, ECDSA, EDDSA
Cipher: IDEA, 3DES, CAST5, BLOWFISH, AES, AES192, AES256, TWOFISH,
        CAMELLIA128, CAMELLIA192, CAMELLIA256
AEAD: EAX, OCB
Hash: SHA1, RIPEMD160, SHA256, SHA384, SHA512, SHA224
Compression: Uncompressed, ZIP, ZLIB, BZIP2
```