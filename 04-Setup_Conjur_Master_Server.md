# Objectives
Setup the CyberArk Conjur Master server in utilities host

## [OPTIONAL] Creating SSL Certificates for Conjur

Conjur uses certificates for communication between the Master, Standby, and follower nodes in Conjur cluster.
To understand Conjur certificate architecture, read: https://docs.cyberark.com/Product-Doc/OnlineHelp/AAM-DAP/Latest/en/Content/Deployment/HighAvailability/certificate-architecture.htm

Below guide is provided to generate a self-signed CA, and use the self-signed CA to sign the Conjur Master and followers certificates.
You may also proceed with the tasks using the sample certificates provided ![here](https://github.com/rajnishgargcloudrepository/conjur-aks/raw/main/task04/conjur-certificates.tgz)

1.0 Generate a self-signed certificate authority
- Generate private key of the self-signed certificate authority
```console
azureuser@VM-ConjurDemoAKS:~$ openssl genrsa -out ConjurDemoCA.key 2048
Generating RSA private key, 2048 bit long modulus (2 primes)
.....................................................+++++
............................+++++
e is 65537 (0x010001)
```
- Generate certificate of the self-signed certificate authority
```console
azureuser@VM-ConjurDemoAKS:~$ openssl req -x509 -new -nodes -key ConjurDemoCA.key -days 365 -sha256 -out ConjurDemoCA.pem
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:SG
State or Province Name (full name) [Some-State]:Singapore
Locality Name (eg, city) []:
Organization Name (eg, company) [Internet Widgits Pty Ltd]:CyberArk Software Pte. Ltd.
Organizational Unit Name (eg, section) []:
Common Name (e.g. server FQDN or YOUR name) []:Conjur Demo Certificate Authority
Email Address []:joe.tan@cyberark.com
```
2.0 Generate certificate for Conjur Master
- Generate private key of the Conjur Master certificate
```console
azureuser@VM-ConjurDemoAKS:~$ openssl genrsa -out master.conjur.demo.key 2048
Generating RSA private key, 2048 bit long modulus (2 primes)
............+++++
..........................................................................+++++
e is 65537 (0x010001)
```
- Create certificate signing request for the Conjur Master certificate
```console
azureuser@VM-ConjurDemoAKS:~$ openssl req -new -key master.conjur.demo.key -out master.conjur.demo.csr
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:SG
State or Province Name (full name) [Some-State]:Singapore
Locality Name (eg, city) []:
Organization Name (eg, company) [Internet Widgits Pty Ltd]:CyberArk Software Pte. Ltd.
Organizational Unit Name (eg, section) []:
Common Name (e.g. server FQDN or YOUR name) []:master.conjur.demo
Email Address []:joe.tan@cyberark.com

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:
```
- Generate certificate of the Conjur Master certificate
```console
azureuser@VM-ConjurDemoAKS:~$ openssl x509 -req -in master.conjur.demo.csr -CA ConjurDemoCA.pem -CAkey ConjurDemoCA.key -CAcreateserial -days 365 -sha256 -out master.conjur.demo.pem
Signature ok
subject=C = SG, ST = Singapore, O = CyberArk Software Pte. Ltd., CN = master.conjur.demo, emailAddress = joe.tan@cyberark.com
Getting CA Private Key
```
3.0 Generate certificate for Conjur Followers
- Generate private key of the Conjur Followers certificate
```console
azureuser@VM-ConjurDemoAKS:~$ openssl genrsa -out followers.default.svc.cluster.local.key 2048
Generating RSA private key, 2048 bit long modulus (2 primes)
.........................................................................+++++
...................................................................................+++++
e is 65537 (0x010001)
```
- Create certificate signing request for the Conjur Followers certificate
```console
azureuser@VM-ConjurDemoAKS:~$ openssl req -new -key followers.default.svc.cluster.local.key -out followers.default.svc.cluster.local.csr
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:SG
State or Province Name (full name) [Some-State]:Singapore
Locality Name (eg, city) []:
Organization Name (eg, company) [Internet Widgits Pty Ltd]:CyberArk Software Pte. Ltd.
Organizational Unit Name (eg, section) []:
Common Name (e.g. server FQDN or YOUR name) []:followers.default.svc.cluster.local
Email Address []:joe.tan@cyberark.com

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:
```
- Generate certificate of the Conjur Followers certificate
```console
azureuser@VM-ConjurDemoAKS:~$ openssl x509 -req -in followers.default.svc.cluster.local.csr -CA ConjurDemoCA.pem -CAkey ConjurDemoCA.key -CAcreateserial -days 365 -sha256 -out followers.default.svc.cluster.local.pem
Signature ok
subject=C = SG, ST = Singapore, O = CyberArk Software Pte. Ltd., CN = followers.default.svc.cluster.local, emailAddress = joe.tan@cyberark.com
Getting CA Private Key
```
