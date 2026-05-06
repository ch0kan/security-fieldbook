# How the Web Works

The web is built on a client-server model. A client, usually a browser, sends an HTTP request to a server. The server processes the request and returns an HTTP response containing content such as HTML, CSS, JavaScript, images, JSON, or other resources.

Understanding how the web works is essential for web application testing, API testing, vulnerability discovery, and secure development.

---

## Client-Server Model

The client-server model is the basic communication pattern used by websites and web applications.

```text
Client -> Request -> Server
Client <- Response <- Server
```

Examples of clients:

- Web browsers
- Mobile apps
- API clients
- cURL
- Burp Suite
- Python scripts

Examples of servers:

- Apache
- Nginx
- IIS
- Node.js
- Python Flask / Django apps
- PHP applications

---

## Web Request Flow

When a user visits a website, several steps happen.

Example:

```text
https://example.com/login
```

Typical flow:

1. The browser parses the URL.
2. DNS resolves the domain name to an IP address.
3. The browser connects to the server.
4. If HTTPS is used, a TLS handshake establishes encryption.
5. The browser sends an HTTP request.
6. The server processes the request.
7. The server returns an HTTP response.
8. The browser renders the response.

```text
Browser
  -> DNS lookup
  -> TCP/TLS connection
  -> HTTP request
  -> Server processing
  <- HTTP response
  <- Rendered page
```

---

## URL Anatomy

A URL identifies where a resource is and how to access it.

Example:

```text
https://example.com:443/blog/post?id=10#comments
```

| Component | Example | Purpose |
|---|---|---|
| Scheme | `https://` | Protocol used |
| Host | `example.com` | Domain or IP address |
| Port | `:443` | Service port |
| Path | `/blog/post` | Resource location |
| Query string | `?id=10` | Parameters sent to the server |
| Fragment | `#comments` | Client-side page section |

### Security Notes

Avoid putting sensitive data in URLs.

Bad examples:

```text
https://example.com/reset?token=secret-token
https://user:password@example.com
```

URLs can appear in:

- Browser history
- Server logs
- Proxy logs
- Referrer headers
- Screenshots

---

## HTTP and HTTPS

HTTP stands for HyperText Transfer Protocol. It is the main protocol used for web communication.

HTTPS is HTTP protected by TLS encryption.

| Protocol | Port | Security |
|---|---:|---|
| HTTP | 80 | Unencrypted |
| HTTPS | 443 | Encrypted with TLS |

### HTTP

HTTP sends data in cleartext.

This means anyone who can capture the traffic may be able to read requests, responses, cookies, credentials, or submitted data.

### HTTPS

HTTPS protects HTTP traffic with TLS.

HTTPS provides:

- Confidentiality
- Integrity
- Server authentication
- Protection against basic eavesdropping

!!! note
    HTTPS protects the content of the communication, but it does not automatically make the web application itself secure.

---

## HTTP Messages

HTTP communication uses messages.

There are two main types:

| Message Type | Direction | Purpose |
|---|---|---|
| Request | Client to server | Ask for a resource or perform an action |
| Response | Server to client | Return a result |

Both requests and responses commonly contain:

- Start line
- Headers
- Empty line
- Optional body

---

## HTTP Request Structure

A basic HTTP request looks like this:

```http
GET /index.html HTTP/1.1
Host: example.com
User-Agent: Mozilla/5.0
Accept: text/html

```

Parts of a request:

| Part | Purpose |
|---|---|
| Request line | Defines method, path, and HTTP version |
| Headers | Metadata about the request |
| Empty line | Separates headers from body |
| Body | Optional data sent to the server |

---

## HTTP Request Line

The request line is the first line of an HTTP request.

```http
GET /login HTTP/1.1
```

| Part | Example | Purpose |
|---|---|---|
| Method | `GET` | Action to perform |
| Path | `/login` | Resource being requested |
| Version | `HTTP/1.1` | HTTP version |

---

## HTTP Methods

HTTP methods describe the action the client wants to perform.

| Method | Purpose |
|---|---|
| `GET` | Retrieve data |
| `POST` | Submit or create data |
| `PUT` | Replace or update data |
| `PATCH` | Partially update data |
| `DELETE` | Delete data |
| `HEAD` | Retrieve headers only |
| `OPTIONS` | Show supported methods |

Security relevance:

- Sensitive actions should not be performed with unauthenticated `GET` requests.
- `PUT`, `PATCH`, and `DELETE` require strict authorization.
- `OPTIONS` can reveal enabled methods.
- `TRACE` is usually disabled because of security concerns.

