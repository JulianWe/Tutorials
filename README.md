# Tutorials

https://dadhacks.org/2017/12/27/building-a-root-ca-and-an-intermediate-ca-using-openssl-and-debian-stretch/



# mkdir /root/ca
# cd /root/ca
# mkdir newcerts certs crl private requests


openssl genrsa -aes256 -out private/ca.ansible-vcsa-1.qa-de-5.cloud.sap.key.pem 2048


openssl req -config openssl_root.cnf -new -x509 -sha512 -extensions v3_ca -key private/ca.ansible-vcsa-1.qa-de-5.cloud.sap.key.pem -out certs/ca.ansible-vcsa-1.qa-de-5.cloud.sap.crt.pem -days 3650 -set_serial 0


Creating signing request 
openssl req -config openssl_intrmediate.cnf -new -newkey rsa:2048 -keyout private/int.qa-de-5.cloud.sap.key.pem -out csr/int.ca.qa-de-5.cloud.sap.csr


Creating the intermediate certificate
openssl ca -config openssl_root.cnf -extensions v3_intermediate_ca -days 3650 -notext -md sha512 -in intermediate/csr/int.ca.qa-de-5.cloud.sap.csr -out intermediate/certs/int.qa-de-5.cloud.sap.crt.pem


Creating certificate chain
cat intermediate/certs/int.qa-de-5.cloud.sap.crt.pem certs/ca.ansible-vcsa-1.qa-de-5.cloud.sap.crt.pem > intermediate/certs/chain.qa-de-5.cloud.sap.crt.pem

create server cert signing request:
openssl req -out intermediate/csr/int.ca.qa-de-5.cloud.sap.csr.pem -newkey rsa:2048 -nodes -keyout intermediate/private/int.qa-de-5.cloud.sap.key.pem -config openssl_csr_san.cnf

Creating certificate:
openssl ca -config openssl_intrmediate.cnf -extensions server_cert -days 3750 -notext -md sha512 -in intermediate/csr/int.ca.qa-de-5.cloud.sap.csr.pem -out intermediate/certs/ansible-vcsa-1.qa-de-5.cloud.sap.crt.pem

