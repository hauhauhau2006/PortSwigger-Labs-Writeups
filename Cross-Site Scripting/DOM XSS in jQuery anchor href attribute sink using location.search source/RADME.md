### Lab : DOM XSS in jQuery anchor href attribute sink using location.search source

## 1. Vulnerability

**In lab type about href sink, web normally has Client-side script**

*Source: location.search, urlParams.get('returnPath') will reception anything behind 'returnPath='. Therefore, we need to insert harm URL here*

*jQuery: .attr('href', backPath) is responsible about finding HTML containing class .btn-back to change URL value*

*Sink: HTML will change from :*

```

   <a class="btn-back" href="/">Back</a>

--> <a class="btn-back" href="javascript:alert(document.cookie)">Back</a>.

```
## 2.Logic

**Normally, href used to contain web address. However, it also accepts 'javascript:', and web will perform what standing behind :**

**It has a access right to document.cookie to steal sensitive information**

## 3.Prevention

**Ensure that input does't start with javascript:(or dangerous protocols)**

**URL encoding**
