---
layout: post
title:  "SSL Certificate Management"
date:   2021-05-17 00:00:00 -0400
categories: Security General
tags: security certificate pem crt cer pfx p7b p12 openssl
---

## PEM Format (\*.pem \\ \*\.crt \\ \*\.cer \\ \*.key) {#pem}
The [PEM](https://datatracker.ietf.org/doc/html/rfc1421){:target="_blank"} format is a [base64](https://datatracker.ietf.org/doc/html/rfc4648#section-4){:target="_blank"} encoded file format representing an [X.509](https://datatracker.ietf.org/doc/html/rfc5280){:target="_blank"} certificate, or the certificate you are presented in your browser. If you open the certificate file in a text editor and it looks like a long block of text beginning with `-----BEGIN CERTIFICATE-----`, then it is likely in PEM format. PEM files can also contain private keys, but are usually seperate from their public key counterpart.
### View Public Key {#pem2pk}
```bash
openssl x509 -in mycert.pem -text -noout
```

### View Private Key {#pem2sk}
```bash
# If encrypted, you will be prompted for the private key password
openssl pkey -in mypriv.key -text -noout
```

### Convert to DER (\*.cer \\ *.der) {#pem2der}
```bash
openssl x509 -in mycert.pem -outform der -out mycert.cer
```

### Convert to PKCS7 (\*.p7b) {#pem2p7b}
```bash
openssl crl2pkcs7 -nocrl -certfile mycert.pem -out mycert.p7b
```

### Convert to PKCS12 (\*.pfx) {#pem2pfx}
```bash
# If encrypted, you will be prompted for the private key password
# You will be prompted to provide a password for the PFX file
openssl pkcs12 -export -in mycert.pem -inkey mypriv.key -out mycert.pfx
```

---
## DER Format (\*.cer \\ *.der) {#der}
The [DER](https://datatracker.ietf.org/doc/html/rfc1421){:target="_blank"} format is a binary file representing an [X.509](https://datatracker.ietf.org/doc/html/rfc5280){:target="_blank"} certificate, or the certificate you are presented in your browser. Opening this file in a text editor will display garbage, due to it being binary and not text, like PEM. Your private key will ***not*** be in DER format.
### View Public Key {#der2pk}
```bash
openssl x509 -inform der -in mycert.cer -text -noout
```

### Convert to PEM (\*.pem \\ \*\.crt \\ \*\.cer) {#der2pem}
```bash
openssl x509 -inform der -in mycert.cer -outform pem -out mycert.pem
```

### Convert to PKCS12 (\*.pfx) {#der2pfx}
```bash
# If encrypted, you will be prompted for the private key password
# You will be prompted to provide a password for the PFX file
openssl x509 -inform der -in mycert.cer | openssl pkcs12 -export -inkey mypriv.key -out mycert.pfx
```
---

## PKCS7 Format (\*.p7b) {#p7b}
The [PKCS7](https://datatracker.ietf.org/doc/html/rfc2315){:target="_blank"} format is a [base64](https://datatracker.ietf.org/doc/html/rfc4648#section-4){:target="_blank"} encoded file format typically containing a [certificate revocation list](https://en.wikipedia.org/wiki/Certificate_revocation_list){:target="_blank"} and a [certificate chain](https://knowledge.digicert.com/solution/SO16297.html){:target="_blank"}. The PKCS7 file does ***not*** contain a private key.

### View Public Key {#p7b2pk}
```bash
openssl pkcs7 -in mycert.p7b -print_certs -text -noout
```

### Convert to PEM (\*.pem \\ \*\.crt \\ \*\.cer) {#p7b2pem}
```bash
openssl pkcs7 -in mycert.p7b -print_certs -out mycert.pem
```

---
## PKCS12 Format (\*.pfx) {#pfx}
Can either have a [PFX](https://datatracker.ietf.org/doc/html/rfc7292){:target="_blank"} or P12 file extension. These formats are ***not*** the same. The PFX format is a binary file, while the P12 format is simply a [base64](https://datatracker.ietf.org/doc/html/rfc4648#section-4){:target="_blank"} encoded version of the PFX file. Converting between these two is noted below. The PKCS12 file ***does*** contain a private key.

### View Public Key {#pfx2pk}
```bash
openssl pkcs12 -in mycert.pfx -nokeys | openssl x509 -text -noout
```

### View Private Key {#pfx2sk}
```bash
# You will be prompted for the PFX file password
openssl pkcs12 -in mycert.pfx -nocerts -nodes | openssl pkey -text -noout
```

### Convert to DER (\*.cer \\ *.der) {#pfx2der}
```bash
# Output public key to mycert.cer
openssl pkcs12 -in mycert.pfx -nokeys | openssl x509 -outform der -out mycert.cer
# Output Private key to mypriv.key
# You will be prompted to provide a private key password
openssl pkcs12 -in mycert.pfx -nocerts -out mypriv.key
```

### Convert to PEM (\*.pem \\ \*\.crt \\ \*\.cer \\ \*.key) {#pfx2pem}
```bash
# Output public key to mycert.pem
openssl pkcs12 -in mycert.pfx -nokeys -out mycert.pem
# Output Private key to mypriv.key
# You will be prompted to provide a private key password
openssl pkcs12 -in mycert.pfx -nocerts -aes256 -out mypriv.key
```

### Convert to (\*.p12) {#pfx2p12}
```bash
openssl base64 -in mycert.pfx -out mycert.p12
```

### Convert from (\*.p12) {#p122pfx}
```bash
openssl base64 -d -in mycert.p12 -out mycert.pfx
```