Reverse proxy
=============================
2016-10-22




proxy_pass
================



Setup
-----------
Web browser  <--> nginx (reverse proxy 192.168.33.10) <--> backend application server (192.168.33.15)



Basic principle
------------------
```bash
location / {
	proxy_pass http://192.168.33.15;
}

location /admin {
	proxy_pass http://192.168.33.15;
}
```



Lab section
--------------

```bash
# reverse proxy at /usr/local/nginx/conf/nginx.conf, inside http context
server {
	server_name _;
	location / {
		proxy_pass http://192.168.33.15;
	}
}


# backend server
server {
    root "/var/www/html";      
}

```


check the logs
```bash
# reverse proxy (or us nnlog alias)
tail -f /usr/local/nginx/logs/access.log

# backend server (or us nnlog alias)
tail -f /var/log/nginx/access.log

```

Here is some sample logs that I collected, the fun thing is the 192.168.33.1 ip.
I believe it's the "nginx generated client sort of, or the client browser on the reverse proxy sort of"'s ip.
Anyway, ...

```txt
# log from reverse proxy server
192.168.33.1 - - [22/Oct/2016:19:54:19 +0000] "GET / HTTP/1.1" 304 0 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.11; rv:49.0) Gecko/20100101 Firefox/49.0"


# log from backend server
192.168.33.10 - - [22/Oct/2016:19:53:59 +0000] "GET / HTTP/1.0" 304 0 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.11; rv:49.0) Gecko/20100101 Firefox/49.0"
```





X-Real-IP
===================
2016-10-22


In the previous section, by analyzing the logs on the back end server, we see that the back end server thinks
that all requests come from 192.168.33.15 (the reverse proxy).

Accessing the ip of the real client would be useful though (stats, ...?).

We can add a X-Real-IP (or other) custom header in the configuration of the reverse proxy and that info
would reach the back end server. 

We also need to configure the back end server's logs accordingly.



```bash
# reverse proxy (192.168.33.10)
server {
	server_name _;
	location / {
		proxy_pass http://192.168.33.15;
		proxy_set_header X-Real-IP $remote_addr;
	}
}


# backend server
http {
	#...
	log_format main '$remote_addr - $remote_user [$time_local] "$request" '
					'$status $body_bytes_sent "$http_referer" '
					'"$http_user_agent" "$http_x_real_ip"';
	access_log /var/log/nginx/access.log main;				
}

```

Now, our backend server log looks like this (notice the client address 192.168.33.1 at the end of the line):

```txt
192.168.33.10 - - [22/Oct/2016:18:52:33 +0000] "GET / HTTP/1.0" 304 0 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.11; rv:49.0) Gecko/20100101 Firefox/49.0" "192.168.33.1"
```



Forwarding the Host header
=========================
2016-10-22


In the last section, we saw how the client's ip information was not transmitted by default by the reverse proxy to the back end server.

Similarly, the Host header is not transmitted.

This can be a problem for the case when the backend server serves multiple domains,
the issue being that the backend server could respond with an http response for another domain (not the one requested).

To remedy this problem, we can use the same solution as before, and make the reverse proxy add the Host header.

First, on the client machine (the one that executes the browser), configure the /etc/hosts file.

```txt
192.168.33.10 example.com
192.168.33.10 example.net
```

Then create the index files on the back end server.

```bash
mkdir -p /var/www/html/example.com
vim /var/www/html/example.com/index.html
mkdir -p /var/www/html/example.net
vim /var/www/html/example.net/index.html
```



Then, here is our configurations.

```nginx
# reverse proxy (192.168.33.10)
server {
	server_name _;
	location / {
		proxy_pass http://192.168.33.15;
		proxy_set_header Host $host;
	}
}



# back end (192.168.33.15)
server {
	server_name example.com;
	location / {
		root /var/www/html/example.com;
	}
}

server {
	server_name example.net;
	location / {
		root /var/www/html/example.net;
	}
}


```

Although this setup isn't perceptible via the access logs, it allows us to target the right landing page,
for either example.com or example.net.





