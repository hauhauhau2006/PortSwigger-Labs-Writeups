### Lab: DOM XSS in document.write sink using source location.search

## Vulnerability:

*The danger of javascript sink(document.write) to change data on the web. It takes input from source(location.search) without proper sanitization*

## Exploitation:

*I search a on search box and provided 5 picture*

*Open burp, I replace order postid -> a to access*

*Send 200 Ok, I return web to use payload*

**Payload**
```
"><svg onload=alert(1)>
```

**"><svg ensure to exit source code 'a'**


![Intruder Results](Screenshot%202026-02-26%20231423.png)
