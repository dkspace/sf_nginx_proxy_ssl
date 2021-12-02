# sf_nginx_proxy_ssl

This is an exercise for DevOps training from SkillFactory.ru - DEVOPS

To configure access to Artifactory over ssl via nginx proxy

## [Generate root CA key and certificate](https://www.ibm.com/docs/en/runbook-automation?topic=certificate-generate-root-ca-key) 


```bash

# It`s possible case when ssl apply self signed certificate so root CA key and Certificate are unnecesary 
# we can generate self sined sertificate like below 
#openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/pki/nginx/private/nginx-01.key -out /etc/pki/nginx/nginx-01.crt -config req.conf

mkdir SAN
cd SAN

#For trusted chain with root CA key and Certificates:
>openssl genrsa -out rootCAKey.pem 2048
Generating RSA private key, 2048 bit long modulus
.................................+++
................................................................+++
e is 65537 (0x10001)

>openssl req -x509 -sha256 -new -nodes -key rootCAKey.pem -days 3650 -out rootCACert.pem 
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:RU
State or Province Name (full name) []:NW
Locality Name (eg, city) [Default City]:SPb
Organization Name (eg, company) [Default Company Ltd]:home
Organizational Unit Name (eg, section) []:home
Common Name (eg, your name or your server's hostname) []:myartifactory.net
Email Address []:dkspace@mail.ru


# Configuration files should be prepared in advance
>cat req.conf 
[req]
distinguished_name = req_distinguished_name
x509_extensions = v3_req
prompt = no
[req_distinguished_name]
C = RU
ST = NW
L = SPb
O = home
OU = home
CN = myartifactory.net
[v3_req]
keyUsage = keyEncipherment, dataEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @alt_names
[alt_names]
DNS.1 = virtualpc1.myartifactory.net


>cat v3.ext 
subjectAltName = DNS:virtualpc1.myartifactory.net

```

## [Generate  server key and certificate request](https://www.ibm.com/docs/en/runbook-automation?topic=certificate-generate-rba-server-key-request)

```bash
>openssl genrsa -out nginx-01.key 2048
>openssl req -new -key nginx-01.key -sha256 -out nginx-01.csr -config req2.conf 
```

## [Generate the server certificate](https://www.ibm.com/docs/en/runbook-automation?topic=certificate-generate-rba-server)

```bash
>openssl x509 -req -sha256 -in nginx-01.csr -CA rootCACert.pem -CAkey rootCAKey.pem -CAcreateserial -out nginx-01.crt -days 365 -extfile v3.ext
Signature ok
subject=/C=RU/ST=NW/L=SPb/O=home/OU=home/CN=myartifactory.net
Getting CA Private Key

#check
>openssl x509 -in nginx-01.crt -text
X509v3 extensions:
    X509v3 Subject Alternative Name: 
	DNS:virtualpc1.myartifactory.net
	    Signature Algorithm: sha256WithRSAEncryption

```

















