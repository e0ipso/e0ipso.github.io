---
title: 'How to use HTTPS in your local environment'
category: drupal
categories:
  - web-development
  - drupal
tags:
  - Drupal Development
image: /assets/images/2020/encryption.jpg
---
I have several local development environments in my machine. I would like to use HTTPS on them
without much hustle. That is why I decided to create my own custom Certificate Authority to simplify
the process.

<!-- more -->

First I want to disclose that I am not an expert in this matter. I did some research and stapled
together several articles that I found. My main source of inspiration was [this article](https://deliciousbrains.com/ssl-certificate-authority-for-local-https-development/).

Please note that I am fully aware there are local development toolkits that may handle this for me.
I have never found a need to virtualize and or containerize my local development environment. I like
running things in my metal because they run faster, I have more control over them, and debugging
them is simpler. If I ever need to work on several sites in parallel requiring conflicting setups
in my local, I will likely jump into that wagon. However, this article is not about that.

I did the whole process in three steps:
  1. I created the Certificate Authority (CA). This is a one time setup.
  1. I installed the CA into Linux and Firefox. This is also a one time setup.
  1. I generated one certificate per site, signed by the CA, and added it to nginx. You need to do
  this once per site.

## Creating a custom Certificate Authority
Since I have a directory in my local with all my sites I decided to put all
the files in a directory there: `/home/e0ipso/Sites/certs`. Please change the path to the one you use.
Run the commands in that directory. 

To create the CA you need to type:

```
openssl genrsa -des3 -out myCA.key 2048
```

During the execution of that command you will need to provide a passphrase. I recommend using your
password manager to generate and store such password. You will need the password in the last step.

After you generate the key, generate the root certificate.
```
openssl req -x509 -new -nodes -key myCA.key -sha256 -days 1825 -out myCA.pem
```

Openssl will prompt you with questions. You can leave the questions on their defaults, provide silly
answers, or give it truthful values. It doesn't matter.

## Making your system accept the new authority
If you use Linux you need to install the ca-certificate on the system. This is useful if you are not
using Firefox to make the HTTP requests. Think of curl, [Insomnia](https://insomnia.rest) (which is
amazing), a PHP program, etc. I learned how to do this in
[this post](https://www.techrepublic.com/article/how-to-install-ca-certificates-in-ubuntu-server/).

```
# First transform the .pem into a .crt.
openssl crl2pkcs7 -nocrl -certfile myCA.pem | openssl pkcs7 -print_certs -out myCA.crt
# Copy the ca-certificate where the OS expects it.
sudo cp myCA.crt /usr/local/share/ca-certificates
# Update the certificates to pick it up.
sudo update-ca-certificates
```

In addition to that you will need to install it into Firefox. Yes, even if you installed it system
wide.

![Firefox preferences pane](/assets/images/firefox-ca-cert.png)

In the firefox preferences access the _Privacy and Security_ section. After that under the
_Certificates_ heading select _View Certificates..._. This will open a dialog with an _Authorities_
tab. There you will be able to import `myCA.pem`.

## Creating the site certificate
This step is necessary every time you need to enable SSL for a new site in your local environment.
This is why I wanted to simplify the process as much as possible, because next time I need to do
this I will not remember how to do it. This is the shell script (you can also
[download it](/assets/documents/generate-certificate.sh)).

```
#!/bin/sh

if [ "$#" -ne 1 ]
then
  echo "Usage: Must supply a domain"
  exit 1
fi

DOMAIN=$1

openssl genrsa -out ./certs/$DOMAIN.key 2048
openssl req -new -key ./certs/$DOMAIN.key -out ./certs/$DOMAIN.csr

cat > ./certs/$DOMAIN.ext << EOF
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
subjectAltName = @alt_names
[alt_names]
DNS.1 = $DOMAIN
EOF

openssl x509 -req -in ./certs/$DOMAIN.csr -CA ./certs/myCA.pem -CAkey ./certs/myCA.key -CAcreateserial \
-out ./certs/$DOMAIN.crt -days 825 -sha256 -extfile ./certs/$DOMAIN.ext
```

I have this script in `/home/e0ipso/Sites/generate-certificate.sh`, one level above
`/home/e0ipso/Sites/certs/myCA.key`. Then I can run it with the domain name I want the certificate
for:

```
./generate-certificate.sh local.contrib.com
```

Once again this will prompt you with some questions, you can skip them if you like. At some point
it will require the password we saved in our password manager earlier (hopefully not a napkin). This
will generate the `local.contrib.com.key` and `local.contrib.com.crt` files. The last step is to add
it you your local webserver.

I use nginx, so I add:

```nginx
server {
    # ...
    listen 443 ssl;
    ssl_certificate /home/e0ipso/Sites/certs/local.contrib.com.crt;
    ssl_certificate_key /home/e0ipso/Sites/certs/local.contrib.com.key;
    # ...
```

If you are curious, [this is the nginx configuration file](/assets/documents/local.contrib.com.conf)
I use for my local development in a Drupal project.

<small>Photo by <a href="https://unsplash.com/@maurosbicego?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Mauro Sbicego</a> on <a href="https://unsplash.com/s/photos/encryption?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Unsplash</a></small>
