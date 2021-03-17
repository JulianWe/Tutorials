# Tutorials

Authentication Guide:
https://docs.vmware.com/en/VMware-vSphere/7.0/vsphere-esxi-vcenter-server-70-authentication-guide.pdf

https://dadhacks.org/2017/12/27/building-a-root-ca-and-an-intermediate-ca-using-openssl-and-debian-stretch/

curl http://dadhacks.org/wp-content/uploads/2017/12/openssl_root.cnf_.txt --out openssl_root.cnf


**Generate Root private key**
```sh
openssl genrsa -aes256 -out private/ca.vdi.sclabs.net.key.pem 2048
```

**Signing the root certificate**
```sh
openssl req -config openssl_root.cnf -new -x509 -sha512 -extensions v3_ca -key /root/ca/private/ca.vdi.sclabs.net.key.pem -out /root/ca/certs/ca.vdi.sclabs.net.crt.pem -days 3650 -set_serial 0
```

**Creating an Intermediate Certificate Authority**
```sh
mkdir /root/ca/intermediate
cd /root/ca/intermediate
mkdir certs newcerts crl csr private
touch index.txt
touch index.txt.attr
echo 1000 > /root/ca/intermediate/crlnumber
echo '1234' > serial

curl http://dadhacks.org/wp-content/uploads/2017/12/openssl_intermediate.cnf_.txt --out openssl_intermediate.cnf
```

**creating the private key and certificate signing rewuest for the intermediate ca**
```sh
openssl req -config openssl_intermediate.cnf -new -newkey 2048 -keyout /root/ca/intermediate/private/int.vdi.sclabs.net.key.pem -out /root/ca/intermediate/csr/int.vdi.sclabs.net.csr
```
**creating the intermediate certificate**
```sh
openssl ca -config openssl_root.cnf -extensions v3_intermediate_ca -days 3650 -notext -md sha512 -in /root/ca/intermediate/csr/int.vdi.sclabs.net.csr -out /root/ca/intermediate/certs/int.vdi.sclabs.net.crt.pem
```
**creating the certificate chain:**
```sh
cat intermediate/certs/int.vdi.sclabs.net.crt.pem certs/ca.vdi.sclabs.net.crt.pem > intermediate/certs/chain.vdi.sclabs.net.crt.pem
```

**What are all these files for?**

So now that you have created all these files, which ones are the ones you need?

In /root/ca/certs, ca.DOMAINNAME.crt.pem is the Root CA certificate.
In /root/ca/intermediate/certs, int.DOMAINNAME.crt.pem is the Intermediate CA certificate.
In /root/ca/intermediate/certs, chain.DOMAINNAME.crt.pem is the concatenation of the Root CA certificate and the Intermediate CA certificate.

```sh
curl http://dadhacks.org/wp-content/uploads/2017/12/openssl_csr_san.cnf_.txt --out openssl_csr_san.cnf
```
-> RSA

**creating server certificate signing request**
```sh
openssl req -out intermediate/csr/jw-vcsa7.vdi.sclabs.net.csr.pem -newkey rsa:2048 -nodes -keyout intermediate/private/jw-vcsa7.vdi.sclabs.net.key.pem -config openssl_csr_san.cnf
```

**creating the server certificate by signing the signing the signing request with the intermediate ca**
```sh
openssl ca -config openssl_intermediate.cnf -extensions server_cert -days 3750 -notext -md sha512 -in intermediate/csr/jw-vcsa7.vdi.sclabs.net.csr.pem -out intermediate/certs/jw-vcsa7.vdi.sclabs.net.crt.pem
```

**creating a combined certificate for use with Apache server**
```sh
openssl pkcs12 -inkey private/ca.vdi.sclabs.net.key.pem -in certs/ca.vdi.sclabs.net.crt.pem -export -out jw-vcsa7.vdi.sclabs.net.combinedcertchain.pfx
openssl pkcs12 -in jw-vcsa7.vdi.sclabs.net.combinedcertchain.pfx -nodes -out jw-vcsa7.vdi.sclabs.net.combined.crt
```




