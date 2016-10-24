Web Application Firewall
=============================
2016-10-23




Modular architecture
==========================
2016-10-23


List of compiled modules

```bash
nginx -V
```




Compiling from source
==========================
2016-10-24


Either use (recommended) my software installation technique (melp install-programs) or read those lines below:



```bash
cd /usr/src
wget http://nginx.org/download/nginx-1.9.0.tar.gz
tar -xzvf nginx-1.9.0.tar.gz
cd nginx-1.9.0.tar.gz

# in order to compile from source, you need some more packages
# this is the list of what you need on centos:
yum -y install gcc gcc-c++ make zlib-devel pcre-devel openssl-devel


# below here is an example of how to compile nginx with just the gzip static module
./configure --help
./configure \
--prefix=/etc/nginx \
--sbin-path=/usr/sbin/nginx \
--conf-path=/etc/nginx.conf \
--error-log-path=/var/log/nginx/error.log \
--user=nginx \
--group=nginx \
--pid-path=/var/run/nginx.pid
--with-http_gzip_static_module


# the previous created a Makefile
make
make install

useradd nginx

# at this point the service nginx ... command does not work
# the line below basicall creates the necessary /etc/init.d/nginx service file
wget -0 /etc/init.d/nginx https://gist.githubusercontent.com/sairam/5892520/raw/94c24d0f2d69d035e7195564961a19e38535676b/etc-init.d-nginx
chmod +x /etc/init.d/nginx
service nginx status

nginx -V

curl 127.0.0.1
```


Application firewall
========================
2016-10-24

owasp top 10

- injection
- broken authentication and session management
- cross-site-scripting xss
- insecure direct object references
- security misconfiguration
- sensitive data exposure
- missing function level access control
- cross-site request forgery (csrf)
- using components with known vulnerabilities
- unvalidated redirects and forwards



Web application firewall
----------------------------

- naxsi nbs-system
- You need to compile nginx from the sources

```bash


cd /usr/src
wget http://nginx.org/download/nginx-1.9.5.tar.gz
tar -xzvf nginx-1.9.5.tar.gz

wget https://github.com/nbs-system/naxsi/archive/master.zip
unzip master.zip

# in order to compile from source, you need some more packages
# this is the list of what you need on centos:
yum -y install gcc make GeoIP GeoIP-devel pcre-devel openssl openssl-devel


cd nginx-1.9.5.tar.gz


./configure \
--prefix=/etc/nginx \
--sbin-path=/usr/sbin/nginx \
--conf-path=/etc/nginx.conf \

--add-module=../naxsi-master/naxsi_src

--error-log-path=/var/log/nginx/error.log \
--user=nginx \
--group=nginx \
--pid-path=/var/run/nginx.pid

--with-http_ssl_module
--with-http_realip_module
--with-http_addition_module
--with-http_sub_module

make
make install

# you may want to install the init.d for nginx, see notes above
service nginx status
```


Configuring the naxsi module
-------------------------------

```bash
cp /usr/src/naxsi-master/naxsi_config/naxsi_core.rules /etc/nginx
```

In your nginx conf:

```nginx
http {
	include /etc/nginx/naxsi_core.rules;

	server {
		listen 80;
		server_name example.com;

		location / {
			root /var/www/html;

			# naxsi rules
			LearningMode;
			SecRulesEnabled;
			#SecRulesDisabled;
			DeniedUrl "/RequestDenied.txt";

			## check & blocking rules
			CheckRule "$SQL >= 8" BLOCK;
			CheckRule "$RFI >= 8" BLOCK;
			CheckRule "$TRAVERSAL >= 4" BLOCK;
			CheckRule "$EVADE >= 4" BLOCK;
			CheckRule "$XSS >= 8" BLOCK;
		}
	}
}

```


































