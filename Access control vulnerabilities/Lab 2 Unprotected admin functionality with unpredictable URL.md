# Unprotected Admin Functionality with Unpredictable URL

**Difficulty:** `APPRENTICE` | **Category:** `Access Control`

---

## 🎯 Lab Goal

Find the hidden admin panel URL and delete the user "carlos".

---

## 📋 Lab Description

This lab has an unprotected admin panel, but its location is obscured with an unpredictable URL disclosed somewhere in the application.

---

## 🔎 Reconnaissance

### Initial Enumeration

Checked common enumeration files:

1. **`/robots.txt`** → Nothing useful
2. **`/sitemap.xml`** → Nothing found

### Source Code Analysis

Examined the page source and discovered interesting JavaScript code:

```javascript
var isAdmin = false;
if (isAdmin) {
   var topLinksTag = document.getElementsByClassName("top-links")[0];
   var adminPanelTag = document.createElement('a');
   adminPanelTag.setAttribute('href', '/admin-4yl2mj');
   adminPanelTag.innerText = 'Admin panel';
   topLinksTag.append(adminPanelTag);
   var pTag = document.createElement('p');
   pTag.innerText = '|';
   topLinksTag.appendChild(pTag);
}
```

🔍 **Key Finding:** The script conditionally renders an admin link, but the URL is hardcoded: `/admin-4yl2mj`

---

## 🧪 Testing & Exploitation

### Step 1: Extract Hidden URL

From the JavaScript code, identified the admin panel path:

```
/admin-4yl2mj
```

Even though the code checks `if (isAdmin)`, the URL itself is exposed in the client-side code.

### Step 2: Access Admin Panel

Navigated directly to the discovered path:

```
https://[lab-id].web-security-academy.net/admin-4yl2mj
```

**Result:** Full admin access granted without authentication!

### Step 3: Delete User

Successfully deleted user "carlos" through the admin interface.

---

## 🚩 Solution

**Hidden Path:** `/admin-4yl2mj`

---

## 💡 Key Takeaways

### Client-Side Security is No Security

- JavaScript runs in the browser (attacker's territory)
- All client-side code is visible and modifiable
- Conditional rendering doesn't hide sensitive data

### Common Mistakes

- **Hardcoding URLs** in JavaScript
- **Relying on client-side checks** (e.g., `if (isAdmin)`)
- **Security through obscurity** with "random" URLs
- Assuming users won't read source code

### Where to Look for Hidden URLs

- **JavaScript files** (inline and external)
- **HTML comments**
- **API responses**
- **Network tab** (XHR/Fetch requests)
- **LocalStorage/SessionStorage**
- **Cookies**

---

## 🛡️ Remediation

### Server-Side Enforcement

**Bad Example (Client-Side Check):**
```javascript
// ❌ This is visible to attackers!
if (isAdmin) {
    showAdminPanel('/admin-4yl2mj');
}
```

**Good Example (Server-Side Check):**
```python
# ✅ Check on server
@app.route('/admin')
@require_admin
def admin_panel():
    return render_template('admin.html')
```

### Best Practices

- ✅ **Never expose sensitive URLs** in client-side code
- ✅ **Implement server-side authentication** for all admin routes
- ✅ **Use role-based access control (RBAC)**
- ✅ **Validate permissions on every request**
- ✅ **Don't rely on URL obscurity** for security
- ✅ **Minimize data sent to client** (only what's needed)
- ✅ **Use proper session management**

### Secure Admin URL Handling

```javascript
// ✅ Good: Fetch admin URL only if authenticated
fetch('/api/user/menu')
  .then(r => r.json())
  .then(data => {
    if (data.isAdmin && data.adminUrl) {
      renderAdminLink(data.adminUrl);
    }
  });
```

```python
# Server side: Only return URL if user is admin
@app.route('/api/user/menu')
def user_menu():
    if current_user.is_admin:
        return {'isAdmin': True, 'adminUrl': '/admin'}
    return {'isAdmin': False}
```

### Defense in Depth

1. **Authentication:** Verify user identity on server
2. **Authorization:** Check permissions on server
3. **Session Security:** Use secure, httpOnly cookies
4. **Rate Limiting:** Prevent brute force attacks
5. **Logging:** Monitor access to sensitive endpoints

---

**Status:** ✅ Solved
