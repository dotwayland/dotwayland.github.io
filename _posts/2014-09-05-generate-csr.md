---
layout: post
title:  "Generating a Certificate Signing Request (CSR)"
date:   2014-09-05 00:00:01
categories: jekyll update
---

Create a directory for your domain underneath /etc/ssl and change into it

`derp@arkham:~# mkdir /etc/ssl/domain.tld && cd /etc/ssl/domain.tld`

Generate an RSA private key for your domain and the certificate signing request, I've specified a 4096 bit long modulus and a SHA-2 cert in this example

`derp@arkham:/etc/ssl/domain.tld# openssl req -nodes -newkey rsa:4096 -sha256 -keyout domain.tld.key -out domain.tld.csr`

```
The fields required in a certificate signing request along with an explanation and example are listed below:
Common name – The fully qualified domain name (FQDN) for your web server. This must be an exact match.
Organization – The exact legal name of your organization. Do not abbreviate your organization name.
Organization unit - Section of the organization, can be left empty if this does not apply to your case.
City/Locality - The city where your organization is legally located.
State/country/region - The state/county/region where your organization is legally located. Must not be abbreviated.
Country - The two-letter ISO abbreviation for your country.
Email address - The email address used to contact your organization.
```

Use restrictive permissions to protect the key file by making it readable only by its owner

`derp@arkham:/etc/ssl/domain.tld# chmod 400 domain.tld.key`

Make root the owner of the keyfile

`derp@arkham:/etc/ssl/domain.tld# chown root domain.tld.key`

Stream your newly created Base 64 representation of the Certificate Signing Request that you will use to obtain your SSL certificate.

`derp@arkham:/etc/ssl/domain.tld# cat domain.tld.csr`

```
-----BEGIN CERTIFICATE REQUEST-----
Base 64 encoded data
-----END CERTIFICATE REQUEST-----
```
[jekyll-gh]: https://github.com/mojombo/jekyll
[jekyll]:    http://jekyllrb.com