-----BEGIN PRIVATE KEY-----
MIIEvgIBADANBgkqhkiG9w0BAQEFAASCBKgwggSkAgEAAoIBAQCw7fWKtM6W4/y0
DcbrFRoBK9dQCIobFK8VIL0g2WsnTEqbnHT88q5wewuD/nPWjFjxAA2m3rwsqKj0
5Kixi823Dv/KcHkXk6EkfMfHOZ7xjcXF43WJklnZlNS1wBA2/e110qhqPkylfYuV
BJ49PrYEFXJpQjEltSN87soVHeFfWc/nwqyPaYcxXC+Rr7xVA9cwG+C7ir/6JSUb
2pkIajbCZIs2/8koEuM5lj4DTurN03hv4NfT681qPYuvOitVYNPcU6L605+vAfqc
mx+L/wA+bjBieDIjky/I0EJQd9t74BH3T8U2dm4AMKCrNTib1j+KwRI2o3WMSQ7+
KOHteCpVAgMBAAECggEASMaIgjZe56f9kN493QKAAM1UskHg9MSsQ6eEw9dKgQ6b
fah8YnM8F141XWSzpyNxjif0dZgWlNQHMzw+u1EDG/Iaet2KoY0C8mw1DJiB7V/g
YsZt2VmOhbX3TI8k3EnUe+tbhN/9TPD4EiKlKBH8cm+T8QHeD2GTqFbcXpU816f9
YzEyfRp0fVqCy41H7nFT1ybRiR7XwiRJ9a8M8tRM3fW0uWHQu0rYFWvzOqjp5FSZ
ybU6wUCX1THCdnU+IhFUlU/XyeNImMhZiRyESqUjS1zoSaQ7SnhNq/dJuX17yhbn
gy9fXMV9hkw2oLr1npcL/gtmAF2pEouU7M+as5HQ4QKBgQDeKzBLLmeOhbqX81lG
6sMaVprfP9fRTpPBBxB4k/B3JjrvP1twkq9Uy8ypSKY+gTywHBV82K+srLKI403w
TVZG6PNvHaMhVB8gjt1c7eZjdPcJ3nEcTi1Md4jHJvWXeX4u6S6h8zLwIDFXoY4D
MzimfDLXCIT58LHzGIagwtpKrQKBgQDL3zW5m+pqBhOu3ZgJoZZPFVOHjo49EwYE
jhvNE2/YrbdgqQOzS0p45cRonJKAsqw+ZUvjNxHWtPZCZK2wmLBpgNVLdzRmzYUq
LBKqYXTL7GHtyYwiJN6UN7vWIdpEnwNGqgu6KZYSRV3Qsn9Kg68h8KgUm3AN9UBC
D2UVAW87SQKBgQCAAkB8QQuX+gOOQ7+f9epehaIMmht+1RibMrfR0ePOsy9n5IiK
L2pooFiW/W4UO6C9FCFpYuytwH/KEbY5jEX264g/8MKqlG6u8sInJkgF7EHe5NUl
awH8ui8MGK2PDoie/OpKk/c4lkP36vUJcPzmKE+eyKDd5kqR+AKyJDNkrQKBgQC3
lImGV7XgPxSeVABCO/VjxSpwWJgQuv6iP20dX7FJhjQooEkqvFOVRiF0qfjqVvnv
Pbv2IHK5yj4uTwZwjS3d8xseV3siT1LoRMOSFSvdLUCJpQHBBT5AbWeBTP6E6ENE
8H6a5jOyxC/Ua8dfy/B6OYDA/a8LgpqYYdB998q3sQKBgF+iY2OBwwStewinh/r8
CgV1z7hszwMXosdcjtg0BRjDKUwRhqkzHGLSOoC6QLubWOaXOe/ieCkgPdMqLNRZ
VMYn8pUQz+Dzeaw1GZr6MIcaVt5sqbeflkS7lU1I0F9thGRbwTw3JyzLMv5qxbQC
gdsRSP6LhKCkCUdRbjrit/1g
-----END PRIVATE KEY-----




