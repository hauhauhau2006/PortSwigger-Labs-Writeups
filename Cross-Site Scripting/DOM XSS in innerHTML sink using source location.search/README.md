### Lab: DOM XSS in innerHTML sink using source location.search

## Vulnerability:

*Category: OWASP A03:2021 - Injection*

*Mechanism: The application ultilizes a dangerous Javascript sink(innerHTML) to render user-controlled input from URL(location.search) directly into the DOM*

## Exploitation:

**Payload**
```

<img src=1 onerror=alert(1)>
```

**Explain: By injecting an <img> tag with an invalid source, I triggered the onerror  event handle. Since the application does not sanitize the input before passing it to innerHTML, the browser executes the javascript payload automatically**


