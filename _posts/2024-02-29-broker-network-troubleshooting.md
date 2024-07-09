There are a couple of other pages on troubleshooting the Broker container [![](Broker%20Network%20Troubleshooting%20-%20Stephen%20Perciballi%20-%20Confluence/spaces%252F-MdwVZ6HOZriajCf5nXH%252Favatar-1631192016346.png)Troubleshooting Broker | Snyk User Docs](https://docs.snyk.io/enterprise-configuration/snyk-broker/troubleshooting-broker) and how to install additional tools in the Broker container, [![](Broker%20Network%20Troubleshooting%20-%20Stephen%20Perciballi%20-%20Confluence/01HZKPCSJZC0RZFW3BZGQ6KF6S.undefined)Broker Troubleshooting](https://support.snyk.io/hc/en-us/articles/4404288846353-Broker-Troubleshooting) . This guide is specific to identifying what is in the Brokers path preventing it from connecting to broker.snyk.io.

When a customer stores their code in an on-premise SCM they are typically going to have several network security solutions in place that may prevent the Broker connection to b[roker.snyk.io.](http://broker.snyk.io/ "http://broker.snyk.io") The types of network gateways include:

-   DNS Firewall
    
-   SSL/TLS Decryption
    
-   HTTP Proxy
    
-   Network Firewall
    

**All of the following commands need to be run from a host in the same subnet as the Broker. These commands are not installed in the Broker container. Ask the customer to deploy a small Linux container in the same place and subnet as they deployed Broker for troubleshooting.**

## What Success Looks Like

### Test Network Access

When connecting in a web browser you will get `{"status":"OK"}`.

When connecting with `curl -v https://broker.snyk.io` you will get `HTTP/1.1 200 OK` as displayed in the bottom third of the following output.

-   Trying 23.10.139.101:443...
    
-   Connected to [broker.snyk.io](http://broker.snyk.io/ "http://broker.snyk.io") (23.10.139.101) port 443 (#0)
    
-   ALPN: offers h2,http/1.1
    
-   TLSv1.2 (OUT), TLS handshake, Client hello (1):
    
-   CAfile: /etc/ssl/cert.pem
    
-   CApath: none
    
-   TLSv1.2 (IN), TLS handshake, Server hello (2):
    
-   TLSv1.2 (IN), TLS handshake, Certificate (11):
    
-   TLSv1.2 (IN), TLS handshake, Server key exchange (12):
    
-   TLSv1.2 (IN), TLS handshake, Server finished (14):
    
-   TLSv1.2 (OUT), TLS handshake, Client key exchange (16):
    
-   TLSv1.2 (OUT), TLS change cipher, Change cipher spec (1):
    
-   TLSv1.2 (OUT), TLS handshake, Finished (20):
    
-   TLSv1.2 (IN), TLS change cipher, Change cipher spec (1):
    
-   TLSv1.2 (IN), TLS handshake, Finished (20):
    
-   SSL connection using TLSv1.2 / ECDHE\-RSA-AES256\-GCM\-SHA384
    
-   ALPN: server accepted http/1.1
    
-   Server certificate:
    
-   subject: C=GB; L=London; O=Snyk Ltd; [CN\=snyk.io](http://cn=snyk.io/ "http://CN=snyk.io")
    
-   start date: Jan 16 00:00:00 2024 GMT
    
-   expire date: May 29 23:59:59 2024 GMT
    
-   subjectAltName: host "[broker.snyk.io](http://broker.snyk.io/ "http://broker.snyk.io")" matched cert's "[broker.snyk.io](http://broker.snyk.io/ "http://broker.snyk.io")"
    
-   issuer: C=US; O=DigiCert Inc; [OU\=www.digicert.com](http://ou=www.digicert.com/ "http://OU=www.digicert.com"); CN\=GeoTrust RSA CA 2018
    
-   SSL certificate verify ok.
    
-   using HTTP/1.1
    

> GET / HTTP/1.1  
> Host: [broker.snyk.io](http://broker.snyk.io/ "http://broker.snyk.io")  
> User-Agent: curl/8.1.2  
> Accept: _/_

< HTTP/1.1 200 OK  
< Server: nginx/1.25.1  
< Content-Type: application/octet-stream  
< Content-Type: application/json  
< Content-Length: 15  
< Expires: Thu, 29 Feb 2024 03:52:43 GMT  
< Cache-Control: max-age=0, no-cache, no-store  
< Pragma: no-cache  
< Date: Thu, 29 Feb 2024 03:52:43 GMT  
< Connection: keep-alive  
< X-Frame-Options: SAMEORIGIN  
< X-Content-Type-Options: nosniff  
< X-Xss-Protection: 1; mode=block  
< Strict-Transport-Security: max-age=31536000; preload  
<

-   Connection #0 to host [broker.snyk.io](http://broker.snyk.io/ "http://broker.snyk.io") left intact
    

### Test for HTTP Proxy/SSL Decryption

When connecting with `openssl s_client -connect broker.snyk.io:443`you should see the DigiCert certificate issued to Snyk in the subjet and issuer after `end certificate` (this output is concatenated so you may have to scroll up to find this).

`CONNECTED(00000005)`  
`depth=2 C = US, O = DigiCert Inc, OU =` [TLS/SSL Certificate Authority | Leader in Digital Trust | DigiCert](http://www.digicert.com/) `, CN = DigiCert Global Root CA`  
`verify return:1`  
`depth=1 C = US, O = DigiCert Inc, OU =` [TLS/SSL Certificate Authority | Leader in Digital Trust | DigiCert](http://www.digicert.com/) `, CN = GeoTrust RSA CA 2018`  
`verify return:1`  
`depth=0 C = GB, L = London, O = Snyk Ltd, CN = snyk.io`  
`verify return:1`

`Certificate chain`  
`0 s:C = GB, L = London, O = Snyk Ltd, CN = snyk.io`  
`i:C = US, O = DigiCert Inc, OU =` [TLS/SSL Certificate Authority | Leader in Digital Trust | DigiCert](http://www.digicert.com/) `, CN = GeoTrust RSA CA 2018`  
`a:PKEY: rsaEncryption, 2048 (bit); sigalg: RSA-SHA256`  
`v:NotBefore: Jan 16 00:00:00 2024 GMT; NotAfter: May 29 23:59:59 2024 GMT`  
`1 s:C = US, O = DigiCert Inc, OU =` [TLS/SSL Certificate Authority | Leader in Digital Trust | DigiCert](http://www.digicert.com/) `, CN = GeoTrust RSA CA 2018`  
`i:C = US, O = DigiCert Inc, OU =` [TLS/SSL Certificate Authority | Leader in Digital Trust | DigiCert](http://www.digicert.com/) `, CN = DigiCert Global Root CA`  
`a:PKEY: rsaEncryption, 2048 (bit); sigalg: RSA-SHA256`  
`v:NotBefore: Nov 6 12:23:45 2017 GMT; NotAfter: Nov 6 12:23:45 2027 GMT`

`Server certificate`  
`-----BEGIN CERTIFICATE-----`  
`MIIH7zCCBtegAwIBAgIQAqDKqZBc5QFlMWcjBP9HBzANBgkqhkiG9w0BAQsFADBe`  
`MQswCQYDVQQGEwJVUzEVMBMGA1UEChMMRGlnaUNlcnQgSW5jMRkwFwYDVQQLExB3`  
`d3cuZGlnaWNlcnQuY29tMR0wGwYDVQQDExRHZW9UcnVzdCBSU0EgQ0EgMjAxODAe`  
`Fw0yNDAxMTYwMDAwMDBaFw0yNDA1MjkyMzU5NTlaMEMxCzAJBgNVBAYTAkdCMQ8w`  
`DQYDVQQHEwZMb25kb24xETAPBgNVBAoTCFNueWsgTHRkMRAwDgYDVQQDEwdzbnlr`  
`LmlvMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAleiEAwNNIsKQoUpe`  
`3V859SwHZVKa0BpOsj0zE3fBSwLs3+Cx9uuTJSEg88UL3SH2Dr1OExVolOMZcKb9`  
`IRzFH+Njrza5BWYE5neW9XSXTKvIr18Z94ztj9fwGwYKEaXG+KLg0erawHv6V5yM`  
`VqiafHRuQfB2uOWAIqXbfzAYAgH9h76RpMWE7LpsAm/xjbrxc6ObGM0hbgRNMDFp`  
`82TiOUTyjqrgqdqqKjEGDjkSp6FCKpM9/yXa+x6awIhp+K9EnMN698zaRTiRY/LY`  
`0UyvDqh3csvPX5Q8frUU+i/qE2Ojx8WxCX1k32prm/sXk9ix39jZOyvXEb3tPuOu`  
`FJPeWwIDAQABo4IEwjCCBL4wHwYDVR0jBBgwFoAUkFj/sJx1qFFUd7Ht8qNDFjie`  
`bMUwHQYDVR0OBBYEFF6MJjmZ0/+G0WW1aOsKGLUdLPTYMIIBxgYDVR0RBIIBvTCC`  
`AbmCB3NueWsuaW+CC2FwaS5zbnlrLmlvggthcHAuc255ay5pb4IPYXBwcmlzay5z`  
`bnlrLmlvggxhc3BtLnNueWsuaW+CDGJsb2cuc255ay5pb4IOYnJva2VyLnNueWsu`  
`aW+CD2Jyb2tlcjIuc255ay5pb4ITY3AtdGVzdC1hcHAuc255ay5pb4IMZGF0YS5z`  
`bnlrLmlvghBkZWVwcm94eS5zbnlrLmlvghNkb2NzLm9ucHJlbS5zbnlrLmlvggxk`  
`b2NzLnNueWsuaW+CDGVuc28uc255ay5pb4IOaGVhbHRoLnNueWsuaW+CCmlkLnNu`  
`eWsuaW+CDGluZm8uc255ay5pb4INbGVhcm4uc255ay5pb4IMcG9nby5zbnlrLmlv`  
`ghFyZXBvcnRpbmcuc255ay5pb4IRcmVzb3VyY2VzLnNueWsuaW+CEHNlY3VyaXR5`  
`LnNueWsuaW+CEHNuaXBwZXRzLnNueWsuaW+CD3N0YWdpbmcuc255ay5pb4IOc3Rh`  
`dGljLnNueWsuaW+CD3N1cHBvcnQuc255ay5pb4IQdHJhaW5pbmcuc255ay5pb4IL`  
`d3d3LnNueWsuaW8wPgYDVR0gBDcwNTAzBgZngQwBAgIwKTAnBggrBgEFBQcCARYb`  
`aHR0cDovL3d3dy5kaWdpY2VydC5jb20vQ1BTMA4GA1UdDwEB/wQEAwIFoDAdBgNV`  
`HSUEFjAUBggrBgEFBQcDAQYIKwYBBQUHAwIwPgYDVR0fBDcwNTAzoDGgL4YtaHR0`  
`cDovL2NkcC5nZW90cnVzdC5jb20vR2VvVHJ1c3RSU0FDQTIwMTguY3JsMHUGCCsG`  
`AQUFBwEBBGkwZzAmBggrBgEFBQcwAYYaaHR0cDovL3N0YXR1cy5nZW90cnVzdC5j`  
`b20wPQYIKwYBBQUHMAKGMWh0dHA6Ly9jYWNlcnRzLmdlb3RydXN0LmNvbS9HZW9U`  
`cnVzdFJTQUNBMjAxOC5jcnQwDAYDVR0TAQH/BAIwADCCAXwGCisGAQQB1nkCBAIE`  
`ggFsBIIBaAFmAHUA7s3QZNXbGs7FXLedtM0TojKHRny87N7DUUhZRnEftZsAAAGN`  
`EZxb3QAABAMARjBEAiBKrnWT/lc4Vbu92ulVrw59jytoxNYS4qKhzJ/DwTttMAIg`  
`QcQKWpVjM9j5zyP2Hu9D4WalrlPvm8cSSUBdOto8B2kAdgBIsONr2qZHNA/lagL6`  
`nTDrHFIBy1bdLIHZu7+rOdiEcwAAAY0RnFwDAAAEAwBHMEUCIHBEPW8ZpSAjbqR6`  
`XeLVLnkt4e6mGHZMWFneTw/yrgUhAiEAy4WRC3y9zY5R53UHsXuRpch9vp+GFHIL`  
`u05wES6iTawAdQA7U3d1Pi25gE6LMFsG/kA7Z9hPw/THvQANLXJv4frUFwAAAY0R`  
`nFxEAAAEAwBGMEQCID2reur3ZG3q88xtMDr+kJwWrUNaq8j55ueWOM0ShamtAiBS`  
`Ohlo78cN2U4cHWfT/wv6ym0z0WDuI4RvsqphlaBTVjANBgkqhkiG9w0BAQsFAAOC`  
`AQEAu2b6cJXkd0BgIMS8MXBERZRU69oAAtnNE/Sg1Y3rgvasB5HcaY/QYP7gr+IF`  
`AC1QV4VHkKMiQGnn5BljrincglHuU9ldLkseuzJdmaD4Dq1+6ZD6t1z4luTrqEfT`  
`bv7YFXDytLkDoCKbCtcQ0CwZpA5LdRtNJf2cSUfBozU4uq0PIolcFg4N8f+Tcgyl`  
`C/0q8et/iK1Z9gywb0FZisOSvGwZyvYYpuZMi1cJDZixqIE3rrHFEfFaPgfHtjt0`  
`Mt6PitafMBxaEQ6Kq65XC6UdIx4STTj4V5/w/oxKxu1q/5nOe0itGOleH3+oMMML`  
`feftZIGEqPwBpXAEUM4eFEOytw==`  
`-----END CERTIFICATE-----`  
`subject=C = GB, L = London, O = Snyk Ltd, CN = snyk.io`  
`issuer=C = US, O = DigiCert Inc, OU =` [http://www.digicert.com](http://www.digicert.com/) `, CN = GeoTrust RSA CA 2018`

## Container Logs Demonstrating Network Issues

### Broker Container Logs Egress

When the outbound network traffic directed to [https://broker.snyk.io](https://broker.snyk.io/ "https://broker.snyk.io") is being blocked by any of these services you will likely see an error message from the command `docker logs <broker container ID>`

`{"name":"snyk-broker","hostname":"290e720d1f00","pid":1,"level":50,"type":"TransportError","description":0,"msg":"Failed to connect to broker server","time":"2024-02-28T19:31:33.079Z","v":0}`  
`{"name":"snyk-broker","hostname":"290e720d1f00","pid":1,"level":40,"msg":"Reconnect retry #1 of 30 in about 0s","time":"2024-02-28T19:31:33.083Z","v":0}`

### Broker Container Logs Ingress

When the inbound network traffic directed to their SCM/Jira is not able to connect you will see an error message from the command `docker logs <broker container ID>`

This one shows the egress connection is successful.

`{"name":"snyk-broker","hostname":"290e720d1f00","pid":1,"level":30,"url":"https://broker.snyk.io","token":"${BROKER_TOKEN}","metadata":{"capabilities":["post-streams"],"clientId":"9be229d6-78a7-4ef9-b032-4c0fefb6830c","preflightChecks":[{"id":"broker-server-status","name":"Broker Server Healthcheck","output":"HTTP GET https://broker.snyk.io/healthcheck: 200 OK, response: {\"ok\":true,\"version\":\"4.176.6\"}","status":"passing"},{"id":"rest-api-status","name":"REST API Healthcheck","output":"HTTP GET https://api.snyk.io/rest/openapi: 200 OK, response: [\"2021-06-01~experimental\",\"2021-06-01~beta\",\"2021-06-01\",\"2021-06-04~experimental\",\"2021-06-04~beta\",\"2021-06-04\",\"2021-06-07~experimental\",\"2021-06-07~beta\",\"2021-06-07\",\"2021-06-13~experimental\",\"2... (truncated)","status":"passing"}],"version":"4.143.0"},"msg":"successfully established a websocket connection to the broker server","time":"2024-02-28T19:39:21.260Z","v":0}`

This one shows the ingress connection to the SCM is not successful. The reason for this could be;

-   Routing from the Broker to the SCM internally is not established.
    
-   Internal firewalls.
    

`{"name":"snyk-broker","hostname":"290e720d1f00","pid":1,"level":50,"url":"/api/v4/user","requestId":"6f8a6904-124f-4b9a-ac5d-8060cf10d529","streamingID":"42b335d8-bcc9-4313-9d46-ca1accce79a5","maskedToken":"2ef5-...-96b5","transport":"websocket","httpUrl":"https://gitlab.perciballi.ca/api/v4/user","userAgentHeaderSet":true,"headerVarsSubstitution":"private-token","error":{"name":"Error","message":"connect ECONNREFUSED 192.168.1.76:443","stack":"Error: connect ECONNREFUSED 192.168.1.76:443","errno":-111,"code":"ECONNREFUSED","syscall":"connect","address":"192.168.1.76","port":443},"stackTrace":"Error: stacktrace generator\n at Request.<anonymous> (/home/node/.npm-global/lib/node_modules/snyk-broker/dist/lib/stream-posts.js:161:25)\n at Request.onRequestError (/home/node/.npm-global/lib/node_modules/snyk-broker/node_modules/request/request.js:877:8)\n at emitErrorNT (node:internal/streams/destroy:157:8)\n at emitErrorCloseNT (node:internal/streams/destroy:122:3)\n at processTicksAndRejections (node:internal/process/task_queues:83:21)","msg":"received error from request while piping to Broker Server","time":"2024-02-28T19:39:53.546Z","v":0}`

## What Filtering Looks Like

### DNS Firewall

Typically you will get a response with an IP address owned by Akamai if there is no DNS firewalling. Otherwise you may get a response with `0.0.0.0, 127.0.0.1, or some other IP address instead`. To test for DNS firewalling from the host running Broker run `dig broker.snyk.io`.

`; <<>> DiG 9.10.6 <<>> broker.snyk.io`  
`;; global options: +cmd`  
`;; Got answer:`  
`;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 3133`  
`;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1`

`;; OPT PSEUDOSECTION:`  
`; EDNS: version: 0, flags:; udp: 4096`  
`;; QUESTION SECTION:`  
`;broker.snyk.io. IN A`

`;; ANSWER SECTION:`  
`broker.snyk.io. 60 IN A 0.0.0.0`

`;; Query time: 56 msec`  
`;; SERVER: 127.0.2.2#53(127.0.2.2)`  
`;; WHEN: Wed Feb 28 16:01:16 EST 2024`  
`;; MSG SIZE rcvd: 59`

### Identify SSL/TLS Decryption

SSL Decryption is found on an HTTP proxy, application firewall, or dedicated appliance. They are employed for accurately controlling access to Internet applications, Data Loss Prevention (scanning network traffic for sensitive data), Network Intrusion Prevention, malware detection/prevention, Web Application Firewall, and general network visibility.

You can identify if there is a proxy between the Broker agent and [https://broker.snyk.io](https://broker.snyk.io/ "https://broker.snyk.io") by examining the certificate data. The command to view the SSL certificate is `openssl s_client -connect broker.snyk.io:443`

Without SSL Decryption in place the Subject of the certificate is Snyk Ltd and the Issuer is Digicert. When there is a proxy involved you will see a different certificate chain. For example below you can see the Cloudflare Intermediate Certificate Authority, rather than the Digicert certificate issued directly to Snyk in the next section. Some parts of the certificate that are not required for this have been pruned.

`CONNECTED(00000005)`  
`depth=2 C = US, ST = California, L = San Francisco, O = "Cloudflare, Inc", CN = Cloudflare for Teams ECC Certificate Authority`  
`verify return:1`  
`depth=1 C = US, ST = California, L = San Francisco, O = "Cloudflare, Inc.", OU = Gateway Intermediate ECC Certificate Authority`  
`verify return:1`  
`depth=0 CN = broker.snyk.io`  
`verify return:1`

`Certificate chain`  
`0 s:CN = broker.snyk.io`  
`i:C = US, ST = California, L = San Francisco, O = "Cloudflare, Inc.", OU = Gateway Intermediate ECC Certificate Authority`  
`a:PKEY: id-ecPublicKey, 384 (bit); sigalg: ecdsa-with-SHA256`  
`v:NotBefore: Feb 27 13:23:00 2024 GMT; NotAfter: Jun 30 19:56:21 2024 GMT`  
`1 s:C = US, ST = California, L = San Francisco, O = "Cloudflare, Inc.", OU = Gateway Intermediate ECC Certificate Authority`  
`i:C = US, ST = California, L = San Francisco, O = "Cloudflare, Inc", CN = Cloudflare for Teams ECC Certificate Authority`  
`a:PKEY: id-ecPublicKey, 384 (bit); sigalg: ecdsa-with-SHA512`  
`v:NotBefore: Feb 27 13:23:00 2024 GMT; NotAfter: Apr 2 13:23:00 2024 GMT`

`Server certificate`  
`-----BEGIN CERTIFICATE-----`  
`MIICKzCCAbCgAwIBAgIEdictGDAKBggqhkjOPQQDAjCBjjELMAkGA1UEBhMCVVMx`  
`EzARBgNVBAgTCkNhbGlmb3JuaWExFjAUBgNVBAcTDVNhbiBGcmFuY2lzY28xGTAX`  
`BgNVBAoTEENsb3VkZmxhcmUsIEluYy4xNzA1BgNVBAsTLkdhdGV3YXkgSW50ZXJt`  
`ZWRpYXRlIEVDQyBDZXJ0aWZpY2F0ZSBBdXRob3JpdHkwHhcNMjQwMjI3MTMyMzAw`  
`WhcNMjQwNjMwMTk1NjIxWjAZMRcwFQYDVQQDDA5icm9rZXIuc255ay5pbzB2MBAG`  
`ByqGSM49AgEGBSuBBAAiA2IABDjei+Zyw0tax8LNUzoIUL+qeTXCk96HevvmlosT`  
`KZgqZssvLDllojsWsb8wqoMTPlfUDcSK+DbbuuvMq5XhwIeOdAPJb+cerWICM0t8`  
`25H7lhFmzFLeQ/L1BYjtnFPxUaNTMFEwEwYDVR0lBAwwCgYIKwYBBQUHAwEwGQYD`  
`VR0RBBIwEIIOYnJva2VyLnNueWsuaW8wHwYDVR0jBBgwFoAUhmpv5ZZBH58lBkly`  
`aPdZXnOiQ2owCgYIKoZIzj0EAwIDaQAwZgIxAKzsFvzQolpeWq6L1MMS8WLmkkwg`  
`N2huOiTfILhCc4vp08hnHq9vYkCbjXb/MZjxdAIxAKbVnhYv890000oDvaNXzsCj`  
`0C9bv73K/RhZBMYCc1pb5viRo+XP1pbxbzV2PHWa8g==`  
`-----END CERTIFICATE-----`  
`subject=CN = broker.snyk.io`  
`issuer=C = US, ST = California, L = San Francisco, O = "Cloudflare, Inc.", OU = Gateway Intermediate ECC Certificate Authority`

### HTTP Proxy

curl -v [https://broker.snyk.io](https://broker.snyk.io/ "https://broker.snyk.io")

#### 300 Redirects

When the Broker agent is behind an HTTP proxy that is blocking the application traffic you may see a 400 or 300 HTTP message. It may be interpreted as successful because there is a connection, however the agent is only connected to the proxy. When the connection is actually successful you will get a 200 message, as shown in the next example.

For example with an HTTP proxy blocking traffic to [broker.snyk.io](http://broker.snyk.io/ "http://broker.snyk.io") you get the following.

-   `Trying [2600:140a:5000:58e::ecd]:443...`
    
-   `Connected to broker.snyk.io (2600:140a:5000:58e::ecd) port 443 (#0)`
    
-   `ALPN: offers h2,http/1.1`
    
-   `TLSv1.2 (OUT), TLS handshake, Client hello (1):`
    
-   `CAfile: /etc/ssl/cert.pem`
    
-   `CApath: none`
    
-   `TLSv1.2 (IN), TLS handshake, Server hello (2):`
    
-   `TLSv1.2 (IN), TLS handshake, Certificate (11):`
    
-   `TLSv1.2 (IN), TLS handshake, Server key exchange (12):`
    
-   `TLSv1.2 (IN), TLS handshake, Server finished (14):`
    
-   `TLSv1.2 (OUT), TLS handshake, Client key exchange (16):`
    
-   `TLSv1.2 (OUT), TLS change cipher, Change cipher spec (1):`
    
-   `TLSv1.2 (OUT), TLS handshake, Finished (20):`
    
-   `TLSv1.2 (IN), TLS change cipher, Change cipher spec (1):`
    
-   `TLSv1.2 (IN), TLS handshake, Finished (20):`
    
-   `SSL connection using TLSv1.2 / ECDHE-ECDSA-AES256-GCM-SHA384`
    
-   `ALPN: server accepted h2`
    
-   `Server certificate:`
    
-   `subject: CN=broker.snyk.io`
    
-   `start date: Feb 28 08:43:00 2024 GMT`
    
-   `expire date: May 26 02:03:59 2024 GMT`
    
-   `subjectAltName: host "broker.snyk.io" matched cert's "broker.snyk.io"`
    
-   `issuer: C=US; ST=California; L=San Francisco; O=Cloudflare, Inc.; OU=Gateway Intermediate ECC Certificate Authority`
    
-   `SSL certificate verify ok.`
    
-   `using HTTP/2`
    
-   `h2 [:method: GET]`
    
-   `h2 [:scheme: https]`
    
-   `h2 [:authority: broker.snyk.io]`
    
-   `h2 [:path: /]`
    
-   `h2 [user-agent: curl/8.1.2]`
    
-   `h2 [accept: /]`
    
-   `Using Stream ID: 1 (easy handle 0x7fc0db810a00)`
    

> `GET / HTTP/2`  
> `Host: broker.snyk.io`  
> `User-Agent: curl/8.1.2`  
> `Accept: /`

`< HTTP/2 302`  
`< location: https://blocked.teams.cloudflare.com/?footer_text=This+blocked+traffic+was+logged.&account_id=b5600e65c0fdb5395946aff16db6cc75&logo_path=https%3A%2F%2Fdesignlooter.com%2Fimages%2Fkateboard-svg-5.png&device_id=155b3eb5-9ee1-11eb-9439-0215ca94cfd6&source_ip=104.222.114.181%3A59457&header_text=This+website+is+blocked&suppress_footer=false&mailto_address=&mailto_subject=&background_color=%23000000&url=https%3A%2F%2Fbroker.snyk.io%2F&rule_id=5469cff4-2727-4a2a-a584-925e9588ac34&params_sign=aMfeFpr7rgETy%2BfUPVuFJuWVvBR9kg3vW0VB13Wf1X8%3D&user_id=ad3c1692-94b2-413e-a8e0-af2c5160e716&name=Perciballi+Organization`  
`< cf-team: 1e9434e216000039d8f7aaf400000001`  
`< date: Thu, 29 Feb 2024 02:05:12 GMT`  
`<`

-   `Connection #0 to host broker.snyk.io left intact`
    

Connection successful and either not behind an HTTP proxy or the proxy is configured to permit traffic to broker.snyk.io.

-   `Trying 184.24.168.123:443...`
    
-   `Connected to broker.snyk.io (184.24.168.123) port 443 (#0)`
    
-   `ALPN: offers h2,http/1.1`
    
-   `TLSv1.2 (OUT), TLS handshake, Client hello (1):`
    
-   `CAfile: /etc/ssl/cert.pem`
    
-   `CApath: none`
    
-   `TLSv1.2 (IN), TLS handshake, Server hello (2):`
    
-   `TLSv1.2 (IN), TLS handshake, Certificate (11):`
    
-   `TLSv1.2 (IN), TLS handshake, Server key exchange (12):`
    
-   `TLSv1.2 (IN), TLS handshake, Server finished (14):`
    
-   `TLSv1.2 (OUT), TLS handshake, Client key exchange (16):`
    
-   `TLSv1.2 (OUT), TLS change cipher, Change cipher spec (1):`
    
-   `TLSv1.2 (OUT), TLS handshake, Finished (20):`
    
-   `TLSv1.2 (IN), TLS change cipher, Change cipher spec (1):`
    
-   `TLSv1.2 (IN), TLS handshake, Finished (20):`
    
-   `SSL connection using TLSv1.2 / ECDHE-RSA-AES256-GCM-SHA384`
    
-   `ALPN: server accepted http/1.1`
    
-   `Server certificate:`
    
-   `subject: C=GB; L=London; O=Snyk Ltd; CN=snyk.io`
    
-   `start date: Jan 16 00:00:00 2024 GMT`
    
-   `expire date: May 29 23:59:59 2024 GMT`
    
-   `subjectAltName: host "broker.snyk.io" matched cert's "broker.snyk.io"`
    
-   `issuer: C=US; O=DigiCert Inc; OU=www.digicert.com; CN=GeoTrust RSA CA 2018`
    
-   `SSL certificate verify ok.`
    
-   `using HTTP/1.1`
    

> `GET / HTTP/1.1`  
> `Host: broker.snyk.io`  
> `User-Agent: curl/8.1.2`  
> `Accept: /`

`< HTTP/1.1 200 OK`  
`< Server: nginx/1.25.1`  
`< Content-Type: application/octet-stream`  
`< Content-Type: application/json`  
`< Content-Length: 15`  
`< Expires: Thu, 29 Feb 2024 03:21:00 GMT`  
`< Cache-Control: max-age=0, no-cache, no-store`  
`< Pragma: no-cache`  
`< Date: Thu, 29 Feb 2024 03:21:00 GMT`  
`< Connection: keep-alive`  
`< X-Frame-Options: SAMEORIGIN`  
`< X-Content-Type-Options: nosniff`  
`< X-Xss-Protection: 1; mode=block`  
`< Strict-Transport-Security: max-age=31536000; preload`  
`<`

-   `Connection #0 to host broker.snyk.io left intact`
    

#### 403 Forbidden

When the Broker agent is behind an HTTP proxy that is blocking the network traffic destined to TCP port 443 you may see a 400 HTTP message. The message is a proxy redirect to an error page letting the user know it is not allowed. It may be interpreted as successful because there is a connection, however the agent is only connected to the proxy, which is issuing an error. When the connection is actually successful you will get a 200 message, as shown in the next example.

-   `Trying [2600:140a:5000:58e::ecd]:443...`
    
-   `Connected to broker.snyk.io (2600:140a:5000:58e::ecd) port 443 (#0)`
    
-   `ALPN: offers h2,http/1.1`
    
-   `TLSv1.2 (OUT), TLS handshake, Client hello (1):`
    
-   `CAfile: /etc/ssl/cert.pem`
    
-   `CApath: none`
    
-   `TLSv1.2 (IN), TLS handshake, Server hello (2):`
    
-   `TLSv1.2 (IN), TLS handshake, Certificate (11):`
    
-   `TLSv1.2 (IN), TLS handshake, Server key exchange (12):`
    
-   `TLSv1.2 (IN), TLS handshake, Server finished (14):`
    
-   `TLSv1.2 (OUT), TLS handshake, Client key exchange (16):`
    
-   `TLSv1.2 (OUT), TLS change cipher, Change cipher spec (1):`
    
-   `TLSv1.2 (OUT), TLS handshake, Finished (20):`
    
-   `TLSv1.2 (IN), TLS change cipher, Change cipher spec (1):`
    
-   `TLSv1.2 (IN), TLS handshake, Finished (20):`
    
-   `SSL connection using TLSv1.2 / ECDHE-ECDSA-AES256-GCM-SHA384`
    
-   `ALPN: server accepted h2`
    
-   `Server certificate:`
    
-   `subject: CN=broker.snyk.io`
    
-   `start date: Feb 28 13:24:00 2024 GMT`
    
-   `expire date: Jun 25 03:35:49 2024 GMT`
    
-   `subjectAltName: host "broker.snyk.io" matched cert's "broker.snyk.io"`
    
-   `issuer: C=US; ST=California; L=San Francisco; O=Cloudflare, Inc.; OU=Gateway Intermediate ECC Certificate Authority`
    
-   `SSL certificate verify ok.`
    
-   `using HTTP/2`
    
-   `h2 [:method: GET]`
    
-   `h2 [:scheme: https]`
    
-   `h2 [:authority: broker.snyk.io]`
    
-   `h2 [:path: /]`
    
-   `h2 [user-agent: curl/8.1.2]`
    
-   `h2 [accept: /]`
    
-   `Using Stream ID: 1 (easy handle 0x7fd8f0013e00)`
    

> `GET / HTTP/2`  
> `Host: broker.snyk.io`  
> `User-Agent: curl/8.1.2`  
> `Accept: /`

`< HTTP/2 403`  
`< cache-control: private, max-age=0, no-store, no-cache, must-revalidate, post-check=0, pre-check=0`  
`< referrer-policy: same-origin`  
`< expires: Thu, 01 Jan 1970 00:00:01 GMT`  
`< cf-team: 1e948910a30000a22cb11d0400000001`  
`< date: Thu, 29 Feb 2024 03:37:09 GMT`  
`< content-length: 4328`  
`<`

Be the first to add a reaction