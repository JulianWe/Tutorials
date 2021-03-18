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

**creating the server certificate by signing the signing request with the intermediate ca**
```sh
openssl ca -config openssl_intermediate.cnf -extensions server_cert -days 3750 -notext -md sha512 -in intermediate/csr/jw-vcsa7.vdi.sclabs.net.csr.pem -out intermediate/certs/jw-vcsa7.vdi.sclabs.net.crt.pem
```

**creating a combined certificate for use with Apache server**
```sh
openssl pkcs12 -inkey private/ca.vdi.sclabs.net.key.pem -in certs/ca.vdi.sclabs.net.crt.pem -export -out jw-vcsa7.vdi.sclabs.net.combinedcertchain.pfx
openssl pkcs12 -in jw-vcsa7.vdi.sclabs.net.combinedcertchain.pfx -nodes -out jw-vcsa7.vdi.sclabs.net.combined.crt
```



```sh

Server certificate:
-----BEGIN CERTIFICATE-----
MIIFGDCCBACgAwIBAgICEjUwDQYJKoZIhvcNAQENBQAwgZYxCzAJBgNVBAYTAkRF
MRAwDgYDVQQIDAdCYXZhcmlhMQswCQYDVQQKDAJTQzESMBAGA1UECwwJY29tcHV0
ZXJzMSAwHgYDVQQDDBdqdy12Y3NhNy52ZGkuc2NsYWJzLm5ldDEyMDAGCSqGSIb3
DQEJARYjanVsaWFuLndlbmRsYW5kQHNvZWxkbmVyLWNvbnN1bHQuZGUwHhcNMjEw
MzE4MDgzNDMyWhcNMzEwNjI0MDgzNDMyWjBvMQswCQYDVQQGEwJERTEQMA4GA1UE
CAwHQmF2YXJpYTESMBAGA1UEBwwJTnVlcm5iZXJnMRgwFgYDVQQKDA9Tb2VsZG5l
ckNvbnN1bHQxIDAeBgNVBAMMF2p3LXZjc2E3LnZkaS5zY2xhYnMubmV0MIIBIjAN
BgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAq2aMyGAZGVfsxsQmRRZ/oYTv0Vab
J+WrdUc1aLPp7a5Jxnlf2arg74yAZDmVsyv3n7kbMIoKVXcSntj8K9lcEn4nEl1F
rmR3FaOTLyDDt3eflS7clPBqixVBrX57W7Yp985TskI3cliDZuXg8+c+tRA6D3hv
XdnFB0g7ewpmqfORm26ZjC05ppVZ07ySDeqw2grWRH+yPi8G+DQsC6Ofn7bGby1F
n6AgM1hv2j5FCCgHpK0K2mX7emZTaxLnYzsAoKmZTz4VvRulcCqIqJ5dukKZIREr
3iHOS71oDYt/u8d38tBmJw+9nKcxiG6HwNf6UJt8moYWZb9c0CbmvnuA1wIDAQAB
o4IBlDCCAZAwCQYDVR0TBAIwADARBglghkgBhvhCAQEEBAMCBkAwMwYJYIZIAYb4
QgENBCYWJE9wZW5TU0wgR2VuZXJhdGVkIFNlcnZlciBDZXJ0aWZpY2F0ZTAdBgNV
HQ4EFgQUCl+kvW6wgI7uPVXK8ZJ4MfW7QvUwgdIGA1UdIwSByjCBx4AUejwGwUzh
OztN5ui5E2hYpnnHy0WhgaqkgacwgaQxCzAJBgNVBAYTAkRFMRAwDgYDVQQIDAdC
YXZhcmlhMQwwCgYDVQQHDANOQkcxCzAJBgNVBAoMAlNDMRIwEAYDVQQLDAljb21w
dXRlcnMxIDAeBgNVBAMMF2p3LXZjc2E3LnZkaS5zY2xhYnMubmV0MTIwMAYJKoZI
hvcNAQkBFiNqdWxpYW4ud2VuZGxhbmRAc29lbGRuZXItY29uc3VsdC5kZYICEAAw
DgYDVR0PAQH/BAQDAgWgMBMGA1UdJQQMMAoGCCsGAQUFBwMBMCIGA1UdEQQbMBmC
F2p3LXZjc2E3LnZkaS5zY2xhYnMubmV0MA0GCSqGSIb3DQEBDQUAA4IBAQCteOww
q+iIAuuViEBggIwv0LB3epjDak0hSAWd55YjBKe7kdy2x0rByuD2D89EGt1sYD3C
LgAaI9aeItuLhKF93tHls6ZHbajOWdjxvjbcifCvEpuwEe5bkf2ihGw4XEdb1HkO
spUdQmExxMWbEzfKwmBW56TvLBAuEeKTvBo7Z/dZgHeJeQujh/QaOf+g2YB0br/W
2tJrDqVkYRbLDDvhA2m+lPl6gNZ+za6Rx0tHMlHujsTqVz3bE5KFEa3fk1SFHWeo
xXw8oPkU0F36UN1UhjJ0jUNL+XqA43q5rUT03fEM2vdz5uaS2rH15Q61G0gkVT8N
iz87o+IzzgODWjEQ
-----END CERTIFICATE-----




Certificate chain intermediate & root:
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


Private Key:
-----BEGIN PRIVATE KEY-----
MIIEvQIBADANBgkqhkiG9w0BAQEFAASCBKcwggSjAgEAAoIBAQCrZozIYBkZV+zG
xCZFFn+hhO/RVpsn5at1RzVos+ntrknGeV/ZquDvjIBkOZWzK/efuRswigpVdxKe
2Pwr2VwSficSXUWuZHcVo5MvIMO3d5+VLtyU8GqLFUGtfntbtin3zlOyQjdyWINm
5eDz5z61EDoPeG9d2cUHSDt7Cmap85GbbpmMLTmmlVnTvJIN6rDaCtZEf7I+Lwb4
NCwLo5+ftsZvLUWfoCAzWG/aPkUIKAekrQraZft6ZlNrEudjOwCgqZlPPhW9G6Vw
Koionl26QpkhESveIc5LvWgNi3+7x3fy0GYnD72cpzGIbofA1/pQm3yahhZlv1zQ
Jua+e4DXAgMBAAECggEAfT4iAQi3Tl2BFnyduj4GZO/OjRjLpwubjcbKsAdHF/YS
0oQ+Fb9XPbNc3d92E8Y82ulXhNBZXLn1UT0chq39KUYlJrYhBJ1EpvsvwXAfkyBF
66yiYfKK57ZQl4Wkfg9N+1U4szjPay5iVf4DsjV3DLcetc87EUjfP8L4M6AWBHhT
53whN3PqRWm0ze4xNLRchEQZpOkwUyXkN2efOPh9e6lnTeShmykkSoYPPin9CBvn
V35ZHndFzmQai/Vq4YXlUNa3pww/aHx6HfeMsLOo09ixbSp8QWHp2xMFQCKpvNaZ
GBxGbIvJmxHuwzix/F7VgNqZrLiI76vsJdD9hBONgQKBgQDWY1TQryvh2iy9k5VB
8zb/+WkLTLaTxeLB8zGDD0AyNW/jqKkA+fdA3MBGLVelMrDL9x1FR4fUV2khodb+
ox8hMY/juPcF0B/ldEpjK4FQtoF6px9fknAK77FXqhdc/VEqd3cqdlByxA7NzDAS
6og7leNvHx49HEhKGsSlsY8qjQKBgQDMqzpTfjUUxr4Ay+X4aubrKi/fyi5ykX81
Nd82y9mZcUD33PKxnf1hwaBSXdGxyNg6+b2zHZmeQPVHFVvGwbrXh2m5BTFjS4Tu
SXHsTwJmL3Qcek76gWs5U1bnUNnIFO8hAvmdfi96iWpn5h1/llQY4fpesv2ASfPN
auZid33R8wKBgBJ+41RVqH2FqxJ35wqXhwkyZUOaTK4XBmchKgZajHlIbuy/IkV5
S0GHSfdD9inEY8hU+2t8rlU9bU5/feLeA9ODSRymWnlf6UCMddZ0bGWgOS9xt50x
LwVihHRBsl5NZHE7eUZqiqo8C+LpWMRpA3PQjJyLnLo89GegQ5Lf7LAJAoGBAMTl
VU9NczNxnwiVH8BE17IU+8mHb/e4EXDXSs4kfkonsiDB5pkJLOIGrH2Q1FL8rUjP
SbgvGcItK8oeuhQT+/OsygC9Bi5IULIM5hQ4Tk6QCFv9Lk3Ag666hjgyh9D8krBn
dEwXQQXZfQxHTMmZjX4CqCLCfy4T9v//f3PrEJgRAoGALJCPqSHrsKoMRwVKd7AF
LGItkCelfp/iGt9p0ja+SqobenRqhG7YQ69FaA4XgfnCAE+ICceBwnuPffSpjZgk
mK3hQ5ejZ6Q7/ieDWisIqzsapIWZBSbwLaNwYelMTlgJQRetMQbxEv7SrZVCtT8K
clIyyW52GP8N6qtDAKxNW1s=
-----END PRIVATE KEY-----
``` 
