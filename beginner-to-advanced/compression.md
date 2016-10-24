Compression
=============================
2016-10-23




```txt
browser --->  Accept-Encoding: gzip,deflate  ----> server
browser <---  Content-Encoding: gzip  <---- server
			  Content-Type: text/plain
```

Note: the Content-Type is of the type of the original file (not the encoded file).




Compression applies better on text based data (not videos or images).



```nginx

gzip on;
gzip_types text/plain text/css text/xml text/javascript; # required
gzip_disable "MSIE [1-6]\.";
gzip_comp_level 1;
```


```bash
curl http://example.com/mytext.txt > c1.txt
curl -H "Accept-Encoding: gzip" http://example.com/mytext.txt > c2.txt
```




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





