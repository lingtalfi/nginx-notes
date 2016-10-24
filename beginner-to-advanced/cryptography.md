Cryptography
=============================
2016-10-24




Asymmetric key encryption
===========================
2016-10-24


- Document is encrypted with either the public key or the private key
- Only the "other" key (the public key if the private key was used and vice-versa) can decrypt the document



Key based authentication
```txt
server (public) -----> 2+2=? ---------> zeal (private)
server (public) <----- 4 	 <--------- zeal (private)
```



Https internal working
===========================
2016-10-24


Https makes use of both symmetrical and assymetrical encryption.

Symmetrical encryption is used for the communication as its encryption/decryption algorithm is much faster.

Asymmetrical encryption is used to safely transmit the symmetric key from the client (browser) to the website.



```txt
browser <----------------------------------	 send the public key 	<--------- https website (private+public)

browser (public) 					------>  send the symmetric key ---------> https website (private+public)
generates a symmetric key (faster)
and encodes it with the public key


```



Generate a certificate
-------------------------

The commands below generates:

- a cert.pem file: certificate which contains the public key, some info (your organization name, ...) and the host name (example.com) associated with the website
- a key.pem file: the private key



```bash
cd
mkdir certificates
cd certificates
openssl req -x509 -newkey rsa:2048 -keyout key.pem -out cert.pem -days 365 -nodes

mv cert.pem final.pem
cp * /etc/ssl/certs
```


In nginx, open the ssl.conf file (quickstart).

```nginx
server {
	listen 443;
	server_name _;

	ssl on;
	ssl_certificate etc/ssl/certs/final.pem;
	ssl_certificate_key /etc/ssl/certs/key.pem;
	
    ssl_session_cache    shared:SSL:1m;
    ssl_session_timeout  5m;

    ssl_ciphers  HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers  on;

    location / {
        root   /var/www/html;
        index  index.html index.htm;
    }
}
```

How to sign your certificate?
-------------------------------

Ideally, you need to sign the final.pem certificate with a public authority (like geotrust).

In the case of geotrust, you will send them your certificate, and they will eventually sign it and return
to you a zip file (zealvora_com.zip for instance).

If you unzip it, it will contains two files:

- zealvora_com.ca-bundle (the trusted chain bundle)
- zealvora_com.crt (the signed certificate)

You should concatenate them:

```bash
cat zealvora_com.crt zealvora_com.ca-bundle > zeal.crt
cp zeal.crt zeal.key /etc/ssl/certs
```

Update your nginx conf and you're done.

```nginx
server{
	#...
	ssl_certificate etc/ssl/certs/zeal.crt;
	ssl_certificate_key /etc/ssl/certs/zeal.key;
	#...
}
```



SSL Termination
===========================
2016-10-24



```txt

=====: ssl encryption
-----: normal http communication



# SSL Termination at Reverse Proxy
browser <========> Reverse proxy <------> Application server


# SSL Termination to Upstream
browser <========> Reverse proxy <=============> Application server

```

Example ssl conf of the reverse proxy for a SSL Termination at Reverse Proxy

```nginx
# https server
server {
	listen 443 ssl;
	server_name zealvora.com;

	ssl on;
	ssl_certificate etc/ssl/certs/final.pem;
	ssl_certificate_key /etc/ssl/certs/key.pem;
	
    ssl_session_cache    shared:SSL:1m;
    ssl_session_timeout  5m;

    ssl_ciphers  HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers  on;

    location / {
        proxy_pass http://192.168.10.50:80; # this is the ip of the upstream server, note the http protocol used here
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header Host $host;
    }
}
```
















