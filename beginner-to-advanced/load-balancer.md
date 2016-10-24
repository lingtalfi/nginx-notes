Load balancer
=============================
2016-10-22






Load balancer
====================
2016-10-22



## Setup

We have a total of three boxes.


```txt
			Client (browser)  
					|
					|
		load balancer (192.168.33.10)	
		    |						\								
		    |						 \								
		    |						  \								
		    |						   \								
		server1 (192.168.33.15)		server2 (192.168.33.20)    

```


On the machine from which the browser is executed, the /etc/hosts look like this:
```txt
192.168.33.10 example.com
```

Below is the nginx configuration for the 3 machines.
Notice that "backend" is a keyword that you can customize: it represents the group of servers used by the load balancer.


```nginx
# configuration of the load balancer server (192.168.33.10)
upstream backend {
	server 192.168.33.15;
	# server 192.168.33.15 down; # use this line to emulate a down server
	server 192.168.33.20;
}


server {
	listen 80;
	server_name example.com;

	location / {
		proxy_pass http://backend;
	}
}


# configuration of server1 (192.168.33.15)
server {
	server_name _;
	root /var/www/html; # the index.html in this directory says server1
}

# configuration of server2 (192.168.33.20)
server {
	server_name _;
	root /var/www/html; # the index.html in this directory says server2
}
```





passive & active monitoring
==================================
2016-10-23


- nginx: passive monitoring

	- when nginx detects that server1 is down (after sending $max_fails requests that failed), it doesn't send subsequent requests to server1 for 10 (fail_timeout) seconds 
	- max_fails=3
	- fail_timeout=50


- nginx plus: active monitoring
	- use of the health_check directive





Passive monitoring
======================
2016-10-23


```nginx
# configuration of the load balancer server (192.168.33.10)
upstream backend {
	server 192.168.33.15 max_fails=3 fail_timeout=50;
	server 192.168.33.20;
}

server {
	listen 80;
	server_name example.com;

	location / {
		proxy_pass http://backend;
	}
}
```






health_check (Nginx+)
==================
2016-10-23


```nginx
# configuration of the load balancer server (192.168.33.10)
upstream backend {
	server 192.168.33.15;
	server 192.168.33.20;

	# Defines the name and size of the shared memory zone that keeps the group’s configuration and run-time state 
	# that are shared between worker processes.
	zone backend 64k; 
}


match server_test {
	status 200-399;
	body !~ maintenance; # if body does not contain the expression maintenance
	# body ~ "Welcome to Nginx";
}


server {
	listen 80;
	server_name example.com;

	location / {
		proxy_pass http://backend;
		health_check;
		# health_check interval=10 fails=3 passes=2 uri=/text.txt match=server_test;
	}
}
```


reverse proxy sends get requests (to / by default) every 5 seconds (by default), and check whether or not each server in the backend group is down.

Options:

- interval: 10 -- the load balancer checks its server every 10 seconds (instead of the default 5 seconds)
- fails: 3 -- if a server fails to respond to a get request 3 consecutive times, it's considered down
- passes: 2 -- a server must respond positively to 2 consecutives get request to be considered not down
- uri=/text.txt -- set the target of the request (default is /)
- match=server_test -- use the settings defined in the match server_test block to decide whether or not the server is down





shared_memory
===================
2016-10-23


Sharing memory allow the different servers in the back end group to share which servers are down (by default,
this information is confined to a worker process, which is less efficient).



```nginx
# configuration of the load balancer server (192.168.33.10)
upstream backend {
	server 192.168.33.15;
	server 192.168.33.20;
	zone backend 64k; # this is where the shared memory zone is defined
}



server {
	listen 80;
	server_name example.com;

	location / {
		proxy_pass http://backend;
		health_check;
	}
}
```


server weights
======================
2016-10-23


In some cases, one server of the group is better than the others (for instance it has more ram).
Instead of the round robin equal default distribution, you can add weights on servers.


With the configuration below, if we make 4 requests to the load balancer, the server2 is called 3 times,
while server1 is called only once.



```nginx
# configuration of the load balancer server (192.168.33.10)
upstream backend {
	server 192.168.33.15;
	server 192.168.33.20 weight=3;
}

server {
	listen 80;
	server_name example.com;

	location / {
		proxy_pass http://backend;
	}
}
```



least connect
======================
2016-10-23


It tells the balancer to forward the request to the server with the least open connections.

This might be useful in cases where a server is handling a request that takes a long time to execute (20s for instance).



```nginx
# configuration of the load balancer server (192.168.33.10)
upstream backend {
	least_conn;
	server 192.168.33.15;
	server 192.168.33.20;
}

server {
	listen 80;
	server_name example.com;

	location / {
		proxy_pass http://backend/test.php;
	}
}


# configuration of 192.168.33.15
server {
	server_name _;
	root /var/www/html;

	location ~ \.php {
	    include fastcgi_params;
	    include fastcgi.conf;
	    fastcgi_pass 127.0.0.1:9000;
	}
}

# configuration of 192.168.33.20
server {
	server_name _;
	root /var/www/html;

	location ~ \.php {
	    include fastcgi_params;
	    include fastcgi.conf;
	    fastcgi_pass 127.0.0.1:9000;
	}
}
```


