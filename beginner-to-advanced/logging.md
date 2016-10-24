Logging
=============================
2016-10-23



Access log
===============

- $remote_addr: ip address of the client
- $remote_user: user if basic authentication
- $time_local: date
- $request: http request line
- $status: status code
- $body_bytes_sent: size of the response
- $http_referer: last url visited
- $http_user_agent: browser info 
- $http_x_forwarded_for: useful for when there is a proxy in between



http {
	log_format main '$remote_addr - $remote_user [$time_local] "$request" '
					'$status $body_bytes_sent "$http_referer"'
					'"$http_user_agent" "$http_x_forwarded_for"';

	log_format context_name '$remote_addr - $status';


	access_log /var/log/nginx/access.log main; # main context
	#access_log /var/log/nginx/access.log main; # context_name context



	server {
		server_name example.com;
		access_log /var/log/nginx/example.log main;

		location / {
			root /var/www/html;
		}
	}

}


Error log
===============

- emerg
- alert
- crit
- error
- warn
- notice
- info
- debug



Formats are predefined (unlike the access log formats which are customizable);
however we can define the verbosity level.


```nginx
error_log /var/log/nginx/error.log;
#error_log /var/log/nginx/error.log notice;
#error_log /var/log/nginx/error.log info;
```


Example of error
```txt
2016/10/23 17:55:56 [error] 10768#0: *158 no user/password was provided for basic authentication, client: 192.168.33.1, server: example.com, request: "GET /admin HTTP/1.1", host: "example.com"
```

The 10768 number (after [error]) is the process id of the worker that handled the request.

#0 means it is single threaded (nginx is single threaded).












