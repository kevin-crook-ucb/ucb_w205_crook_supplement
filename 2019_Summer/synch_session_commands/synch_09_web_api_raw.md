### Web API Raw Mode

### Using telnet in raw mode

You can use telnet to test unencrypted connections to a web API server.  He's how to connect to google.com:

```
telnet google.com 80
```

Inside the telnet session, you can issue a GET on / using HTTP version 1.0.  Version 1.0 hangs up after every request.

```
GET / HTTP/1.0
hit enter if no headers to be sent
```

Inside the telnet session, you can issue a GET on / using HTTP version 1.1.  It will hang on to the connection so you can make multiple requests.  If you hang up with a control-C it will also hang up.  If it sits a while without you entering a command, it will timeout and hang up.

```
GET / HTTP/1.1
hit enter if no headers to be sent
```

The response will be in this format:

* the first line will give you the version of HTTP, the return code, and the return code string. 200 is success.  400's are errors.

* the next line starts the headers in the format of key colon values.  note the cookies are set this way using headers.

* a blank line follows

* if it's HTTP/1.1, the next line has the number of bytes in the return payload.  We need this because we need to know when the message ends as it will be multiple TCP/IP packets.  if it's HTTP/1.0, this line is omitted

* the next line starts the return payload, typically HTML or encoded binary for images, videos, etc.

Let's now do a POST instead of a get.  A POST is similar to GET, except it allows us to pass a payload. In the modern era, it's customary to pass JSON for the payload (previously it was customary to pass key / value pairs)

```
telnet httpbin.org 80
```

Inside the telnet session, we will issue the POST command.  We must at least give a web header Content-Length that must of course match our payload size.  We follow with a blank line.  We follow that with the payload which could be multi-line.  We follow that with a blank line to signal the send of the POST.

```
POST /post HTTP/1.0
Content-Length: 15
hit enter if no more headers to be sent
{'key':'value'}
hit enter when the payload is complete 
```

### Using openssl in raw mode

telnet only works for unencrypted unauthenticated http traffic.  If a website or web API server is running https, then the traffic will be encrypted and also must be authenticated. There is a utility in your virtual machine (available in all Linux versions that I know of) called openssl that we can use in place of telnet.

To connect to google.com using https:

```
openssl s_client -connect google.com:443
```

It will respond with the details of the authentication, including the certificate and the 3rd party validation chain for the security certificate

You can can enter commands as before with telnet.

In the above example, we are using google.com which only had 1 host per IP address.  It's possible for a web server to have multiple hosts per IP address, typically using a system called **SNI (Server Name Identification)**.  When using SNI, we will to add a web header with the host name, such as Host: google.com

One really good website to learn how to use a web API is **Where is the ISS at?**  This website tracks the location of the International Space Station.  It uses HTTPS and returns everything in JSON format.

https://wheretheiss.at/w/developer

you can connect like this:

```
openssl s_client -connect api.wheretheiss.at:443
```

you can try commands like these.  since they use SNI to run multiple hosts on the same ip address, we need to add a header with the specific host:

```
GET /v1/satellites HTTP/1.1
Host: api.wheretheiss.at
hit enter if no more headers to be sent

GET /v1/satellites/25544/tles HTTP/1.1
Host: api.wheretheiss.at
hit enter if no more headers to be sent

GET /v1/satellites/25544/tles?format=text HTTP/1.1
Host: api.wheretheiss.at
hit enter if no more headers to be sent
```

### using openssl to study certificates further

If you would like to study certificates futher.  Here are some examples:

The following command will download and extract the final "leaf" certificate for google.com and save it into a file google.com.crt
```
echo | openssl s_client -connect google.com:443 2>&1 | sed --quiet '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' > google.com.1.crt
```

To look at the certificate in machine readable format (Base64 encoded ASCII files):
```
cat google.com.1.crt
```

To look at the certificate in human readable text:
```
openssl x509 -in google.com.1.crt -noout -text
```

If you want to extract the entire certificate chain, add the -showcerts option:
```
echo | openssl s_client -connect google.com:443 -showcerts 2>&1 | sed --quiet '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' > google.com.2.crt
```

To look at the certificate in machine readable format (Base64 encoded ASCII files):
```
cat google.com.2.crt
```

To look at the certificate in human readable text:
```
openssl x509 -in google.com.2.crt -noout -text
```

I you want to extract a certificate from a host using SNI, where there are multiple hosts, add the -servername option:
```
echo | openssl s_client -connect api.wheretheiss.at:443 -showcerts -servername api.wheretheiss.at 2>&1 | sed --quiet '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' > api.wheretheiss.at.crt
```

To look at the certificate in machine readable format (Base64 encoded ASCII files):
```
cat api.wheretheiss.at.crt
```

To look at the certificate in human readable text:
```
openssl x509 -in api.wheretheiss.at.crt -noout -text
```

### using openssl to generate RSA public key / private key pair and examine them

Since we are on the subject of openssl, you may also want to generate an RSA public key / private key pair and see what they look like.

Create a public RSA private key:
```
openssl genrsa -out rsa_private.key 4096
```

Take a look at the RSA private key in machine readable format (Base64 encoded ASCII files):
```
cat rsa_private.key
```

Take a look at the RSA private key in human readable text:
```
openssl rsa -noout -text -in rsa_private.key
```

Generate an RSA public key from corresponding RSA private key:
```
openssl rsa -in rsa_private.key -pubout -out rsa_public.key
```

Take a look at the RSA public key in machine readable format (Base64 encoded ASCII files):
```
cat rsa_public.key
```

Take a look at the RSA public key in human readable text:
```
openssl rsa -noout -text -pubin -in rsa_public.key
```




