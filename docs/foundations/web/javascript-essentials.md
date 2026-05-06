# JavaScript Essentials

JavaScript is a programming language used to make web pages interactive. It runs in the browser, interacts with HTML and CSS, and can update page content without requiring a full page reload.

For cybersecurity, JavaScript is important because many web vulnerabilities involve browser-side behavior, DOM manipulation, client-side validation, cookies, APIs, and Cross-Site Scripting.

---

## What JavaScript Does

JavaScript can:

- Update page content
- React to user actions
- Send requests to APIs
- Validate forms
- Read and modify the DOM
- Handle cookies and browser storage
- Build dynamic single-page applications

Example:

```html
<p id="message">Original text</p>

<script>
  document.getElementById("message").innerText = "Updated by JavaScript";
</script>
```

---

## JavaScript in the Browser

The browser receives HTML, CSS, and JavaScript from the server.

```text
Browser -> HTTP Request -> Server
Browser <- HTML/CSS/JS <- Server
Browser executes JavaScript locally
```

Important security point:

!!! warning
    JavaScript sent to the browser is visible to users. Never place secrets, passwords, API keys, or private logic in frontend JavaScript.

---

## Variables

Variables store values.

Modern JavaScript commonly uses:

| Keyword | Scope | Reassignable | Notes |
|---|---|---:|---|
| `var` | Function scoped | Yes | Older style, often avoided |
| `let` | Block scoped | Yes | Use when value changes |
| `const` | Block scoped | No | Use by default when value should not change |

Example:

```javascript
const username = "alice";
let loginAttempts = 0;

loginAttempts = loginAttempts + 1;

console.log(username);
console.log(loginAttempts);
```

---

## Data Types

Common JavaScript data types:

| Type | Example |
|---|---|
| String | `"hello"` |
| Number | `42` |
| Boolean | `true` / `false` |
| Null | `null` |
| Undefined | `undefined` |
| Object | `{ name: "alice" }` |
| Array | `["admin", "user"]` |

Example:

```javascript
const name = "alice";
const age = 25;
const isAdmin = false;
const roles = ["user", "editor"];

console.log(typeof name);    // string
console.log(typeof age);     // number
console.log(typeof isAdmin); // boolean
```

---

## Objects

Objects store key-value pairs.

```javascript
const user = {
  id: 1,
  username: "alice",
  role: "user",
  active: true
};

console.log(user.username);
console.log(user.role);
```

Security relevance:

- API responses are often JSON objects.
- Sensitive fields may accidentally appear in frontend responses.
- Role or permission values visible in JavaScript should not be trusted alone.

---

## Arrays

Arrays store ordered lists.

```javascript
const ports = [22, 80, 443];

console.log(ports[0]); // 22
console.log(ports.length); // 3
```

Loop through an array:

```javascript
const users = ["alice", "bob", "charlie"];

for (let i = 0; i < users.length; i++) {
  console.log(users[i]);
}
```

---

## Conditions

Conditional statements run code only when a condition is true.

```javascript
const role = "admin";

if (role === "admin") {
  console.log("Access granted");
} else {
  console.log("Access denied");
}
```

Common comparison operators:

| Operator | Meaning |
|---|---|
| `===` | Equal value and type |
| `!==` | Not equal value or type |
| `>` | Greater than |
| `<` | Less than |
| `>=` | Greater than or equal |
| `<=` | Less than or equal |

!!! note
    JavaScript also has `==`, but `===` is usually preferred because it avoids type coercion surprises.

---

## Functions

Functions are reusable blocks of code.

```javascript
function greet(name) {
  return "Hello, " + name;
}

console.log(greet("alice"));
```

Arrow function syntax:

```javascript
const greet = (name) => {
  return "Hello, " + name;
};

console.log(greet("bob"));
```

Functions help avoid repeating logic.

---

## Loops

Loops repeat code.

### For Loop

```javascript
for (let i = 0; i < 5; i++) {
  console.log(i);
}
```

### While Loop

```javascript
let count = 0;

while (count < 5) {
  console.log(count);
  count++;
}
```

### For...of Loop