---

## HTTP Request Headers

Request headers provide metadata about the client, session, content, or expected response.

Common request headers:

| Header | Purpose |
|---|---|
| `Host` | Identifies the target website on the server |
| `User-Agent` | Identifies the client software |
| `Accept` | Describes acceptable response types |
| `Cookie` | Sends stored cookies to the server |
| `Authorization` | Sends credentials or tokens |
| `Content-Type` | Describes request body format |
| `Content-Length` | Size of request body |

Example:

```http
POST /login HTTP/1.1
Host: example.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 29

username=admin&password=test
```

---

## HTTP Request Body

The request body contains data sent to the server.

It is common with:

- `POST`
- `PUT`
- `PATCH`

Common body formats:

| Format | Content-Type |
|---|---|
| URL encoded form | `application/x-www-form-urlencoded` |
| Multipart form | `multipart/form-data` |
| JSON | `application/json` |
| XML | `application/xml` |

### URL Encoded Form

```http
POST /login HTTP/1.1
Host: example.com
Content-Type: application/x-www-form-urlencoded

username=admin&password=password123
```

### JSON Body

```http
POST /api/login HTTP/1.1
Host: example.com
Content-Type: application/json

{
  "username": "admin",
  "password": "password123"
}
```

### Multipart Form Data

Multipart form data is commonly used for file uploads.

```http
POST /upload HTTP/1.1
Host: example.com
Content-Type: multipart/form-data; boundary=----Boundary123

------Boundary123
Content-Disposition: form-data; name="file"; filename="image.jpg"
Content-Type: image/jpeg

[Binary data]
------Boundary123--
```

---

## HTTP Response Structure

A basic HTTP response looks like this:

```http
HTTP/1.1 200 OK
Server: nginx
Content-Type: text/html
Content-Length: 1256

<html>...</html>
```

Parts of a response:

| Part | Purpose |
|---|---|
| Status line | Shows HTTP version and status code |
| Headers | Metadata about the response |
| Empty line | Separates headers from body |
| Body | Returned content |

---

## HTTP Status Codes

Status codes describe the result of a request.

| Range | Category | Meaning |
|---|---|---|
| `100-199` | Informational | Request received, continue |
| `200-299` | Success | Request succeeded |
| `300-399` | Redirection | Client should go elsewhere |
| `400-499` | Client error | Problem with the request |
| `500-599` | Server error | Problem on the server |

Common status codes:

| Code | Meaning |
|---|---|
| `200` | OK |
| `201` | Created |
| `301` | Moved Permanently |
| `302` | Found / temporary redirect |
| `400` | Bad Request |
| `401` | Unauthorized / authentication required |
| `403` | Forbidden |
| `404` | Not Found |
| `405` | Method Not Allowed |
| `500` | Internal Server Error |
| `503` | Service Unavailable |

Security relevance:

- `401` means authentication is required.
- `403` means access is denied.
- `404` may hide whether a resource exists.
- Verbose `500` errors can leak backend details.

---

## HTTP Response Headers

Response headers provide metadata and browser instructions.

Common response headers:

| Header | Purpose |
|---|---|
| `Server` | Identifies server software |
| `Content-Type` | Tells browser how to interpret the body |
| `Content-Length` | Size of response body |
| `Set-Cookie` | Tells browser to store a cookie |
| `Location` | Redirect destination |
| `Cache-Control` | Controls caching behavior |

Example:

```http
HTTP/1.1 302 Found
Location: /login
Set-Cookie: session=abc123; HttpOnly; Secure
```

---

## Security Headers

Security headers tell browsers to enforce protections.

| Header | Purpose |
|---|---|
| `Content-Security-Policy` | Restricts where scripts and resources can load from |
| `Strict-Transport-Security` | Forces HTTPS usage |
| `X-Frame-Options` | Helps prevent clickjacking |
| `Referrer-Policy` | Controls referrer leakage |
| `X-Content-Type-Options` | Prevents MIME sniffing |

Example:

```http
Strict-Transport-Security: max-age=31536000; includeSubDomains
X-Content-Type-Options: nosniff
Referrer-Policy: no-referrer
```

---

## Cookies

Cookies are small pieces of data stored by the browser and sent back to the server.

They are commonly used for:

- Sessions
- Authentication
- Preferences
- Tracking
- CSRF tokens

Example response:

```http
Set-Cookie: session=abc123; HttpOnly; Secure; SameSite=Lax
```

Example request:

```http
Cookie: session=abc123
```

