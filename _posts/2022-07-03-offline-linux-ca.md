---
layout: post
title:  "Offline Root Certificate Authority - Linux"
date:   2022-07-03 00:00:00 -0400
categories: [Services, Certificate Authorities]
tags: [linux, certificate, security]
---
Every organization that has an Active Directory structure, or any other service that uses SSL/TLS services (ie. HTTPS, RDP, etc.), should have a certificate authority. This certificate authority is used to verify that you are indeed talking directly with the server you think you are and that the connection is secure. Having a certificate authority is also required to enable secure LDAP (LDAPS), though this can also be done with an [external certificate authority](https://docs.microsoft.com/en-us/troubleshoot/windows-server/identity/enable-ldap-over-ssl-3rd-certification-authority){:target="_blank"}.

Choosing to host your offline root CA on a Linux-based system, rather than Windows Server, will save you from licensing a server that will rarely be turned on.

## Why have an **offline** Root CA?
> If a root CA is in some way compromised (broken into, hacked, stolen, or accessed by an unauthorized or malicious person), then all of the certificates that were issued by that CA are also compromised. Since certificates are used for data protection, identification, and authorization, the compromise of a CA could compromise the security of an entire organizational network. For that reason, many organizations that run internal PKIs install their root CA offline. That is, the CA is never connected to the company network, which makes the root CA an offline root CA. Make sure that you keep all CAs in secure areas with limited access.[^fn-rootCA] <br><br>
*-- Microsoft TechNet*

## System Requirements
* Linux-based, single-purpose physical server
    * You will need to be able to transfer files to and from this device via USB drive.
    * We'll be using Ubuntu 22.04 LTS Server for this example.
    * This system will be used as an offline root CA, therefore it will be powered off after we're done.
* USB drive formatted with ExFAT or FAT32 file system

## Ensure OpenSSL is Installed
1. Update the server
```shell
sudo apt update && sudo apt dist-upgrade -y
```
2. Install OpenSSL *(likely already installed)*
```shell
sudo apt install -y openssl
```

---

> At this point, there is no need for the server to be connected to the network. You should remove the network cable from the ethernet port. If this is a device with a wireless card, ensure it is not connected to the wireless or even remove the WLAN card.
{: .prompt-warning }
---

## Create Certificate Authority Directory Structure
1. Create all directories that will be needed
```shell
cd /etc/ssl
sudo mkdir -p root-ca/{private,newcerts,certs,crl,csr}
sudo chmod 700 root-ca/private
```
    - `-p` Create the parent directories if they don't exist
    - `700` Set the permissions so that only the file owner can read/write `(rwx------)`

2. Create needed database files
```shell
cd root-ca
sudo openssl rand -out serial -hex 16
sudo openssl rand -out crlnumber -hex 16
sudo touch index
```

3. Create the configuration file

```shell
sudo echo '[ca]
#/etc/ssl/root-ca/openssl.cnf
#see man ca
default_ca                      = CA_default

[CA_default]
dir                             = /etc/ssl/root-ca
certs                           = $dir/certs
crl_dir                         = $dir/crl
new_certs_dir                   = $dir/newcerts
database                        = $dir/index
serial                          = $dir/serial
RANDFILE                        = $dir/private/.rand

private_key                     = $dir/private/ca.key
certificate                     = $dir/certs/ca.crt

crlnumber                       = $dir/crlnumber
crl                             = $dir/crl/ca.crl
crl_extensions                  = crl_ext
default_crl_days                = 30

default_md                      = sha256

name_opt                        = ca_default
cert_opt                        = ca_default
default_days                    = 397
preserve                        = no
policy                          = policy_loose

[ policy_strict ]
countryName                     = match
stateOrProvinceName             = match
organizationName                = match
organizationalUnitName          = optional
commonName                      = supplied
emailAddress                    = optional

[ policy_loose ]
countryName                     = optional
stateOrProvinceName             = optional
localityName                    = optional
organizationName                = optional
organizationalUnitName          = optional
commonName                      = supplied
emailAddress                    = optional

[ req ]
# Options for the req tool, man req.
default_bits                    = 2048
distinguished_name              = req_distinguished_name
string_mask                     = utf8only
default_md                      = sha256
# Extension to add when the -x509 option is used.
x509_extensions                 = v3_ca

[ req_distinguished_name ]
countryName                     = Country Name (2 letter code)
stateOrProvinceName             = State or Province Name
localityName                    = Locality Name (City or Town)
0.organizationName              = Organization Name (Company Name)
organizationalUnitName          = Organizational Unit Name (Department)
commonName                      = Common Name (Descriptive Name)
emailAddress                    = Email Address

[ v3_ca ]
# Extensions to apply when createing root ca
# Extensions for a typical CA, man x509v3_config
subjectKeyIdentifier            = hash
authorityKeyIdentifier          = keyid:always,issuer
basicConstraints                = critical, CA:true
keyUsage                        = critical, digitalSignature, cRLSign, keyCertSign

[ v3_intermediate_ca ]
# Extensions to apply when creating intermediate or sub-ca
# Extensions for a typical intermediate CA, same man as above
subjectKeyIdentifier            = hash
authorityKeyIdentifier          = keyid:always,issuer
# pathlen:0 ensures no more sub-ca can be created below an intermediate
basicConstraints                = critical, CA:true, pathlen:0
keyUsage                        = critical, digitalSignature, cRLSign, keyCertSign

[ server_cert ]
# Extensions for server certificates
basicConstraints                = CA:FALSE
nsCertType                      = server
nsComment                       = "OpenSSL Generated Server Certificate"
subjectKeyIdentifier            = hash
authorityKeyIdentifier          = keyid,issuer:always
keyUsage                        = critical, digitalSignature, keyEncipherment
extendedKeyUsage                = serverAuth

[ crl_ext ]
# CRL extensions.
# Only issuerAltName and authorityKeyIdentifier make any sense in a CRL.
authorityKeyIdentifier=keyid:always' | sudo tee openssl.cnf
```

## Create the Root CA Certificate
1. Generate the new private key and certificate for the root CA
```shell
sudo openssl req -new -newkey rsa:4096 -x509 -config /etc/ssl/root-ca/openssl.cnf -extensions v3_ca -days 7300 -out certs/ca.crt -keyout private/ca.key
```
    - `req` This is a certificate request
    - `-new` This is a new request for a certificate
    - `-newkey` The new private key will be a 4096-bit [RSA](https://en.wikipedia.org/wiki/RSA_(cryptosystem)){:target="_blank"} key
    - `-x509` Generate a certificate (self-signed) instead of a certificate request
    - `-config` Location of the config file to use
    - `-extensions` Name of the extention to use (defined in the config file)
    - `-days` Length of certificate life in days (7300 days = 20 years)
    - `-out` Location of the new root CA certificate file
    - `-keyout` Location of the private key file for the root CA ***(Keep this safe!)***

2. Provide a password for the new root CA key ***(document this)*** and provide DN information. Only the Common Name field is required, but all are recommended.
```shell
Enter PEM pass phrase:
Verifying - Enter PEM pass phrase:
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) []:US
State or Province Name []:Full State Name
Locality Name (City or Town) []:Your city
Organization Name (Company Name) []:Your Organization Name
Organizational Unit Name (Department) []:Your Department Name
Common Name (Descriptive Name) []:Your Org Root CA
Email Address []:
```

    > Do not lose the `ca.key` file **or the password to it**
    {: .prompt-danger }

3. Review the certificate you've created.
```shell
openssl x509 -in certs/ca.crt -text -noout
```
    - `x509` This is a x509 certificate operation
    - `-in` The certficate file we're examining
    - `-text` Display the certificate in text form
    - `-noout` Don't display the certificate in base64 form

- The output of the above command should look similar to this:
```
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            2b:06:96:df:af:57:e8:c4:5e:74:0a:fc:24:c4:da:88:ae:42:21:a3
        Signature Algorithm: sha256WithRSAEncryption
        Issuer: C = US, ST = North Carolina, L = RTP, O = MCNC, OU = CNE, CN = MCNC Root CA
        Validity
            Not Before: Jul  3 22:43:56 2022 GMT
            Not After : Jun 28 22:43:56 2042 GMT
        Subject: C = US, ST = North Carolina, L = RTP, O = MCNC, OU = CNE, CN = MCNC Root CA
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (4096 bit)
                Modulus:
                    00:9c:2b:3f:e5:02:ee:4c:f4:ba:61:07:f0:63:a7:
                    f7:30:26:5f:32:af:c8:38:20:93:1d:62:35:98:5e:
                    20:d4:17:e1:3f:db:86:c8:99:e3:69:75:1c:d2:99:
                    ...
```

## Generate the Certificate Revocation List
1. Create the CRL and sign it using the root CA private
```shell
sudo openssl ca -config /etc/ssl/root-ca/openssl.cnf -gencrl -out crl/ca.crl
```
    - `ca` This CA operation
    - `-config` Location of the config file to use
    - `-gencrl` Generate a certificate revocation list signed by the private key
    - `-out` File to save the CRL to

2. You will be asked to provide the password for the root CA private key
```
Using configuration from /etc/ssl/root-ca/openssl.cnf
Enter pass phrase for /etc/ssl/root-ca/private/ca.key:
```

## Copy the Root CA Certificate and CRL from the Server
1. Insert your USB drive
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

5. Copy the root CA certificate and CRL to the drive
```shell
sudo cp /etc/ssl/root-ca/certs/ca.crt /etc/ssl/root-ca/crl/ca.crl /media
```

6. Unmount the USB drive **before** removing it
```shell
sudo umount /media
```

    > If creating a Windows Enterprise Subordinate CA, you will need both the `ca.crl` and `ca.crt` files to import into Active Directory
    {: .prompt-tip }

## Shutdown and Store the Machine
You may now shutdown the server and store it. The only times the server will be needed are when you stand up a new subordinate (intermediate) certificate authority, or when a sub-CA needs its certificate renewed. Sub-CA's should renew their certificates every ten years or so.

## Footnotes
[^fn-rootCA]: <https://social.technet.microsoft.com/wiki/contents/articles/2900.offline-root-certification-authority-ca.aspx>