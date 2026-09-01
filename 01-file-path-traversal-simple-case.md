File Path Traversal — Simple Case (PortSwigger Web Security Academy)

Lab: [File path traversal, simple case](https://portswigger.net/web-security/file-path-traversal/lab-simple)
Difficulty: Apprentice
Status: Solved
Tools used:** Burp Suite Community Edition (Proxy, Repeater)

Objective

Retrieve the contents of `/etc/passwd` by exploiting a path traversal vulnerability in the product image display functionality.

Hypothesis

While browsing the shop, I noticed that product images were being requested via a URL parameter that exposed the actual filename being loaded from the server (`/image?filename=42.jpg`). Having the filename visible and directly editable in the URL looked suspicious — it suggested the server might be taking that value and using it to build a file path without validating it, which could allow requesting files outside the intended images directory.

Steps

1\. Identify the vulnerable request

Browsing a product page triggers a request to load its image:

```
GET /image?filename=42.jpg HTTP/2
Host: <lab-id>.web-security-academy.net
```

[Screenshot: original request + response — image loads normally

2\. Modify the request

Sent the request to Burp Repeater and changed the `filename` parameter to traverse up and out of the images directory to the system's password file:

```
GET /image?filename=../../../etc/passwd HTTP/2
Host: <lab-id>.web-security-academy.net
```

3\. Observe the response

The response returned the raw contents of `/etc/passwd` as plain text instead of an image, confirming the traversal worked.

[Screenshot: modified request + response — /etc/passwd contents]

4\. Confirm

PortSwigger marked the lab as **Solved** upon receiving the payload.

Payload

```
../../../etc/passwd
```

Each `../` moves up one directory level. Starting from wherever the images are stored on the server, three levels up reaches the filesystem root, from which `etc/passwd` is a direct, valid path — landing on a well-known Linux system file used here purely to prove file read access.

Root Cause

The server takes the `filename` parameter directly from user input and concatenates it onto a base directory path to locate the file (something like `/var/www/images/` + `filename`) without sanitizing or validating the input. Because there's no check for `../` sequences or any restriction confining the resolved path to the intended images folder, an attacker can supply a relative path that escapes that folder entirely and reads arbitrary files elsewhere on the filesystem.

Fix Recommendations

Validate that the resolved file path stays within the intended base directory (e.g. canonicalize the path and check it starts with the expected prefix).
* Avoid taking a raw filename from user input at all — instead, map an ID or index to a fixed, server-controlled filename.
* Strip or reject any input containing `../`, `..\\`, or URL-encoded traversal sequences as a defense-in-depth measure (not a substitute for proper path validation).