Important cookie attributes:

| Attribute | Purpose |
|---|---|
| `HttpOnly` | Prevents JavaScript from reading the cookie |
| `Secure` | Sends cookie only over HTTPS |
| `SameSite` | Controls cross-site cookie behavior |
| `Expires` / `Max-Age` | Controls cookie lifetime |

!!! warning
    Session cookies should not store sensitive values in plaintext. They should be random, unpredictable, and protected with secure attributes.

---

## Front End

The front end is the part of a web application that runs in the user's browser.

Core frontend technologies:

| Technology | Purpose |
|---|---|
| HTML | Structure |
| CSS | Styling |
| JavaScript | Interactivity |

Example HTML:

```html
<!DOCTYPE html>
<html>
<head>
  <title>Example Page</title>
</head>
<body>
  <h1>Hello</h1>
  <p>This is a page.</p>
</body>
</html>
```

Security relevance:

- Frontend code is visible to users.
- Client-side validation can be bypassed.
- Sensitive data should not be placed in HTML comments or JavaScript files.
- JavaScript can be a source of XSS and logic flaws.

---

## Back End

The back end runs on the server and handles logic that users do not directly see.

Backend responsibilities:

- Authentication
- Authorization
- Business logic
- Database access
- File handling
- API responses
- Session management

Common backend technologies:

| Type | Examples |
|---|---|
| Languages | PHP, Python, Ruby, JavaScript, Java, C# |
| Frameworks | Laravel, Django, Flask, Express |
| Databases | MySQL, PostgreSQL, MongoDB, MSSQL |
| Web servers | Apache, Nginx, IIS |

Security relevance:

- Backend input handling affects injection vulnerabilities.
- Authentication and authorization must be enforced server-side.
- Backend errors may leak stack traces or database details.
- File upload and path handling must be controlled carefully.

---

## Static vs Dynamic Content

### Static Content

Static content does not change based on user input or backend logic.

Examples:

- Images
- CSS files
- JavaScript files
- Fixed HTML pages
- Fonts

### Dynamic Content

Dynamic content is generated based on request data, user identity, database values, or backend logic.

Examples:

- Search results
- User dashboards
- Shopping carts
- Blog comments
- Account settings

Dynamic content introduces more security risk because user input and backend logic affect the response.

---

## Web Servers

A web server listens for HTTP or HTTPS requests and returns responses.

Common web servers:

| Server | Common Platform |
|---|---|
| Apache | Linux |
| Nginx | Linux |
| IIS | Windows |
| Node.js | JavaScript runtime |

Common web root directories:

| Platform | Common Web Root |
|---|---|
| Apache/Nginx on Linux | `/var/www/html` |
| IIS on Windows | `C:\inetpub\wwwroot` |

---

## Virtual Hosts

Virtual hosting allows one server to host multiple websites.

The server uses the `Host` header to decide which site should handle the request.

Example:

```http
GET / HTTP/1.1
Host: site-one.example
```

Possible mapping:

```text
site-one.example -> /var/www/site-one
site-two.example -> /var/www/site-two
```

Security relevance:

- Incorrect virtual host configuration can expose unintended sites.
- Host header testing can reveal hidden applications.
- Misconfigured default sites can leak information.

---

## Databases

Databases store application data.

Common database types:

| Type | Examples |
|---|---|
| Relational | MySQL, PostgreSQL, MSSQL |
| Non-relational | MongoDB, Redis |

Common stored data:

- Users
- Password hashes
- Sessions
- Orders
- Messages
- Logs
- Application settings

Security relevance:

- SQL injection targets database queries.
- Weak database permissions increase impact.
- Sensitive data should be encrypted or hashed where appropriate.

---

## Load Balancers

A load balancer distributes traffic across multiple backend servers.

Benefits:

- Handles more traffic
- Provides failover
- Improves availability
- Enables maintenance without full downtime

Example:

```text
Client -> Load Balancer -> Web Server 1
                       -> Web Server 2
                       -> Web Server 3
```

Security relevance:

- Load balancers may terminate TLS.
- Headers like `X-Forwarded-For` may affect logging.
- Misconfigured routing can expose internal services.

---

## CDN

A Content Delivery Network, or CDN, serves content from geographically distributed servers.

Common CDN content:

- Images
- CSS
- JavaScript
- Videos
- Static files

Benefits:

- Faster load times
- Reduced origin server load
- Better availability
- DDoS absorption in some cases

Security relevance:

- CDN caching rules must not cache private data.
- Real origin IP exposure can bypass CDN protections.
- WAF/CDN settings may affect testing results.

