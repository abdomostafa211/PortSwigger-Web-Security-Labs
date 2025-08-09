
## 🧪 Lab 7 - Reflected XSS into attribute with angle brackets HTML-encoded (APPRENTICE)

---

## 🎯 Mission Description  
The search functionality reflects user input inside an HTML attribute, but `<` and `>` are HTML-encoded. We need to inject JavaScript by breaking out of the attribute context.

---

## 🎯 Mission Objective  
Inject an attribute-based payload to trigger `alert()` despite angle bracket encoding.

---

## 🪜 Steps

1. Typed `test` in the search bar to observe reflection.  
   - Found reflection in two places: inside `<h1>` and inside the `value` attribute of the `<input>` field.  
2. Noticed `<` becomes `&lt;`, so injecting tags in `<h1>` is impossible.  
3. Focused on the `value` attribute inside the `<input>` tag.  
4. Closed the existing `value` attribute with `"` and injected an event handler.  
5. Payload used:  
```

"onmouseover="alert(1)

````
6. This produced:  
```html
<input type=text placeholder='Search the blog...' name=search value="test" onmouseover="alert(1)">
````

7. Hovered over the search box → **Alert triggered** 💥.

---

## 💣 Final Payload

```
"onmouseover="alert(1)
```

---

## 📝 Notes

* Encoding of `<` and `>` doesn’t matter for attribute injection.
* Exploitation worked by staying within attribute context.

---

**✅ Status: Solved**

```
```
