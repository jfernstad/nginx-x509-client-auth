# HTTP Client x509 auth

I'm abig fan of using x509 certificates for authentication. This is a simple proof of concept using nginx to require a correctly signed client certificate for HTTP authentication.

Create a server cert, self signed certificate with CN=hostname:  

```bash
openssl req -new -x509 -newkey rsa:2048 -keyout server.key -out server.crt -subj "/C=SE/O=None/OU=Yep/CN=realtest.local" -nodes
```

Create a CA cert, self signed cert to sign client certs:  

```bash
openssl req -new -x509 -newkey rsa:2048 -keyout ca.key -out ca.crt -subj "/C=SE/O=None/OU=Yep/CN=MIGHTY CA" -nodes
```

Create and sign client cert with CA:  

```bash
openssl req -nodes -new -keyout user.key -out user.csr -subj "/C=SE/O=None/OU=Yep/CN=bestclient" 
openssl x509 -req -days 365 -in user.csr -CA ca.crt -CAkey ca.key -set_serial 01 -out user.crt
```

Execute nginx server using docker:  

```bash
docker run -v `pwd`:/etc/ssl/private/ -v `pwd`/nginx.conf:/etc/nginx/nginx.conf -v `pwd`/index.html:/var/www/index.html -p 443:443 nginx:1.11-alpine 
```

Verify:  

```
# Works
curl -k --resolve realtest.local:443:0.0.0.0 -E user.crt --key user.key https://realtest.local:443 -vvvvv

# Does not work, certificate not signed by correct CA
curl -k --resolve realtest.local:443:0.0.0.0 -E server.crt --key server.key https://realtest.local:443 -vvvvv

# Does not work, no client certificate
curl -k --resolve realtest.local:443:0.0.0.0 server.key https://realtest.local:443 -vvvvv

```

