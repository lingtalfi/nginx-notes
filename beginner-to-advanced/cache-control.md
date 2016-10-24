Cache control
=============================
2016-10-23



Base concept
-----------------

The response from the upstream server can be cached either in 
the browser, or in the cache server.


```txt
Browser   <----->   Cache (server)  <----------> Upstream server
```



### Theory

- Cache-Control: no-store
	- do not store the response anywhere (browser or cache server)
- Cache-Control: no-cache
	- the response can be cached anywhere, however it must revalidate (is it still fresh) everytime with the origin server (before sending the response to the client)

- Cache-Control: max-age=0
	- maximum age in seconds of the cache
- Cache-Control: s-maxage=0
	- exclusively for the intermediate node: the cache server, same as max-age otherwise

- Cache-Control: must-revalidate
	- the cache must revalidate the document after it has expired
- Pragma: no-cache
	- same as Cache-Control: no-cache, but specifically for HTTP 1.0 client/browsers



```nginx
server {
	server_name example.com;

	location / {
		root /var/www/html;
	}

	location ~ \.(png) {
		root /var/www/html;
		expires 1h;
	}

	location ~ \.(txt) {
		root /var/www/html;
		expires -1;
	}
}
```


Here is the result of a request to logo.png:
```txt
$ curl -I http://192.168.33.20/logo.png
HTTP/1.1 200 OK
Server: nginx/1.1.19
Date: Sun, 23 Oct 2016 12:37:06 GMT
Content-Type: image/png
Content-Length: 0
Last-Modified: Sun, 23 Oct 2016 12:34:44 GMT
Connection: keep-alive
Expires: Sun, 23 Oct 2016 13:37:06 GMT
Cache-Control: max-age=3600
Accept-Ranges: bytes
```


Note that the Expires header is included only for HTTP/1.0 compatibility: it does the equivalent of the Cache-Control header in this case.
Modern browsers accepting Cache-Control header will ignore the "Expires" header.




Cache headers
=============
2016-10-23


```nginx
server {
	server_name example.com;

	location ~ \.(png) {
		root /var/www/html;

		# in the curl test below, the following lines are tested one by one (except for the two no-cache directives tested in one request)
		add_header Cache-Control max-age=120;
		add_header Cache-Control no-store;
		add_header Cache-Control s-maxage=200; # the intermediary cache will use this header over max-age in case of conflicts
		add_header Cache-Control no-cache; # after 120 seconds, the client should revalidate with the server
		add_header Pragma no-cache;
	}
}
```

```txt
# output with max-age=120
$ curl -I http://192.168.33.20/logo.png
HTTP/1.1 200 OK
Server: nginx/1.1.19
Date: Sun, 23 Oct 2016 13:45:56 GMT
Content-Type: image/png
Content-Length: 15752
Last-Modified: Sun, 23 Oct 2016 13:44:16 GMT
Connection: keep-alive
Cache-Control: max-age=120
Accept-Ranges: bytes


# output with no-store
$ curl -I http://192.168.33.20/logo.png
HTTP/1.1 200 OK
Server: nginx/1.1.19
Date: Sun, 23 Oct 2016 13:50:59 GMT
Content-Type: image/png
Content-Length: 15752
Last-Modified: Sun, 23 Oct 2016 13:44:16 GMT
Connection: keep-alive
Cache-Control: no-store
Accept-Ranges: bytes


# output with s-maxage=200
$ curl -I http://192.168.33.20/logo.png
HTTP/1.1 200 OK
Server: nginx/1.1.19
Date: Sun, 23 Oct 2016 13:54:36 GMT
Content-Type: image/png
Content-Length: 15752
Last-Modified: Sun, 23 Oct 2016 13:44:16 GMT
Connection: keep-alive
Cache-Control: s-maxage=200
Accept-Ranges: bytes


# output with no-cache and Pragma no-cache
$ curl -I http://192.168.33.20/logo.png
HTTP/1.1 200 OK
Server: nginx/1.1.19
Date: Sun, 23 Oct 2016 13:58:39 GMT
Content-Type: image/png
Content-Length: 15752
Last-Modified: Sun, 23 Oct 2016 13:44:16 GMT
Connection: keep-alive
Cache-Control: no-cache
Pragma: no-cache
Accept-Ranges: bytes
```


Related: https://github.com/lingtalfi/http-cache-notes

- Firefox: about:cache
- Chrome: chrome://cache



 



must-revalidate
=============================
2016-10-23

Indicates that cache(s)? should never serve a stale copy.

```nginx
server {
	server_name example.com;

	location ~ \.(png) {
		root /var/www/html;
		add_header Cache-Control must-revalidate;


		# add_header Cache-Control no-cache must-revalidate;
	}
}
```


public private
=============================
2016-10-23


```nginx
server {
	server_name example.com;

	location ~ \.(png) {
		root /var/www/html;

		 
		add_header Cache-Control private max-age=200; 
		add_header Cache-Control public s-maxage=500; 
	}
}
```



keep alive
=============================
2016-10-23


Optional in HTTP/1.0, default in HTTP/1.1.

Do not close the tcp connection after each http request.

With the old model, if you send 3 http requests, you initiate 3 tcp three-way handshakes.

With the new model, if you send 3 http requests, you can manage to send have only one tcp three-way handshake.


### Communication basic

```http
GET /pid1.png HTTP/1.1
Host: example.com
Connection: Keep-Alive

HTTP/1.1 200 OK
Content-Type: image/png
Connection: Keep-Alive
```



By default, nginx is configured to keep the connection alive for 75s.
If you set this value to 0, nginx will not keep connections at all (old model).


```nginx
http {
	keepalive_timeout 75s; 
	# keepalive_timeout 0; # uncomment this if you don't want persistent connections
}
```




