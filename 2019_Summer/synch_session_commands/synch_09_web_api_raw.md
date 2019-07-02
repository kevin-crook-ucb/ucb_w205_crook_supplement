### Web API Raw Mode

You can use telnet to test unencrypted connections to a web API server:

```
telnet google.com 80
```

You can issue a GET on / using HTTP version 1.0.  Version 1.0 hangs up after every request.

```
GET / HTTP/1.0
```

You can issue a GET on / using HTTP
