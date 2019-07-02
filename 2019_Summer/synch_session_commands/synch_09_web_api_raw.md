### Web API Raw Mode

You can use telnet to test unencrypted connections to a web API server:

```
telnet google.com 80
```

You can issue a GET on / using HTTP version 1.0.  Version 1.0 hangs up after every request.

```
GET / HTTP/1.0
hit enter if no payload to be sent
```

You can issue a GET on / using HTTP version 1.1.  It will hang on to the connection until it times out or until you hang up.

```
GET / HTTP/1.1
hit enter if no payload to be sent
```
