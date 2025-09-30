# User Role Can Be Modified in User Profile

**Difficulty:** `APPRENTICE` | **Category:** `Access Control`

---

## 🎯 Lab Goal

Modify the user role to gain admin access and delete user "carlos".

---

## 📋 Lab Description

The app has an admin panel at `/admin` that is restricted to users with `roleid=2`. Normal users are assigned `roleid=1`.

---

## 🔎 Reconnaissance

### Initial Access

Logged in with provided credentials:

| Field | Value |
|-------|-------|
| Username | `wiener` |
| Password | `peter` |

### Admin Panel Test

Attempted to access `/admin` → **Access Denied**

---

## 🧪 Testing & Exploitation

### Step 1: Cookie Analysis

Opened browser **DevTools → Storage/Cookies** and found:

```
roleid=1
```

🔍 **Vulnerability Identified:** Role is stored client-side and can be manipulated!

### Step 2: Role Modification

Changed the cookie value:

**Before:**
```
roleid=1
```

**After:**
```
roleid=2
```

### Step 3: Privilege Escalation

Reloaded the page and accessed `/admin`.

**Result:** Full admin panel access granted! ✅

### Step 4: Delete User

Successfully deleted user "carlos" through the admin interface.

---

## 🚩 Solution

**Modified Cookie:** `roleid=2`

---

## 💡 Key Takeaways

### Broken Access Control

- **Client-side role storage** is inherently insecure
- Users can trivially modify their own cookies
- Role identifiers must be validated server-side

### Similar Vulnerable Patterns

```
roleid=2
role_id=admin
usertype=administrator
access_level=high
privilege=2
```

All of these are vulnerable if stored client-side without cryptographic protection.

---

## 🛡️ Remediation

### Secure Implementation

**❌ Bad (Client-Side Role):**
```python
@app.route('/admin')
def admin():
    if request.cookies.get('roleid') == '2':
        return render_admin_panel()
    return "Access Denied"
```

**✅ Good (Server-Side Validation):**
```python
@app.route('/admin')
@login_required
def admin():
    user = User.query.get(session['user_id'])
    if user.role_id != 2:
        abort(403)
    return render_admin_panel()
```

### Best Practices

- ✅ **Store roles in database**, not cookies
- ✅ **Validate on every request** from server-side
- ✅ **Use server-side sessions** for user state
- ✅ **Never trust client-supplied authorization data**
- ✅ **Implement proper RBAC** (Role-Based Access Control)
- ✅ **Log privilege escalation attempts**

---

**Status:** ✅ Solved
