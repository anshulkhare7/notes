# Understanding HTTP Security Headers

## HTTP Strict Transport Security

HTTP Strict Transport Security, also known as HSTS, is a protocol standard which enforces secure connections to the server via HTTP over SSL/TLS. HSTS is configured and transmitted from the server to any HTTP web client using the HTTP header Strict-Transport-Security.

` Strict-Transport-Security: max-age=63072000; includeSubDomains; preload `

The server needs to send the Strict-Transport-Security header with a given value, which represents the duration of time in seconds that the web client should send requests with a secure HTTPS connection. 

Refer to https://hstspreload.org/ for more details
