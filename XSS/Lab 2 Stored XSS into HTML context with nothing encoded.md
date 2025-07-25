## 🧪 Lab: Stored XSS into HTML context with nothing encoded (APPRENTICE)

### 🎯 Mission Description  
This lab has a stored XSS vulnerability inside the blog comment section. The goal is to inject a payload that triggers an `alert()` when someone views the blog post.

### 🎯 Mission Objective  
Submit a comment that triggers `alert(1)` when the blog post is viewed.

### 🧩 Steps  
1. Open the lab URL and go to any blog post that allows comments.

2. You'll find 4 input fields:
   - **Comment**
   - **Name**
   - **Email**
   - **Website** (optional)

3. Fill the form like this:
   - Comment: `good`
   - Name: `ahmed`
   - Email: `aaaa@gmail.com`
   - Website: (leave empty)

4. Check the comment in the source code and you'll see:
   ```html
   <p>good</p>
   ```

5. Try a test injection like `X<"x` to see how it's handled:
   ```html
   <p>X<"x</p>
   ```

6. Since nothing is encoded, XSS is possible!

7. Now inject the final payload:
   ```html
   <img src=x onerror=alert(1)>
   ```

8. Submit the comment and once the post is viewed — 💥 alert pops!

### 💣 Final Payload  
```html
<img src=x onerror=alert(1)>
```

### 📝 Notes  
- Classic stored XSS.
- Inputs aren't sanitized or encoded.
- Even `<script>` would work, but `<img>` is smaller and stealthier.

**Status: Solved ✅**
