# Unprotected Admin Functionality

**Difficulty:** `APPRENTICE` | **Category:** `Access Control`

---

## 🎯 Lab Goal

Delete the user "carlos" by accessing an unprotected admin panel.

---

## 📋 Lab Description

This lab has an unprotected admin panel that can be accessed without authentication.

---

## 🔎 Reconnaissance

### robots.txt Analysis

Checked `/robots.txt` for any disclosed paths:

```
User-agent: *
Disallow: /administrator-panel
```

🚨 **Critical finding:** The admin panel path is disclosed in `robots.txt`

---

## 🧪 Testing & Exploitation

### Step 1: Access Admin Panel

Navigated directly to the discovered path:

```
/administrator-panel
```

**Result:** Admin panel loaded successfully without any authentication!

### Step 2: Delete User

The admin interface provided user management functionality. Successfully deleted user "carlos" with no restrictions.

---

## 🚩 Solution

**Path:** `/administrator-panel`

---

## 💡 Key Takeaways

### Security Through Obscurity Fails
- Hiding paths in `robots.txt` doesn't protect them
- `robots.txt` is publicly readable by anyone
- Attackers always check common enumeration files

### Missing Access Controls
- No authentication required to access admin panel
- No authorization checks on admin functions
- Critical administrative actions exposed

### Common Enumeration Files
Always check these for information disclosure:
- `/robots.txt`
- `/sitemap.xml`
- `/.well-known/`
- `/security.txt`
- `/admin`, `/administrator`, `/admin-panel`

---

## 🛡️ Remediation

### Implement Proper Authentication

```python
# Example: Flask route protection
from flask import session, redirect

@app.route('/administrator-panel')
def admin_panel():
    if 'user_role' not in session or session['user_role'] != 'admin':
        return redirect('/login')
    return render_template('admin.html')
```

### Best Practices

- ✅ **Require authentication** for all admin functionality
- ✅ **Implement role-based access control (RBAC)**
- ✅ **Use secure session management**
- ✅ **Never rely on URL obscurity** for security
- ✅ **Log all admin actions** for audit trails
- ✅ **Implement rate limiting** on sensitive endpoints
- ✅ **Use CSRF tokens** for state-changing operations

### Defense in Depth

1. **Authentication Layer:** Verify user identity
2. **Authorization Layer:** Check user permissions
3. **Network Layer:** Restrict admin access by IP if possible
4. **Monitoring Layer:** Alert on suspicious admin access

---

**Status:** ✅ Solved
