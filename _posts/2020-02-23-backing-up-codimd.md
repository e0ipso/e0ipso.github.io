---
title: Backing up Files with CodiMD 
categories:
 - web-development
tags:
 - Open Source
---
I recently started renting a VPS server to host some of the ethical alternatives to the typical host of surveillance
services.

One service that I have enjoyed in the past is Dropbox Paper. However this "free" (as in beer, not in freedom) service
preys on your personal data, just like its big brother Dropbox. To me this is not acceptable, so I decided to give
[CodiMD](https://github.com/codimd/server#readme) a try. CodiMD is an evolution of HackMD, a free software alternative
inspired in its free software cousin [Etherpad](https://etherpad.org/). CodiMD is licensed under [Affero General Public
License](https://www.gnu.org/licenses/agpl-3.0.html)¹.
<!-- more -->
CodiMD has a [demo site](https://demo.codimd.org/features?both) where you can go to play and test all of its (extensive)
features.

![CodiMD features](/assets/images/codimd-features.png)

I am still discovering the intricacies of this software, and I am already in awe of the quality of the product. The
feature that I am missing is the ability to download my MD documents, so I can archive them in my home NAS. For this
reason I have created this small nodejs script, that I want to share with you as AGPL as well. This code is a bit messy
but I wanted something quick that didn't have any NPM dependency to run, only nodejs core.

```js
#!/usr/bin/node

const { request } = require('https');
const querystring = require('querystring');
const { writeFileSync } = require('fs');

const host = process.argv[2];
const email = process.argv[3];
const password = process.argv[4];

if (!host || !email || !password) {
  console.error(`Usage ${process.argv[1]} <host> <username> <password>`);
  process.exit(1);
}

const baseOptions = { host, port: 443 };

const postData = querystring.stringify({ email, password });

const req = request({
    ...baseOptions,
    path: '/login',
    method: 'POST',
    headers: {
      'Content-Type': 'application/x-www-form-urlencoded',
      'Content-Length': Buffer.byteLength(postData)
    }
  }, (res) => {
  const cookie = res.headers['set-cookie'][0];
  baseOptions.headers = { Cookie: cookie };
  const req2 = request({
    ...baseOptions,
    path: '/history',
  }, res2 => {
    let body = '';
    res2.on('data', chunk => { body += chunk });
    res2.on('end', () => {
      let parsed = {};
      try {
        parsed = JSON.parse(body);
      }
      catch (e) {
        console.error('Invalid credentials');
        process.exit(2);
      }
      const reqs = parsed.history.map(({ text, id }) => {
        const fileName = `${text.toLowerCase().replace(/\s+/g, '-')}.md`;
        return request({
          ...baseOptions,
          path: `/${id}/download`,
        }, rsp => {
          let bd = '';
          rsp.on('data', chunk => { bd += chunk });
          rsp.on('end', () => {
            writeFileSync(fileName, bd);
            console.log(`Written file "${fileName}".`);
          });
        });
      });
      reqs.map(r => r.end());
    });
  });
  req2.end();
  res.setEncoding('utf8');
});

req.on('error', (e) => {
  console.error(`problem with request: ${e.message}`);
});

// Write data to request body
req.write(postData);
req.end();
```  

Executing this script will download the files to your local:

```
node backup-notes.js text.mateuaguilo.com email@example.org fooBarIsPassword
```

¹ AGPL is considered one of the most freedom friendly licenses. Companies like Google
[prohibit their use](https://opensource.google/docs/using/agpl-policy/) because of its nature.
