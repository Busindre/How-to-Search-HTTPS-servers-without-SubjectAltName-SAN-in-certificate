# How-to-Search-HTTPS-servers-without-SubjectAltName-SAN-in-certificate
Create SAN (SubjectAltName) certificate and locate servers that only implement CN (Common Name).

**Common name vs Subject Alternative Name**.

The common name can only contain up to one entry: either a wildcard or a single name. It’s not possible to specify a list of names covered by an SSL certificate in the common name field.

The SAN extension was introduced to solve this limitation and allow to issue multi-domain SSL certificates. The SAN extension can be used to integrate or replace the common name, and it supports the ability to specify different domains protected by a single SSL certificate.

As per current best practice, SAN is the primary source to check and CN should be checked only if SAN does not exist; all certificate subject domains should be listed in the SANs.


**Support for commonName matching in Certificates**.

RFC 2818 describes two methods to match a domain name against a certificate - using the available names within the subjectAlternativeName extension, or, in the absence of a SAN extension, falling back to the commonName. The fallback to the commonName was deprecated in RFC 2818 (published in 2000), but support still remains in a number of TLS clients, often incorrectly.

**Consensus & Standardization** (01/05/2017).

- Chrome: https://groups.google.com/a/chromium.org/forum/m/#!topic/security-dev/IGT2fLJrAeo (Removed)
- Firefox: https://bugzilla.mozilla.org/show_bug.cgi?id=1245280 (Opposed)
- Edge: No public signals
- Safari: No public signals
- Web Developers: No signals

Chromium removed support for matching common name in certificates and displays the following error.
```
Error:net::ERR_CERT_COMMON_NAME_INVALID
```

## How-to: Create SAN (SubjectAltName) certificate.

To avoid problems it is recommended to configure SSL certificates with SAN policies on private networks. We can use Nmap and OpenSSL if we want to identify all the certificates that do not have SAN and must be modified.

Create a file (san.conf) with the desired specifications.
```
[req]
distinguished_name = req_distinguished_name
req_extensions = v3_req
prompt = no
 
[req_distinguished_name]
C = XX
ST = XX
L = XX
O = XX
OU = XX
CN = XX
 
[v3_req]
keyUsage = keyEncipherment, dataEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @alt_names
 
[alt_names]
DNS.1 = dominio.com
DNS.2 = XX.dominio.com
DNS.3 = YY.dominio.com
DNS.4 = ZZ.dominio.com
DNS.5 = WW.dominio.com
```
Generate a Certificate Signing Request (file san.csr) to order an SSL certificate.
```bash
openssl req -sha512 -new -key XX.key -out san.csr -config san.conf
# openssl req -text -noout -verify -in san.csr
```

## How-to: Search HTTPS servers on a network without SubjectAltName (SAN) in certificate (nmap / openssl).
```bash
nmap -p 443 --open X.X.X.X/X | egrep -o '([0-9]{1,3}\.){3}[0-9]{1,3}' | xargs  -I {} sh -c "(echo | openssl s_client -showcerts -connect {}:443 2>/dev/null | openssl x509 -inform pem -noout -text | grep -q "DNS:") 2>/dev/null || (echo {} )"
```