cert chain intermediate & root:
-----BEGIN CERTIFICATE-----
MIIEHjCCAwagAwIBAgICEAAwDQYJKoZIhvcNAQENBQAwgaQxCzAJBgNVBAYTAkRF
MRAwDgYDVQQIDAdCYXZhcmlhMQwwCgYDVQQHDANOQkcxCzAJBgNVBAoMAlNDMRIw
EAYDVQQLDAljb21wdXRlcnMxIDAeBgNVBAMMF2p3LXZjc2E3LnZkaS5zY2xhYnMu
bmV0MTIwMAYJKoZIhvcNAQkBFiNqdWxpYW4ud2VuZGxhbmRAc29lbGRuZXItY29u
c3VsdC5kZTAeFw0yMTAzMTcxNzA2MDJaFw0zMTAzMTUxNzA2MDJaMIGWMQswCQYD
VQQGEwJERTEQMA4GA1UECAwHQmF2YXJpYTELMAkGA1UECgwCU0MxEjAQBgNVBAsM
CWNvbXB1dGVyczEgMB4GA1UEAwwXanctdmNzYTcudmRpLnNjbGFicy5uZXQxMjAw
BgkqhkiG9w0BCQEWI2p1bGlhbi53ZW5kbGFuZEBzb2VsZG5lci1jb25zdWx0LmRl
MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA61/M+hIYYPe3GWkdmlvX
rhMHXMbyMT1ni0Soxto1gpKGA8Cr6evLqnkupyCzVnivSxuzcpPNyf57rOSTZkXy
C2XOCtEVzxG1r/VjfwECF4z9/BQSo7rfVybXEywtynEoXErv0Rq1D/QaPvrdRfbi
i7rngrsFytA/25+3+dasOPUZ2TJSYkk0/79+VVcbSFhJwue5gstHvcSeTc6ZQ/2a
cXXgjQZbhgUZxle1pUYcpcCv/KEe2CKc9snlDeHO5o9TZLayhjuuaoAWxTMFdfzN
OrW+YHumT+/ec/Lshc5edw4b6bhTmVAr6IAPFCcyPTDhpQkL/sUXJDt/k4rjU/5N
xQIDAQABo2YwZDAdBgNVHQ4EFgQUejwGwUzhOztN5ui5E2hYpnnHy0UwHwYDVR0j
BBgwFoAU5o7gOoYot7x7+DdI/Wta+BjFuOQwEgYDVR0TAQH/BAgwBgEB/wIBADAO
BgNVHQ8BAf8EBAMCAYYwDQYJKoZIhvcNAQENBQADggEBAAPmdIZvdXLZFS22c6VG
Ii1sW/QkeyReeODz2oE/09cmdoGFo2j/TOsY/UP8f8WCQyeqd6aBggA+b+lBa8Jt
+N1WA0+DwTzkkPlq+leIZg/HRr0QIEEJshr2jDlT5N38kiGAdm0e+I/aszgHHiCM
OtdnGtDPdos5A6z7mrwMI+gsHxVr08q13aGljTjgByEP/VtFAxAT+zxvdDiSREE1
lvp1OEnsXyv+dkAG8H1dzJqzKIrXmiN+8A5TRaYKhbRakiD4UOMsLiBRvACvmvIr
eE3Ci+nL2YIXln66VRFAC9DhGeRr2ypvqGgH5dBbK93+Jz4bW+9Vw3uVUH13Sggj
D6g=
-----END CERTIFICATE-----
-----BEGIN CERTIFICATE-----
MIIEKDCCAxCgAwIBAgIBADANBgkqhkiG9w0BAQ0FADCBpDELMAkGA1UEBhMCREUx
EDAOBgNVBAgMB0JhdmFyaWExDDAKBgNVBAcMA05CRzELMAkGA1UECgwCU0MxEjAQ
BgNVBAsMCWNvbXB1dGVyczEgMB4GA1UEAwwXanctdmNzYTcudmRpLnNjbGFicy5u
ZXQxMjAwBgkqhkiG9w0BCQEWI2p1bGlhbi53ZW5kbGFuZEBzb2VsZG5lci1jb25z
dWx0LmRlMB4XDTIxMDMxNzE2NDgyMloXDTMxMDMxNTE2NDgyMlowgaQxCzAJBgNV
BAYTAkRFMRAwDgYDVQQIDAdCYXZhcmlhMQwwCgYDVQQHDANOQkcxCzAJBgNVBAoM
AlNDMRIwEAYDVQQLDAljb21wdXRlcnMxIDAeBgNVBAMMF2p3LXZjc2E3LnZkaS5z
Y2xhYnMubmV0MTIwMAYJKoZIhvcNAQkBFiNqdWxpYW4ud2VuZGxhbmRAc29lbGRu
ZXItY29uc3VsdC5kZTCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAJPl
8+zoknp+AJAUW5W1ubVNJr2OOGhrLzGeRVggoOfLUviSwfVB5V/KEaLms8v0YvJR
E6eBvaLf6dJZwYuhGSeXoN3pT2Xxyez+q8LEFIWjxMC7kcbH5nEmOsGFr0y+yYES
74JzUhYYt0wSVFlVO7ER5Y/SbqEhkkPEK2WgmtTwsyxyfcvrO87JCCmNgtDayG7e
WGC1SWHtJKOuK9Mq8Vw0+Ed4ujDJ3gLLlt6R15NBdSumMdmahZBIqLw1+QpriAqT
jXSg4sJoeDb2o2qW9fRRugbOJoCm84kuvCYZUha8f1LeqvvVQtz33/YPW7z6bWN6
1b8sQr7bdfwLRBDMLE0CAwEAAaNjMGEwHQYDVR0OBBYEFOaO4DqGKLe8e/g3SP1r
WvgYxbjkMB8GA1UdIwQYMBaAFOaO4DqGKLe8e/g3SP1rWvgYxbjkMA8GA1UdEwEB
/wQFMAMBAf8wDgYDVR0PAQH/BAQDAgGGMA0GCSqGSIb3DQEBDQUAA4IBAQAzyoeQ
4t/KvfU5HEc0LyenTpQVft0UP0kmN/LuPFNohtxKrVjsHVhs32b3U8xu1htrQvkX
iOD5mOjwI+8/ysz2A6Cf791/L/Fi04NT1SmxsW4w83q5kBQ8TfGWxniA9yPvclzA
/P1k/UUTUFZFf3k4nCdHLgPjCKSD5j2RO0r2k5bvUPFmtVhpf0NHe5lLp0sMlfDI
CMcV1CKG2cU8d2HC79eAhGnLstN7Di8gnb54ByMbT3Xr8qeiAg146aKUA7yM2Cm/
g9W+xMa+BAijWfMc8jGecX/i0pwxUicc1lneRGQep/iF5WSjHcHuGpMyj28BIoAg
5hk/mDLivp7ow3bc
-----END CERTIFICATE-----



