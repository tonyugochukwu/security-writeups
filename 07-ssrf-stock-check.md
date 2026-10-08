SSRF — Server-Side Request Forgery via Stock Check API (PortSwigger Web Security Academy)

Lab: Basic SSRF against localhost
Category: Server-Side Request Forgery (SSRF)
Difficulty: Apprentice
Status: Solved
Tools used: Burp Suite Community Edition (Proxy, Repeater)

Objective
Exploit an SSRF vulnerability to make the server access an internal admin interface, and use it to delete the user `carlos`.

Hypothesis
Many applications include a feature where the server itself fetches a resource on behalf of the user — in this case, a "check stock" 
feature. The `POST /product/stock` request contained a `stockApi` parameter holding a full URL:
```
stockApi=http://stock.weliketoshop.net:8080/product/stock/check?productId=2&storeId=1
```
Whenever a parameter like this directly controls a URL that the server fetches (rather than the user's own browser), it's worth 
testing whether the server restricts that URL to expected, safe destinations — or whether it will fetch any URL supplied, including 
ones pointing to internal, normally unreachable systems.

 Steps

 1. Identify the server-side request
Intercepted the stock-check request and observed the `stockApi` parameter contained a complete URL that the server fetches to retrieve stock data.

 2. Replace the URL with an internal target
Using Burp Repeater, replaced the `stockApi` value with a request to the server's own internal admin functionality:
```
stockApi=http://localhost/admin/delete?username=carlos
```
Since this request originates from the server itself, `localhost` refers to the server's own machine — not the attacker's — meaning this request could reach internal functionality that is not exposed externally and would otherwise be unreachable directly.

 3. Forward the request and confirm impact
Sent the modified request. The server fetched the forged URL as instructed, executing the delete action against its own internal admin panel. The lab updated to "Solved," confirming the user `carlos` had been deleted server-side — the response did not need to visibly render an admin page back to the browser for the action to have taken effect.

 Root Cause
The server fetched whatever URL was supplied in the `stockApi` parameter without validating or restricting it to a known, expected destination (such as only allowing requests to `stock.weliketoshop.net`). Because the fetch is performed by the server itself, the server's own network position — able to reach `localhost` and any other internal-only addresses — became something an external attacker could exploit by proxy. This is the essence of Server-Side Request Forgery: the attacker cannot reach the internal resource directly, but can trick the server into reaching it on their behalf.

 Fix Recommendations
- Maintain an allowlist of permitted hosts/URLs that a server-side fetch is allowed to target, and reject anything outside it.
- Never allow user-controllable input to directly determine a server-side request's destination without validation.
- Block server-side requests to `localhost`, loopback addresses, and other internal/private IP ranges by default unless explicitly required and tightly controlled.
- Apply network-level segmentation so that even if SSRF occurs, the server cannot reach highly sensitive internal services.
