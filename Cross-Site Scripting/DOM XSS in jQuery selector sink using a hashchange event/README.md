### Lab: DOM XSS in jQuery selector sink using a hashchange event

## 1 Vulnerability

**Source: location.hash**

**sink: jQuery selector function $()**

**Requirement: active print() function on victim's browser**

**Logic:  web uses jQuery to lead to a post based on ID but no input data filtering**

## 2 Exploitation

**Firstly, go to exploit server**

**Insert payload into body**

```
payload :

<iframe src="https://0a96000d0491a20287ed33b4006700ed.web-security-academy.net/#" onload="this.src+='<img src=x onerror=print()>'"></iframe>
```

**Then we store and deliver to victim**

*I use <iframe> on exploit server to trigger the hashchange event on the victim's browser*

*Appending <img> with an onerror attribute to the URL hash, I force the jQuery selector to execute the print() function*

## 3 Prevention

**We should use specialized function for scrolling that don't interpret input as HTML to avoid danger**
