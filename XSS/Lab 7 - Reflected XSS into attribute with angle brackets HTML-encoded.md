```markdown
## Lab 7 - Reflected XSS into attribute with angle brackets HTML-encoded (Apprentice)

**Mission Description:**  
The search functionality reflects our input back into an HTML attribute, but `<` and `>` are HTML-encoded. The goal is to inject JavaScript through an HTML attribute.

**Mission Objective:**  
Inject an attribute payload to trigger `alert()` despite `<` and `>` being encoded.

---

**Steps:**  
1. Typed `test` in the search bar to see how it’s reflected.  
   - Found it appears in two places: inside `<h1>` and inside the `value` attribute of the search `<input>`.  
2. Noticed that `<` becomes `&lt;`, so direct tag injection in `<h1>` won’t work.  
3. Focused on the `value` attribute inside the `<input>` tag.  
4. Plan: close the `value` attribute with `"` and inject a new event handler.  
5. Tried payload:  
```

"onmouseover="alert(1)

````
This made the HTML become:  
```html
<input type=text placeholder='Search the blog...' name=search value="test" onmouseover="alert(1)">
````

6. Moved the mouse over the input field → **XSS triggered!**

---

**Final Payload:**

```
"onmouseover="alert(1)
```

**Notes:**

* We didn’t need `<` or `>` since the injection point was inside an attribute.
* Angle bracket encoding is irrelevant when exploiting within existing HTML attributes.

**Status:** Solved ✅

```
```
