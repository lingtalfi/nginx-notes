Nginx random notes
=====================
2016-10-17


Some random notes related to nginx.



About gzip
------------------

gzip_comp_level efficiency?

jquery.min.js
- raw: 84.9 KB
- gzip level 1: 34.6 KB
- gzip level 2: 33.3 KB
- gzip level 3: 32.6 KB
- gzip level 4: 30.8 KB
- gzip level 5: 29.9 KB
- gzip level 6: 29.7 KB
- gzip level 7: 29.7 KB
- gzip level 8: 29.7 KB
- gzip level 9: 29.7 KB




Understand fastcgi_cache_path sub directories system
----------------------------------------------------------------
See source 1.



Forward the client IP address
--------------------------------

Say you have a nginx reverse proxy (mouche), and a backend (http://192.168.33.10:80).
In mouche's conf:

```nginx
 server {
    server_name mouche; 


    location / {
        proxy_pass http://192.168.33.10:80;
        proxy_set_header X-Real-IP $remote_addr; 
    }

 }

```

Then if your backend uses apache, use the %{VARNAME}i format string, like so for instance:

``apache
LogFormat "%h -- %{X-Real-IP}i --  paul --  %l %u %t \"%r\" %>s %b" testformat

# and then you can reference it like this...
CustomLog ${APACHE_LOG_DIR}/access.log testformat
``

Source: http://httpd.apache.org/docs/current/mod/mod_log_config.html#page-header


If your backend uses nginx, you should be (not tested personally) able to use this:

```nginx
log_format testformat '$remote_addr - blabla - "$http_x_real_ip"';

# and then reference it like this
access_log /var/log/nginx/access.log testformat;
```



Some quick configs to copy paste
--------------------------

simple html

```nginx
 server {
    listen 80; 
    server_name tan6; 
    root "/sites/bootstrap";
 }

```


simple php

```nginx
server {
	listen 80; 
	server_name tan6;
	root "/sites/bootstrap";

	# pass all php files to php-fpm/php-fcgi server 
	location ~ \.php {
	    include fastcgi_params;
	    include fastcgi.conf;
	    fastcgi_pass 127.0.0.1:9000;
	}
}
```





Sources
------------
1: https://www.digitalocean.com/community/tutorials/how-to-setup-fastcgi-caching-with-nginx-on-your-vps