---

## WAF

A Web Application Firewall, or WAF, filters HTTP traffic before it reaches the web application.

A WAF may block:

- SQL injection patterns
- Cross-site scripting payloads
- Malicious user agents
- Automated traffic
- High request rates

!!! note
    A WAF is a defense-in-depth control. It does not fix vulnerable application code.

---

## Web Application Architecture

Web applications can be built in different layouts.

### One Server Model

Everything runs on one machine:

```text
Web server + application + database
```

Pros:

- Simple
- Easy to deploy
- Common in small labs

Cons:

- Single point of failure
- Full compromise if server is breached
- Harder to scale

### Many Servers, One Database

Multiple web servers connect to one database.

```text
Web Server 1
Web Server 2 -> Database
Web Server 3
```

Pros:

- Better scaling
- More resilient web tier

Cons:

- Database is still central dependency
- Requires secure network segmentation

### Three-Tier Architecture

A common architecture pattern:

| Tier | Purpose |
|---|---|
| Presentation | User interface |
| Application | Business logic |
| Data | Database and storage |

```text
Browser -> Web/App Server -> Database
```

Security relevance:

- Each tier should have limited permissions.
- Internal services should not be exposed publicly.
- Access control should be enforced in the application layer.

---

## APIs

An API allows software to communicate with other software.

Web APIs often use HTTP and return JSON.

Example API response:

```json
{
  "id": 1,
  "username": "alice",
  "role": "user"
}
```

Common API actions map to HTTP methods:

| CRUD Action | HTTP Method |
|---|---|
| Create | `POST` |
| Read | `GET` |
| Update | `PUT` / `PATCH` |
| Delete | `DELETE` |

Security relevance:

- APIs need authentication and authorization.
- Object IDs should not be trusted as proof of access.
- Verbose API errors can leak internal details.
- Rate limiting can reduce abuse.

---

## cURL Basics

cURL is useful for inspecting and manually sending web requests.

### Fetch a Page

```bash
curl https://example.com
```

### Show Response Headers

```bash
curl -I https://example.com
```

### Show Request and Response Details

```bash
curl -v https://example.com
```

### Ignore TLS Certificate Errors

Useful in labs with self-signed certificates:

```bash
curl -k https://example.com
```

### Send a POST Request

```bash
curl -X POST https://example.com/login \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "username=admin&password=password123"
```

### Send JSON

```bash
curl -X POST https://example.com/api/login \
  -H "Content-Type: application/json" \
  -d '{"username":"admin","password":"password123"}'
```

---

## Browser Developer Tools

Browser developer tools help inspect web applications.

Useful tabs:

| Tab | Use |
|---|---|
| Elements | View and edit HTML/CSS |
| Console | View JavaScript output/errors |
| Network | Inspect requests and responses |
| Application | Inspect cookies, storage, cache |
| Sources | Review JavaScript files |

Security testing use cases:

- Inspect hidden fields
- View request headers
- Review cookies
- Check JavaScript files
- Analyze API responses
- Observe redirects and status codes

---

## Security Notes

When reviewing a web application, check for:

- Sensitive data in HTML comments
- Secrets in JavaScript files
- Insecure cookies
- Missing security headers
- Verbose error messages
- Exposed admin paths
- Unsafe HTTP methods
- Client-side-only validation
- Misconfigured CORS
- Publicly accessible backups
- Information disclosure in response headers

---

## Quick Reference

| Concept | Meaning |
|---|---|
| URL | Address of a web resource |
| HTTP | Web communication protocol |
| HTTPS | HTTP over TLS |
| Request | Client message to server |
| Response | Server message to client |
| Header | Metadata in HTTP messages |
| Body | Payload/content of HTTP messages |
| Cookie | Browser-stored state value |
| Front end | Browser-side code |
| Back end | Server-side code |
| API | Interface for software communication |
| WAF | Web traffic filtering layer |
| CDN | Distributed content delivery |
| Load balancer | Distributes traffic across servers |

---

## Notes to Remember

- The browser sends requests; the server returns responses.
- URLs contain scheme, host, port, path, query, and fragment.
- HTTP is unencrypted; HTTPS uses TLS.
- Headers provide metadata and security instructions.
- Cookies are used for state and sessions.
- Frontend code is visible and should not contain secrets.
- Backend code enforces real security decisions.
- Static content is served directly; dynamic content is generated by backend logic.
- APIs often use JSON and HTTP methods.
- cURL and browser DevTools are essential web testing tools.