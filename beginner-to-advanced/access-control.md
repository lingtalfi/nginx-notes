Access control
=============================
2016-10-23





White list
================
2016-10-23



```nginx

server {
	server_name example.com;

	location / {
		root /var/www/html;
	}

	location /admin {
		root /var/www/html;
		allow 172.18.10.5;
		deny all;
	}
}

```


Limit connection module
===========================
2016-10-23

Useful for sites involving big files download.


For instance, you have a website with a 100mbps bandwidth.

There is a video on the server.

10 users download it simultaneously, 2 of them have a 50mbps bandwidth, while the other 8 users have a 2mbps bandwidth.

The problem with this is that the 2 users with the 50mbps bandwidth will eat up the 100mbps and the other 8 users
won't have any bandwidth left, which results in a 503 error code, or a slow download at best.

To avoid this problem, we can limit the bandwidth to 10mbps so that all 10 users can download the file at the same time.




### test the download speed of the server

In the following example, the average bandwidth of the server is 920KB/s.

```bash
wget http://cachefly.cachefly.net/100mb.test
# ...
2016-10-23 15:49:03 (920 KB/s) - `100mb.test' saved [104857600/104857600]
```


```nginx
# configuration of server 192.168.33.20 (basic reverse proxy)


# this 
limit_conn_zone $binary_remote_addr zone=addr:10m;

server{
	listen 80;

	location / {
		root /var/www/html;
	}

	location /downloads {
		root /var/www/html;
		limit_rate_after 50m; # if defined first 50m at maximum speed, then limit_rate setting applies
		limit_rate 50k;

		# limit this ip to 1 connection max (if we allowed multiple connections, the same ip could eat 
		# more bandwidth than 50k)
		limit_conn addr 1; 
	}
}
```






Basic Authentication
===========================
2016-10-23

Password is encoded (not encrypted) in base64.


```txt
browser ------->   GET /admin HTTP/1.1 ------> server

browser <-------   HTTP/1.1 401 Authorization Required  <------ server
				   WWW-Authenticate: Basic realm="Family"

browser ------->   GET /admin HTTP/1.1 				------> server
 				   Authorization: Basic Ynelf5012z=

```


Install htpasswd if necessary.

```bash
which htpasswd
# not sure about the line below, do some research maybe?...
apt-get install apache2-dev 
```


```nginx
server {
	server_name example.com;

	location / {
		root /var/www/html;
	}

	location /admin {
		root /var/www/html;
		auth_basic " Basic Authentication ";
		auth_basic_user_file /etc/nginx/.htpasswd;  # contains user name and password, base 64 encoded
	}
}
```


Create the /etc/nginx/.htpasswd file

```bash
cd /etc/nginx
htpasswd -c .htpasswd admin
```




Digest Authentication (principle)
===========================
2016-10-23





```txt
browser ------->   GET /admin HTTP/1.1 ------> server

browser <-------   HTTP/1.1 401 Authorization Required  <------ server
				   WWW-Authenticate: Digest realm="Family"
				   nounce=66c453f4e3060fzef05403
				   qop=auth-int

browser ------->   GET /admin HTTP/1.1 				------> server
 				   Authorization: Digest
				   nounce=66c453f4e3060fzef05403
 				   Response: "E44D03Iamfser32"
```


The Response is a md5 hash.

h1(user+pass+realm) = md5

h2(uri+req.method) = md5

final md5 (the "Response") = md5(h1) + md5(h2) + nounce






GeoIP
===================
2016-10-23


- geoip_country
- geoip_city
- geoip_longitude
- geoip_org


ubuntu12
```bash
apt-get install libgeoip-dev -y
```

```nginx
http {
	geoip_country /usr/share/GeoIP/GeoIP.dat;
	map "$host:$geoip_country_code $deny_by_country  {
		~^example.com:(?!IN) 1; # block non indian user
		default 0;
	}

	server {
		server_name example.com;
		if ( $deny_by_country ) {
			return 403;
		}
		location / {
			root /var/www/html;
		}
	}
}
```

It uses iso3166 codes.
















