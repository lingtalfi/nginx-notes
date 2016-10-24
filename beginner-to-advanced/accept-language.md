Accept language
=============================
2016-10-23



```http
Accept-Language: da, en-gb;q=0.8, en;q=0.7
Content-Language: en
```

Content-Language header: determines the language of the message body for the INTENDED AUDIENCE.

```bash
curl --header "Accept-Language: en" example.com/hi.html
curl --header "Accept-Language: ja" example.com/hi.html
```





