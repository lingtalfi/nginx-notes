Referer - Hot linking
=============================
2016-10-23



Some sites copy text and images from other sites.

If B.com copies content from A.com, A.com can use the referer header to know that the request comes from B.com (and prevent it).



The code below prevents hot linking.

Note that it also block search engines like google (not ideal?).

Since this is probably indesirable, we can add search engines to the valid_referers directive (see example below).




```nginx
server {
	server_name example.com;

	location / {
		root /var/www/html;
	}

	location ~ \.(jpe?g|png|gif)$ {
		valid_referers none blocked google.com bing.com serverb.com *.serverb.com;
		if ( $invalid_referer ) {
			return 403;
		}
	}
}
```




