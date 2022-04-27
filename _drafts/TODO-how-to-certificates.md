# How to manage SSL certificates

## The files

- `.pem` files are generally the public key, used by the client to verify and decrypt data sent by servers.

### Read content of pem certificate

```sh
openssl x509 -noout -text -in certificate.pem
# or
keytool -printcert -file certificate.pem
```

## Common SSL certificates

SSL must be configured to support TLS version 1.1 or higher. TLS version 1.2 using AES 256 or higher with SHA-256 is recommended.

## SAN vs CN

SAN is an extension to the X.509 specification that allows users to specify additional host names for a single SSL certificate. Each of these names will be considered protected by the SSL certificate.

This allows to support multiple host names using the same certificate. For example it is logical to support both of the following host names using the same SSL certificate.

```
www.example.com
example.com
```

Originally, SSL certificates only allowed the designation of a single host name in the certificate subject called Common Name (CN) but now this has undergone change and a certificate is first verified for SAN and if no SAN is defined it falls back to CN.

It is still a practice to define both CN and SAN when requesting a certificate. An important point is that CN and SAN are not complimentary and any CN defined should be a subset of SAN list.

## Certificate Signing Request (CSR)

The workflow for getting a certificate is the following:

1. Create a Certificate Signing Request (CSR) and a private key file.
2. Send the CSR to a Certifying Authority (CA).
3. CA issues the certificate based on the CSR.

There are many ways to generate the CSR but the common way is to use OpenSSL.

When you launch openSSL it looks for a `openssl.cnf` file – this is the master configuration file for openSSL. It is quite something to understand but this page explains it nicely – https://www.phildev.net/ssl/opensslconf.html

This file is not part of the binaries and needs to be created. The path to the .cnf file needs to be defined for your environment. `/usr/lib/ssl/openssl.cnf` is the location where Ubuntu places openssl.cnf for the OpenSSL they provide.

Once the environment is ready we can begin creating the CSR. For this is better to create a config file to avoid having to type out the details on the command line:

```ini
[req]
default_bits = 2048
prompt = no
default_md = sha256
req_extensions = req_ext
distinguished_name = dn

[ dn ]
C = <2 letter country code>
ST = <state>
L = <location>
O = <org>
OU = <org unit>
CN = <common name>

[ req_ext ]
subjectAltName = @alt_names

[ alt_names ]
DNS.1 = <at least the CN>
```

In the config file ebove we have defined that the CSR should use 2048 bit encryption and the sha256 hash algorithm.

The `[dn]` section defines the certificate properties. The `[req_ext]` section contains the extensions like SAN.

NOTE: SAN should always be defined.

Once the config file has been defined, we can generate the CSR and the private key file using the following command:

```sh
openssl req \
  -new -nodes -sha256 -out mycsr.csr \
  -newkey rsa:2048 -keyout myprivatekey.key \
  -config myconfig.cfg
```

## Self-signed certificates

### Create a self-signed certificate

Using SAN:

```sh
openssl req \
  -newkey rsa:4096 -nodes -sha256 -keyout domain.key \
  -addext "subjectAltName = DNS:mysite.domain.com" \
  -x509 -days 365 -out domain.crt
```

Using CN:

```sh
openssl req \
  -newkey rsa:4096 -nodes -keyout cert.key \
  -subj /CN=mysite.domain.com -set_serial 2 \
  -x509 -days 365 -out cert.pem
```

Where:

`-newkey arg` - This option is used to generate a new private key. The argument takes one of several forms. `[rsa:]nbits` like `-newkey rsa:4096` generates an RSA key nbits in size. If nbits is omitted, i.e., -newkey rsa is specified, the default key size specified in the configuration file with the default_bits option is used if present, else 2048.
`-nodes` (DEPRECATED since OpenSSL 3.0) or `-noenc` - If this option is specified then if a private key is created it will not be encrypted.
`-<digest>` like `-sha256`  - This specifies the message digest to sign the request. Any digest supported by the OpenSSL dgst command can be used. This overrides the digest algorithm specified in the configuration file. From dgst manpage: the default digest is sha256.
`-x509` - This option outputs a certificate instead of a certificate request. This is typically used to generate test certificates. It is implied by the -CA option. This option implies the -new flag if -in is not given.
`-days n` - When -x509 is in use this specifies the number of days to certify the certificate for, otherwise it is ignored. n should be a positive integer. The default is 30 days.
`-addext ext` -  Add a specific extension to the certificate (if -x509 is in use) or certificate request. The argument must have the form of a key=value pair as it would appear in a config file. This option can be given multiple times.
`-subj arg` - Sets subject name for new request or supersedes the subject name when processing a certificate request. The arg must be formatted as /type0=value0/type1=value1/type2=... . Giving a single / will lead to an empty sequence of RDNs (a NULL-DN). Multi-valued RDNs can be formed by placing a + character instead of a / between the AttributeValueAssertions (AVAs) that specify the members of the set.
`-set_serial n` - Serial number to use when outputting a self-signed certificate. This may be specified as a decimal value or a hex value if preceded by 0x. If not given, a large random number will be used.

More info: https://www.openssl.org/docs/manmaster/man1/openssl-req.html

### Create a self-signed certificate for aspnet core

```sh
PARENT="mysite.dev.test"
openssl req \
-x509 \
-newkey rsa:4096 \
-sha256 \
-days 365 \
-nodes \
-keyout $PARENT.key \
-out $PARENT.crt \
-subj "/CN=${PARENT}" \
-extensions v3_ca \
-extensions v3_req \
-config <( \
  echo '[req]'; \
  echo 'default_bits= 4096'; \
  echo 'distinguished_name=req'; \
  echo 'x509_extension = v3_ca'; \
  echo 'req_extensions = v3_req'; \
  echo '[v3_req]'; \
  echo 'basicConstraints = CA:FALSE'; \
  echo 'keyUsage = nonRepudiation, digitalSignature, keyEncipherment'; \
  echo 'subjectAltName = @alt_names'; \
  echo '[ alt_names ]'; \
  echo "DNS.1 = www.${PARENT}"; \
  echo "DNS.2 = ${PARENT}"; \
  echo '[ v3_ca ]'; \
  echo 'subjectKeyIdentifier=hash'; \
  echo 'authorityKeyIdentifier=keyid:always,issuer'; \
  echo 'basicConstraints = critical, CA:TRUE, pathlen:0'; \
  echo 'keyUsage = critical, cRLSign, keyCertSign'; \
  echo 'extendedKeyUsage = serverAuth, clientAuth')

openssl x509 -noout -text -in $PARENT.crt
```


## Trust a certificate at the OS level

Ubuntu:
```sh
sudo cp $PWD/domain.crt /usr/local/share/ca-certificates/mysitedomain.com.crt
update-ca-certificates
```

CentOS:
```sh
sudo cp $PWD/domain.crt /etc/pki/ca-trust/source/anchors/mysitedomain.com.crt
update-ca-trust
```