In this example, I try to emulate the server's business with the php sleep function.
Here are the php scripts used for the test.

```php
<?php 
// 192.168.33.15
sleep(20);
echo "I'm server 15";
```

```php
<?php 
// 192.168.33.20
echo "I'm server 20";
```





Using ab to test (sending 20 requests, 10 concurrent max at time), 

```bash
ab -n 20 -c 10 http://example.com:80/
```

and observing logs of the servers in the load balancing group, we can see that the distribution is not equal.
Running this code live, we also observe that the requests from 192.168.33.15 come long (20s) after those from 192.168.33.20.
However, we don't observe a 20/1 ratio (20 times as many requests on 192.168.33.20 compared to 192.168.33.15, and I'm not sure why).




```txt
# logs from 192.168.33.15
192.168.33.10 - - [22/Oct/2016:23:18:58 +0000] "GET /test.php HTTP/1.0" 200 20 "-" "ApacheBench/2.3"
192.168.33.10 - - [22/Oct/2016:23:18:58 +0000] "GET /test.php HTTP/1.0" 200 20 "-" "ApacheBench/2.3" "-"
192.168.33.10 - - [22/Oct/2016:23:18:58 +0000] "GET /test.php HTTP/1.0" 200 20 "-" "ApacheBench/2.3"
192.168.33.10 - - [22/Oct/2016:23:18:58 +0000] "GET /test.php HTTP/1.0" 200 20 "-" "ApacheBench/2.3" "-"
192.168.33.10 - - [22/Oct/2016:23:18:58 +0000] "GET /test.php HTTP/1.0" 200 20 "-" "ApacheBench/2.3"
192.168.33.10 - - [22/Oct/2016:23:18:58 +0000] "GET /test.php HTTP/1.0" 200 20 "-" "ApacheBench/2.3" "-"
192.168.33.10 - - [22/Oct/2016:23:18:58 +0000] "GET /test.php HTTP/1.0" 200 20 "-" "ApacheBench/2.3"
192.168.33.10 - - [22/Oct/2016:23:18:58 +0000] "GET /test.php HTTP/1.0" 200 20 "-" "ApacheBench/2.3" "-"
192.168.33.10 - - [22/Oct/2016:23:18:59 +0000] "GET /test.php HTTP/1.0" 200 20 "-" "ApacheBench/2.3"
192.168.33.10 - - [22/Oct/2016:23:18:59 +0000] "GET /test.php HTTP/1.0" 200 20 "-" "ApacheBench/2.3" "-"


# logs from 192.168.33.20
192.168.33.10 - - [23/Oct/2016:11:34:45 +0000] "GET /test.php HTTP/1.0" 200 20 "-" "ApacheBench/2.3"
192.168.33.10 - - [23/Oct/2016:11:34:45 +0000] "GET /test.php HTTP/1.0" 200 20 "-" "ApacheBench/2.3"
192.168.33.10 - - [23/Oct/2016:11:34:45 +0000] "GET /test.php HTTP/1.0" 200 20 "-" "ApacheBench/2.3"
192.168.33.10 - - [23/Oct/2016:11:34:45 +0000] "GET /test.php HTTP/1.0" 200 20 "-" "ApacheBench/2.3"
192.168.33.10 - - [23/Oct/2016:11:34:45 +0000] "GET /test.php HTTP/1.0" 200 20 "-" "ApacheBench/2.3"
192.168.33.10 - - [23/Oct/2016:11:34:45 +0000] "GET /test.php HTTP/1.0" 200 20 "-" "ApacheBench/2.3"
192.168.33.10 - - [23/Oct/2016:11:34:45 +0000] "GET /test.php HTTP/1.0" 200 20 "-" "ApacheBench/2.3"
192.168.33.10 - - [23/Oct/2016:11:34:45 +0000] "GET /test.php HTTP/1.0" 200 20 "-" "ApacheBench/2.3"
192.168.33.10 - - [23/Oct/2016:11:34:45 +0000] "GET /test.php HTTP/1.0" 200 20 "-" "ApacheBench/2.3"
192.168.33.10 - - [23/Oct/2016:11:34:45 +0000] "GET /test.php HTTP/1.0" 200 20 "-" "ApacheBench/2.3"
192.168.33.10 - - [23/Oct/2016:11:34:45 +0000] "GET /test.php HTTP/1.0" 200 20 "-" "ApacheBench/2.3"
192.168.33.10 - - [23/Oct/2016:11:34:45 +0000] "GET /test.php HTTP/1.0" 200 20 "-" "ApacheBench/2.3"
192.168.33.10 - - [23/Oct/2016:11:34:45 +0000] "GET /test.php HTTP/1.0" 200 20 "-" "ApacheBench/2.3"
192.168.33.10 - - [23/Oct/2016:11:34:45 +0000] "GET /test.php HTTP/1.0" 200 20 "-" "ApacheBench/2.3"
192.168.33.10 - - [23/Oct/2016:11:34:45 +0000] "GET /test.php HTTP/1.0" 200 20 "-" "ApacheBench/2.3"
```









