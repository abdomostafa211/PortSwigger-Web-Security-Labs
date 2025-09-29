# User Role Controlled by Request Parameter

**Difficulty:** `APPRENTICE` | **Category:** `Access Control`

---

## 🎯 Lab Goal

Escalate privileges by manipulating a forgeable cookie to access the admin panel and delete user "carlos".

---

## 📋 Lab Description

This lab has an admin panel at `/admin` that identifies administrators using a forgeable cookie value.

---

## 🔎 Reconnaissance

### Initial Access

Logged in with provided credentials:

| Field | Value |
|-------|-------|
| Username | `wiener` |
| Password | `peter` |

### Admin Panel Discovery

Attempted to access `/admin` endpoint.

**Response:**
```
Admin interface only available if logged in as an administrator.
```

---

## 🧪 Testing & Exploitation

### Step 1: Intercept Request

Used Burp Suite to intercept the request to `/admin`.

**Original Request Headers:**
```http
GET /admin HTTP/1.1
Host: [lab-id].web-security-academy.net
Cookie: session=...; Admin=false
```

🔍 **Vulnerability Identified:** The `Admin` cookie controls access privileges!

### Step 2: Cookie Tampering

Modified the cookie value:

**Before:**
```
Admin=false
```

**After:**
```
Admin=true
```

### Step 3: Privilege Escalation

Resent the modified request to `/admin`.

**Result:** Successfully accessed the admin panel! ✅

### Step 4: Delete User

Used admin functionality to delete user "carlos".

---

## 🚩 Solution

**Tampered Cookie:** `Admin=true`

---

## 💡 Key Takeaways

### Insecure Access Control

- **Client-side role enforcement** is fundamentally flawed
- Cookies can be easily modified by users
- Never trust client-supplied data for authorization

### Cookie Tampering

Client-controlled values that should NEVER determine privileges:
- `Admin=true/false`
- `isAdmin=1/0`
- `role=admin`
- `privilege=administrator`
- Any unencrypted/unsigned role identifier

### Common Broken Access Control Patterns

| Vulnerable Pattern | Why It Fails |
|-------------------|--------------|
| Cookie-based roles | Client can modify cookies |
| URL parameters (`?admin=true`) | Easily manipulated |
| Hidden form fields | Visible in HTML source |
| JavaScript checks | Runs on client side |

---

## 🛡️ Remediation

### Secure Implementation

**❌ Bad (Client-Side Control):**
```python
# Vulnerable: Trust client cookie
@app.route('/admin')
def admin_panel():
    if request.cookies.get('Admin') == 'true':
        return render_template('admin.html')
    return "Access Denied"
```

**✅ Good (Server-Side Control):**
```python
# Secure: Check server-side session
@app.route('/admin')
@login_required
def admin_panel():
    user = get_current_user_from_db(session['user_id'])
    if not user.is_admin:
        abort(403)
    return render_template('admin.html')
```

### Best Practices

#### 1. Server-Side Session Management

```python
# Store role in server-side session
session['user_id'] = user.id
# Don't store: session['is_admin'] = True

# Always verify from database
user = User.query.get(session['user_id'])
if user.role == 'admin':
    # Allow access
```

#### 2. Signed/Encrypted Tokens

If you must use cookies for authorization:

```python
from itsdangerous import TimedJSONWebSignatureSerializer

# Sign the data
s = TimedJSONWebSignatureSerializer(secret_key)
token = s.dumps({'user_id': 123, 'role': 'admin'})

# Verify on each request
try:
    data = s.loads(token)
    # Token is valid and untampered
except:
    # Token invalid or tampered
    abort(403)
```

#### 3. JWT with Proper Validation

```javascript
// Don't store sensitive claims in JWT payload unprotected
// Always verify signature server-side
const jwt = require('jsonwebtoken');

// Verify token
jwt.verify(token, SECRET_KEY, (err, decoded) => {
    if (err) return res.status(403).send('Invalid token');
    // Check role from decoded token
    if (decoded.role !== 'admin') {
        return res.status(403).send('Insufficient privileges');
    }
});
```

### Defense in Depth Checklist

- ✅ **Never trust client input** for authorization decisions
- ✅ **Store roles server-side** (database/session)
- ✅ **Validate permissions** on every sensitive request
- ✅ **Use cryptographic signatures** if client-side storage needed
- ✅ **Implement proper session management**
- ✅ **Log all privilege escalation attempts**
- ✅ **Apply principle of least privilege**
- ✅ **Regular security audits** of access control logic

### Testing for Similar Issues

```bash
# Test cookie manipulation
# Change role/admin cookies
Admin=true
isAdmin=1
role=admin
userType=administrator

# Test parameter tampering
?admin=true
?role=admin
?privilege=high
```

---

**Status:** ✅ Solved
