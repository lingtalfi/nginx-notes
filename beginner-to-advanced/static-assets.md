static assets
=============================
2016-10-23



### Setup

```txt
browser <----> reverse proxy <---> upstream server
```


Copy all static assets (from the upstream server) to the reverse proxy.



```nginx
# reverse proxy
server {
	server_name example.com;

	location / {
		proxy_pass http://192.168.33.20
		proxy_set_header Host $host;
	}
}

# upstream server
server {
	server_name example.com;

	location / {
		root /var/www/html;
	}
}
```

