```javascript
const tools = ["nmap", "burp", "ffuf"];

for (const tool of tools) {
  console.log(tool);
}
```

---

## DOM Basics

The DOM, or Document Object Model, represents the page structure in the browser.

JavaScript can read and modify the DOM.

```html
<h1 id="title">Original Title</h1>

<script>
  const title = document.getElementById("title");
  title.innerText = "Updated Title";
</script>
```

Common DOM methods:

| Method | Purpose |
|---|---|
| `document.getElementById()` | Select element by ID |
| `document.querySelector()` | Select first matching CSS selector |
| `document.querySelectorAll()` | Select all matching elements |
| `element.innerText` | Read or write visible text |
| `element.innerHTML` | Read or write HTML content |

Security relevance:

!!! warning
    Writing untrusted input into `innerHTML` can create Cross-Site Scripting vulnerabilities.

---

## Events

Events let JavaScript respond to user actions.

Examples:

- Clicks
- Key presses
- Form submissions
- Page loads
- Mouse movement

Example:

```html
<button id="btn">Click me</button>

<script>
  document.getElementById("btn").addEventListener("click", () => {
    alert("Button clicked");
  });
</script>
```

---

## Fetch API

JavaScript can send HTTP requests using `fetch()`.

```javascript
fetch("/api/users")
  .then(response => response.json())
  .then(data => {
    console.log(data);
  });
```

POST JSON data:

```javascript
fetch("/api/login", {
  method: "POST",
  headers: {
    "Content-Type": "application/json"
  },
  body: JSON.stringify({
    username: "alice",
    password: "password123"
  })
});
```

Security relevance:

- API endpoints may be visible in JavaScript files.
- Client-side requests can be replayed or modified.
- Authorization must be enforced on the server.

---

## Browser Storage

Browsers can store data locally.

Common storage types:

| Storage | Description |
|---|---|
| Cookies | Sent automatically with matching requests |
| Local Storage | Persistent browser storage |
| Session Storage | Storage cleared when tab/session ends |

Example:

```javascript
localStorage.setItem("theme", "dark");
const theme = localStorage.getItem("theme");
console.log(theme);
```

Security relevance:

!!! warning
    Do not store sensitive tokens in local storage unless you understand the risk. XSS can expose browser-accessible storage.

---

## Client-Side Validation

JavaScript often validates form input before submitting it.

Example:

```javascript
function validatePassword(password) {
  return password.length >= 12;
}
```

Client-side validation improves user experience, but it is not a security boundary.

!!! important
    Any validation done in the browser can be bypassed. The server must enforce validation too.

---

## Common Security Issues

JavaScript is often involved in web vulnerabilities.

| Issue | Description |
|---|---|
| XSS | Untrusted input executed as JavaScript |
| Sensitive data exposure | Secrets left in JS files or page source |
| Client-side trust | Server trusts values controlled by browser |
| Insecure storage | Tokens or secrets stored insecurely |
| DOM manipulation flaws | Unsafe use of attacker-controlled data |
| Weak frontend authorization | Hiding buttons instead of enforcing access server-side |

---

## Practical Review Checklist

When reviewing JavaScript:

- Look for hardcoded secrets
- Search for API endpoints
- Check how user input reaches the DOM
- Look for `innerHTML`, `eval`, and unsafe sinks
- Review authentication and authorization logic
- Check local storage and session storage
- Inspect fetch/XHR requests
- Confirm server-side validation exists

---

## Quick Reference

| Concept | Meaning |
|---|---|
| Variable | Stores data |
| Function | Reusable code block |
| Loop | Repeats code |
| Object | Key-value data structure |
| Array | Ordered list |
| DOM | Browser representation of the page |
| Event | User or browser action |
| Fetch | Browser API for HTTP requests |
| Local Storage | Persistent browser-side storage |

---

## Notes to Remember

- JavaScript runs in the user's browser.
- Frontend JavaScript is visible to users.
- Client-side validation can be bypassed.
- Sensitive values should not be stored in frontend code.
- DOM manipulation is powerful but can be dangerous.
- Server-side authorization is always required.