server cert:
-----BEGIN CERTIFICATE-----
MIIFNzCCBB+gAwIBAgICEjQwDQYJKoZIhvcNAQENBQAwgZYxCzAJBgNVBAYTAkRF
MRAwDgYDVQQIDAdCYXZhcmlhMQswCQYDVQQKDAJTQzESMBAGA1UECwwJY29tcHV0
ZXJzMSAwHgYDVQQDDBdqdy12Y3NhNy52ZGkuc2NsYWJzLm5ldDEyMDAGCSqGSIb3
DQEJARYjanVsaWFuLndlbmRsYW5kQHNvZWxkbmVyLWNvbnN1bHQuZGUwHhcNMjEw
MzE3MTc0NDIxWhcNMzEwNjIzMTc0NDIxWjBcMQswCQYDVQQGEwJERTEQMA4GA1UE
CAwHQmF2YXJpYTEMMAoGA1UEBwwDTkJHMQswCQYDVQQKDAJTQzEgMB4GA1UEAwwX
anctdmNzYTcudmRpLnNjbGFicy5uZXQwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAw
ggEKAoIBAQCw7fWKtM6W4/y0DcbrFRoBK9dQCIobFK8VIL0g2WsnTEqbnHT88q5w
ewuD/nPWjFjxAA2m3rwsqKj05Kixi823Dv/KcHkXk6EkfMfHOZ7xjcXF43WJklnZ
lNS1wBA2/e110qhqPkylfYuVBJ49PrYEFXJpQjEltSN87soVHeFfWc/nwqyPaYcx
XC+Rr7xVA9cwG+C7ir/6JSUb2pkIajbCZIs2/8koEuM5lj4DTurN03hv4NfT681q
PYuvOitVYNPcU6L605+vAfqcmx+L/wA+bjBieDIjky/I0EJQd9t74BH3T8U2dm4A
MKCrNTib1j+KwRI2o3WMSQ7+KOHteCpVAgMBAAGjggHGMIIBwjAJBgNVHRMEAjAA
MBEGCWCGSAGG+EIBAQQEAwIGQDAzBglghkgBhvhCAQ0EJhYkT3BlblNTTCBHZW5l
cmF0ZWQgU2VydmVyIENlcnRpZmljYXRlMB0GA1UdDgQWBBQuSag3/A0aGlpwR+PL
whnoOk0EXTCB0gYDVR0jBIHKMIHHgBR6PAbBTOE7O03m6LkTaFimecfLRaGBqqSB
pzCBpDELMAkGA1UEBhMCREUxEDAOBgNVBAgMB0JhdmFyaWExDDAKBgNVBAcMA05C
RzELMAkGA1UECgwCU0MxEjAQBgNVBAsMCWNvbXB1dGVyczEgMB4GA1UEAwwXanct
dmNzYTcudmRpLnNjbGFicy5uZXQxMjAwBgkqhkiG9w0BCQEWI2p1bGlhbi53ZW5k
bGFuZEBzb2VsZG5lci1jb25zdWx0LmRlggIQADAOBgNVHQ8BAf8EBAMCBaAwEwYD
VR0lBAwwCgYIKwYBBQUHAwEwVAYDVR0RBE0wS4IXanctdmNzYTcudmRpLnNjbGFi
cy5uZXSCF1tBbnkgdmFyaWF0aW9uIG9mIEZRRE5dghdbQW55IHZhcmlhdGlvbiBv
ZiBGUUROXTANBgkqhkiG9w0BAQ0FAAOCAQEAVWOcoDRMZA93gfEdEklOfO3Uhjfa
yXbndDNWPthhsOlKOtjuWg1nblf7ETlx2dXD6XsXjX2SDolY2zv9O4rY4rXFlxZl
GgcvXa7+azI8Yc+TpqouIclCQduG8PMpHfc0NMWCi8MkOIU4+yDH5HGL85zxmv3E
J4kPWnHeaagUvM5Itn70lPCVx5lFenwgEhbvkAuBmbAXYyYGz+Migz70j0bhqO0T
G2yIllcJ+uQ0SX/gC5orlAXJVqjAG0XfjJ+ylaHpnBFWRjv9YedBMZMr+iKhDWty
gqfu7LB5wm9z+uwDTmNW5X+q8BXlbkPhxHlwN+WZh7szRRdAubP+baKQKg==
-----END CERTIFICATE-----
