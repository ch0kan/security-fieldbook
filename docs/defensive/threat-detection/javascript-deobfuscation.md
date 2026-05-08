# JavaScript Deobfuscation

JavaScript deobfuscation is the process of turning unreadable or intentionally hidden JavaScript back into a form that analysts can understand.

It is useful for investigating malicious redirects, web skimmers, browser exploits, phishing kits, and suspicious scripts found on compromised websites.

---

## Why JavaScript Is Obfuscated

JavaScript obfuscation is used for both legitimate and malicious reasons.

| Use | Description |
|---|---|
| Legitimate | Reduce file size, protect intellectual property |
| Malicious | Hide payloads, C2 domains, skimmers, redirects, or exploit logic |

Common targets for analysis:

- Injected website scripts
- Minified `.js` files
- Suspicious browser redirects
- Magecart-style skimmers
- Obfuscated phishing pages
- Packed JavaScript payloads

---

## Minification

Minification removes unnecessary characters without changing functionality.

Removed content often includes:

- Spaces
- Newlines
- Comments
- Long variable names
- Formatting

Example characteristics:

```text
One long line
Very short variable names
No comments
No indentation
```

Minification makes code harder to read, but it does not necessarily hide the logic deeply.

---

## Beautification

Beautification, or pretty printing, restores formatting.

It adds:

- Newlines
- Indentation
- Function structure
- Readable blocks

Beautification does not decode encrypted or packed logic. It only makes the visible structure easier to inspect.

Useful tools:

| Tool | Use |
|---|---|
| Browser DevTools Pretty Print | Format loaded scripts |
| Prettier | Format JavaScript |
| JS Beautifier | Format JavaScript |
| Code editor formatters | Local safe formatting |

---

## Browser DevTools Pretty Print

Most browsers can pretty print minified JavaScript.

General workflow:

1. Open Developer Tools.
2. Go to the Sources or Debugger tab.
3. Select the suspicious script.
4. Click the `{ }` pretty print button.
5. Review function names, strings, URLs, and execution flow.

This is useful for scripts already loaded in the browser.

---

## Packing

Packing hides code more deeply than minification.

Packed JavaScript usually contains:

- Encoded payload
- Dictionary of strings
- Runtime unpacker
- Final execution sink such as `eval()`

The script reconstructs the real code in memory and then executes it.

---

## The `p,a,c,k,e,d` Packer

A common JavaScript packer uses a function with arguments similar to:

```javascript
eval(function(p,a,c,k,e,d){ ... })
```

This is often associated with Dean Edwards-style packing.

| Argument | Meaning |
|---|---|
| `p` | Encoded payload |
| `a` | Base/radix |
| `c` | Keyword count |
| `k` | Keyword dictionary |
| `e` | Encoder/helper |
| `d` | Runtime dictionary object |

This pattern is a strong signal that the script is packed.

---

## How Packed JavaScript Works

Typical flow:

```text
Original JavaScript
  -> Extract keywords
  -> Build dictionary
  -> Replace words with encoded indexes
  -> Wrap in unpacker function
  -> Execute unpacker in browser
  -> Pass decoded code to eval()
```

The original logic is hidden until runtime.

---

## Execution Sinks

Execution sinks dynamically run strings as JavaScript.

Common sinks:

```javascript
eval()
Function()
setTimeout("code", delay)
setInterval("code", delay)
```

Security relevance:

- Obfuscated malware often reconstructs code as a string.
- The final string is passed into one of these functions.
- Analysts often intercept this step to print the decoded code instead of running it.

---

## Safe Manual Unpacking Concept

A common analysis technique is to replace the final execution sink with output.

Conceptually:

```javascript
eval(decoded_payload)
```

becomes:

```javascript
console.log(decoded_payload)
```

This allows the analyst to view the decoded JavaScript without executing it.

!!! warning
    Only do this in an isolated analysis environment. Obfuscated JavaScript may contain browser exploits or credential theft logic.

---

## Deobfuscation Workflow

Recommended process:

```text
Identify suspicious script
  -> Beautify
  -> Look for eval/Function/setTimeout
  -> Identify packer pattern
  -> Unpack or intercept decoded output
  -> Beautify decoded payload
  -> Repeat if multiple layers exist
  -> Extract IOCs and behavior
```

---

## What to Look For

During analysis, search for:

- Domains
- IP addresses
- URLs
- Form field names
- Payment fields
- Cookie access
- Local storage access
- External script loads
- `XMLHttpRequest`
- `fetch`
- `document.cookie`
- `atob`
- `btoa`
- `eval`
- `Function`
- `setTimeout`
- `String.fromCharCode`

---

## Common Malicious JavaScript Behaviors

| Behavior | Possible Indicator |
|---|---|
| Credential theft | Reads login form fields |
| Web skimming | Reads card/payment fields |
| Redirect | Changes `window.location` |
| C2 polling | Repeated `fetch` or XHR |
| Cookie theft | Reads `document.cookie` |
| Payload staging | Loads external script dynamically |
| Anti-analysis | Checks browser/devtools/timing |

---

## Detection Opportunities

Defenders can monitor for:

- Heavy obfuscation
- Long encoded strings
- `eval(function(p,a,c,k,e,d)`
- `eval(atob(...))`
- Suspicious external script loads
- JavaScript injected into CMS templates
- Unexpected changes to production JS files
- Checkout pages loading rare third-party domains

---

## Content Security Policy

Content Security Policy can reduce risk.

Useful restrictions:

```text
Disallow unsafe-eval
Restrict script-src
Restrict external domains
Block inline scripts where possible
```

Example concept:

```http
Content-Security-Policy: script-src 'self' https://trusted-cdn.example;
```

Avoid allowing:

```text
'unsafe-eval'
'unsafe-inline'
*
```

---

## Analysis Safety

Best practices:

- Use an isolated VM.
- Prefer offline tools.
- Avoid pasting sensitive production code into random online tools.
- Do not execute unknown scripts in your main browser.
- Capture IOCs before interacting with live infrastructure.
- Record the original script hash and source URL.

---

## Quick Reference

| Task | Tool / Technique |
|---|---|
| Format minified code | Beautifier / Prettier / DevTools `{ }` |
| Identify packer | Look for `eval(function(p,a,c,k,e,d)` |
| Unpack payload | Dedicated unpacker or intercept `eval()` |
| Extract IOCs | Search URLs, domains, cookies, form fields |
| Prevent execution | CSP without `unsafe-eval` |
| Analyze safely | Isolated VM or offline tooling |

---

## Notes to Remember

- Beautification restores readability but does not decode hidden logic.
- Packing reconstructs the real code at runtime.
- `eval()` is a major execution sink.
- Replacing `eval()` with output is a common unpacking technique.
- Multiple obfuscation layers may require repeated analysis.