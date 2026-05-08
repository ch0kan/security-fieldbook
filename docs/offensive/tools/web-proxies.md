# Web Proxies

## Intercepting and Modifying HTTP Responses

---

## Executive Summary

In web application penetration testing, intercepting HTTP **responses** is just as important as intercepting HTTP **requests**.

Requests show what the browser sends to the server. Responses show what the server sends back to the browser. By intercepting and modifying responses before the browser renders them, you can bypass weak client-side restrictions such as disabled buttons, hidden fields, numeric-only inputs, and short maximum input lengths.

This does **not** bypass proper server-side validation. It only proves whether the application is relying too heavily on browser-side controls.

---

## Response Interception

Client-side controls are enforced by the browser, not by the server.

Examples include:

- `maxlength="3"`
- `type="number"`
- `disabled`
- `hidden`
- JavaScript-based form restrictions

Because these controls are delivered inside the server response, a proxy can rewrite them before the page loads in the browser.

For example, changing this:

```HTML
<input type="number" maxlength="3">
```

to this:

```HTML
<input type="text" maxlength="100">
```

allows the browser to accept input that the original page tried to block.

---

## Manual Response Interception with Burp Suite

Burp Suite can pause server responses before they reach the browser.

Open:

```text
Proxy > Proxy settings
```

Then enable response interception rules.

A typical workflow is:

```text
Browser request -> Burp catches request -> Forward request -> Burp catches response -> Modify response -> Forward response to browser
```

Common edits include changing:

```HTML
type="number"
```

to:

```HTML
type="text"
```

or removing:

```HTML
disabled
```

from a button or input field.

This is useful for one-off tests where you want to quickly prove that a client-side restriction can be bypassed.

---

## Manual Response Interception with OWASP ZAP

OWASP ZAP can also pause responses.

A common workflow is:

```text
Enable intercept -> Send request from browser -> Click Step -> Modify response -> Continue
```

The **Step** button forwards the intercepted request and then pauses on the incoming response.

From there, you can modify the HTML before it reaches the browser.

---

## Quick UI Bypasses

Some restrictions can be bypassed without manually editing every response.

### OWASP ZAP HUD

In **OWASP ZAP**, the HUD can reveal hidden fields and enable disabled controls directly in the browser interface.

| **HUD Feature** | **Purpose** |
|---|---|
| **Lightbulb / Show Enable** | Reveal hidden fields and enable disabled buttons. |
| **Comments Icon** | Highlight hidden HTML comments. |

### Burp Suite Response Modification

In **Burp Suite**, response modification rules can automatically remove common browser-side restrictions.

| **Burp Feature** | **Purpose** |
|---|---|
| **Unhide hidden form fields** | Makes hidden inputs visible. |
| **Enable disabled form fields** | Allows interaction with disabled inputs. |
| **Remove input length limits** | Helps bypass `maxlength` restrictions. |

---

## Match and Replace Rules

Manual interception is useful, but it becomes tedious when the page reloads often.

**Match and Replace** rules solve this by automatically modifying traffic as it passes through the proxy.

You can use these rules on:

- Request headers
- Request bodies
- Response headers
- Response bodies

---

## User-Agent Spoofing

Some applications behave differently based on the browser or tool making the request.

A proxy can automatically replace the `User-Agent` header in every outgoing request.

Example replacement:

```HTTP
User-Agent: HackTheBox Agent 1.0
```

In Burp Suite, this is usually configured as an HTTP Match and Replace rule against the request header.

Example match pattern:

```Regex
^User-Agent.*$
```

Example replacement:

```HTTP
User-Agent: HackTheBox Agent 1.0
```

In OWASP ZAP, the same idea can be configured through the **Replacer** tool.

Shortcut:

```text
CTRL + R
```

---

## Persistent Response Modification

A common response-side bypass is replacing restricted input types.

For example, automatically replace:

```HTML
type="number"
```

with:

```HTML
type="text"
```

This makes the bypass survive page refreshes.

You can also replace:

```HTML
maxlength="3"
```

with:

```HTML
maxlength="100"
```

This is useful when testing whether the server validates input length independently of the browser.

---

## Burp Suite and ZAP Comparison

| **Task** | **Burp Suite** | **OWASP ZAP** |
|---|---|---|
| **Enable response interception** | Proxy settings > Response interception rules. | Intercept request, then click Step. |
| **Reveal hidden fields** | Response modification rules. | HUD Show/Enable. |
| **Enable disabled fields** | Response modification rules. | HUD Show/Enable. |
| **Replace request headers** | HTTP Match and Replace rules. | Replacer tool. |
| **Replace response body text** | HTTP Match and Replace rules. | Replacer tool. |
| **Highlight comments** | Manual source inspection. | HUD comments icon. |

---

## Testing Mindset

When modifying responses, the main question is:

> Does the server enforce this rule, or is the browser the only thing stopping me?

A strong application validates input on the server side.

A weak application relies only on front-end controls.

For example, if the browser blocks more than three digits but the server accepts a longer value after you remove `maxlength`, the application has a server-side validation weakness.

---

## Key Takeaways

- HTTP responses define how the browser renders and enforces many client-side controls.
- Web proxies can modify those responses before the browser sees them.
- Manual response interception is best for quick testing.
- Match and Replace rules are better for persistent changes across refreshes.
- Client-side restrictions are not security controls unless the server enforces the same rule independently.